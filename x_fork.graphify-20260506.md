# x_fork.graphify-20260506

Session 日期：**2026-05-06**
工具：Claude Code (Opus 4.7, 1M context) + `/graphify` skill
代理人：miso168net

本檔記錄該 session 對 `fork260506-go-admin` 倉庫所做的探索、發現、與變更，作為 fork 維護日誌。

---

## 0. Session 起點

- 倉庫：`miso168net/fork260506-go-admin`（go-admin 的 fork，主分支為 `main`，上游為 `master`）
- 起始分支：`main`（HEAD `8ad6a0d docs: add x_fork.branch-origin.md to record main branch source`）
- 工作目錄乾淨。
- 觸發指令：`/graphify . --obsidian`

---

## 1. graphify 第一階段（完整 pipeline）

### Detect

```
Corpus: 196 files · ~62,915 words
  code:     188 files
  docs:     6 files
  images:   2 files
```

### 抽取

- **AST**（結構性、確定）：1,716 nodes / 6,407 edges
- **語意 subagents**（並行 3 個 general-purpose agent）：
  - Chunk 1：6 docs（LICENSE, README × 3, x_fork.branch-origin.md, config/READMEN.md, log.txt）
  - Chunk 2：`static/form-generator/img/logo.e1bc3747.png`
  - Chunk 3：`static/uploadfile/77cfc1dd-...jpg`
- 8 個 uncached 檔案被新跑；其餘 188 命中先前 cache
- 合併後：1,762 nodes / 11,641 edges

### 建圖 + 分群 + 報告

- 1,717 nodes / 5,389 edges / 79 communities（建圖時 dedup）
- 對 79 個社群手動指定 2-5 字標籤（`Frontend Parser Internals`、`Admin REST APIs`、`Admin Models Layer` 等）
- 產出 `graphify-out/graph.html`、`graph.json`、`GRAPH_REPORT.md`、`obsidian/`（1796 notes）+ `graph.canvas`
- benchmark：62.9K words → 83.9K naive tokens；avg query ~26.3K tokens（3.2× 壓縮）

### 第一階段成本

11,800 input / 3,200 output tokens（單一 chunk 1 subagent 成本，images 不計入主成本）

---

## 2. 圖譜驅動的深入分析

針對最高 betweenness 的 cross-community bridge `IndexAction()`（`common/actions/index.go:18`）做了三題追蹤：

### 2.1 `ActiveRecord` 介面（`common/models/type.go:5`）

```go
type ActiveRecord interface {
    schema.Tabler           // TableName() string
    SetCreateBy(int)
    SetUpdateBy(int)
    Generate() ActiveRecord
    GetId() interface{}
}
```

- 圖上原本只有 1 條 `contains` edge — pure AST 看不到 Go 介面的隱式實作。
- 實際上每個 `app/admin/models/sys_*.go` 模型都符合此契約。
- 是 CRUD action factory 的型別邊界（`IndexAction(m models.ActiveRecord, ...)`）。

### 2.2 CRUD Action 對稱性

`common/actions/` 6 個工廠：

| Action | Degree | 寫入 | 權限 scope | by 欄位 |
|---|---|---|---|---|
| `IndexAction` | 18 | — | ✓ | — |
| `ViewAction` | 15 | — | ✓ | — |
| `DeleteAction` | 15 | `Delete` | ✓ | `SetUpdateBy` |
| `UpdateAction` | 14 | `Updates` | ✓ | `SetUpdateBy` |
| `CreateAction` | 12 | `Create` | **無** | `SetCreateBy` |
| `PermissionAction` | 8 | producer | — | — |

`CreateAction` 不掛 Permission scope（合理，新建紀錄無 owner 可比對）。

### 2.3 Permission 流水線

```
[Router middleware]  authMiddleware → AuthCheckRole → PermissionAction()
                                                            │  user.GetUserIdStr(c)
                                                            ▼
                          newDataPermission(db, userId)   (JOIN sys_user × sys_role)
                                                            │
                                                            ▼ c.Set(PermissionKey, p)
                                                       gin.Context
                                                            │
[Service layer]                                             ▼ getPermissionFromContext(c)
                          IndexAction → Permission(tableName, p) → GORM scope
                                                            │
                                            switch p.DataScope:
                                              "1" / 預設 → 不過濾
                                              "2" → role_dept JOIN
                                              "3" → 本部門
                                              "4" → 本部門 + 子層 (dept_path LIKE)
                                              "5" → 僅本人 (create_by = user_id)
```

外層尚有 `if !config.ApplicationConfig.EnableDP { return db }` 全域 kill-switch。

### 2.4 三項關鍵發現

1. **`enabledp` 預設 `false`**：`config/settings.yml`、`settings.full.yml`、`settings.sqlite.yml` 全部 `false`，只有 `settings.demo.yml` 是 `true`。
2. **13 個 admin router 群組沒掛 `PermissionAction()`**：只有 `/sys-user`、`/user`、`/job` 真正會計算 user 的 data scope。其他路徑（`/sys-api`、`/config`、`/role`、`/dept`、`/menu`、`/post`、`/dict`、`/sys-login-log`、`/sys-opera-log`）會收到 zero-value `*DataPermission{DataScope:""}`，落入 `Permission()` 的 `default` 分支，等同**不過濾**。
3. **`DataPermission` struct 重複**：`common/actions/permission.go:15` 與 `app/admin/models/datascope.go:12` 兩份完全相同 4-field struct。`models.DataPermission` 與 `(*DataPermission).GetDataScope` 整段為 dead code，零外部 caller。

雙重關閉的結果：開箱跑起來，所有 admin user 看到所有資料。

---

## 3. 程式碼 / 設定變更（A 動作）

`feat(security): enable admin data-scope permission across CRUD routes`（commit `a4de839`，分支 `data-scope-enable-20260506`，已 push、未開 PR）：

### 3.1 yml 設定

| 檔案 | 變更 |
|---|---|
| `config/settings.yml:14` | `enabledp: false` → `true` |
| `config/settings.full.yml:14` | `enabledp: false` → `true` |
| `config/settings.sqlite.yml:14` | `enabledp: false` → `true` |
| `config/settings.demo.yml` | （原本就 `true`，未動） |

### 3.2 Router middleware 補洞

9 個 router 主要 CRUD group 加上 `.Use(actions.PermissionAction())` 並新增 `"go-admin/common/actions"` import：

| 檔案 | Group |
|---|---|
| `app/admin/router/sys_api.go:18` | `/sys-api` |
| `app/admin/router/sys_config.go:18` | `/config` |
| `app/admin/router/sys_dept.go:18` | `/dept` |
| `app/admin/router/sys_dict.go:18` | `/dict` |
| `app/admin/router/sys_login_log.go:18` | `/sys-login-log` |
| `app/admin/router/sys_menu.go:18` | `/menu` |
| `app/admin/router/sys_opera_log.go:17` | `/sys-opera-log` |
| `app/admin/router/sys_post.go:17` | `/post` |
| `app/admin/router/sys_role.go:19` | `/role` |

未動到的 sub-group（公開 / option-select / 設定型，不需 row-level 過濾）：
- `sys_config.go`：`/configKey`、`/app-config`、`/set-config`
- `sys_dept.go`：`/deptTree`
- `sys_dict.go`：`/dict-data` (option-select)
- `sys_menu.go`：`/menurole`
- `sys_role.go`：`/role-status`、`/roledatascope`

### 3.3 未做（依用戶決定保留）

- `app/admin/models/datascope.go` 是 dead code 重複定義，但**不刪除**，保留現狀。
- 未執行 `go build ./...` — 本地 WSL 環境無 Go toolchain。建議在有 Go 環境的機器跑 `go build ./...` + `go vet ./...` + 角色測試。

---

## 4. 安全文件（B 動作）

新增 `SECURITY_NOTES.md`（111 行）：機制鏈路說明 + 已套用修補清單 + 未動到的 sub-group + 行為改變風險（fail-open default branch）+ `Permission()` 已知缺陷 + dead-code 後續建議。

---

## 5. graphify deep-mode 補抽（C 動作）

未跑 `--update` 全 pipeline（incremental detect 把 `graphify-out/obsidian/` 1056 個自寫 notes 誤判為「新文件」）。改採聚焦策略：

派 2 個 deep-mode subagent 對「ActiveRecord 介面 + permission seam + middleware chain」關鍵 22 檔做語意抽取，再以腳本對齊 ID 後合併：

- **+ 11 條 `implements ActiveRecord`**（pure AST 抓不到的 Go 介面隱式實作）
  - `models_sys{api,config,dept,dictdata,dicttype,loginlog,menu,operalog,post,role,user}_generate` → `models_activerecord`
- **+ 9 條 `uses_middleware → permission_permissionaction`**（剛 wire 進去的中介層）
- **+ 2 條 `duplicates`**：
  - `permission_datapermission` → `datascope_datapermission` (conf 0.95)
  - `permission_permission` → `datascope_getdatascope` (conf 0.85)
- **+ 4 個 hyperedges**：CRUD 工廠家族 + DataPermission middleware pipeline + admin router pattern + ActiveRecord implementers
- 合併重複的 `type_activerecord` 與 `models_activerecord` → canonical `models_activerecord`

### 最終圖譜

- 1,725 nodes / 5,422 edges / 73 communities
- 77% EXTRACTED · 23% INFERRED（avg confidence 0.8）
- 新晉 God Node：`getPermissionFromContext()`（21 edges）
- `IndexAction()` betweenness 從 0.038 升至 0.090
- `ActiveRecord` 新登 betweenness 0.061

### 第二階段成本

額外 ~3,300 input / 1,800 output tokens（2 subagent 平行 deep mode）

### 累計成本

15,100 input / 5,000 output tokens（2 runs）

---

## 6. Git 操作摘要

```
main (8ad6a0d)
 ├── graphify-20260506 (29e116f) ─── PR #1 ─── merge ──┐
 │                                                       │
main (3fcbe48 = merge of #1)  ← origin/main fast-forward  │
 │
 └── data-scope-enable-20260506 (a4de839) ── push only, no PR
```

### Branch 1：`graphify-20260506`

- commit `29e116f chore(graphify): add knowledge graph artefacts (graphify-20260506)`
- 2,060 files / +144,707 / -0
- 內容：`graphify-out/` 全部產出（除 env-specific `.graphify_python`）
- **PR #1** https://github.com/miso168net/fork260506-go-admin/pull/1
- 採用 `--merge` 策略合併（與 repo 既有慣例一致）：merge commit `3fcbe48 Merge pull request #1 from miso168net/graphify-20260506`
- mergedAt `2026-05-05T19:42:45Z`

### Branch 2：`data-scope-enable-20260506`

- commit `a4de839 feat(security): enable admin data-scope permission across CRUD routes`
- 13 files / +132 / -12
- 內容：9 router + 3 yml + `SECURITY_NOTES.md`
- 已 push 到 origin
- **未開 PR**（依指示）

---

## 7. graphify-out/ 產出清單

| 檔案 | 內容 |
|---|---|
| `graph.json` | 1725 nodes / 5422 edges 原始圖資料 |
| `graph.html` | 互動式 D3 圖（瀏覽器開啟） |
| `GRAPH_REPORT.md` | 審計報告（社群、cohesion、God Nodes、surprises、suggested questions） |
| `obsidian/` | 1798 篇 notes + `graph.canvas` |
| `cache/` | 語意抽取快取（`--update` 加速用） |
| `manifest.json` | `--update` 增量偵測 manifest |
| `cost.json` | 累計 LLM token 成本 |

`graphify-out/.graphify_python`（含 local pipx 路徑）是環境特定產物，**未** commit 任一分支，僅保留為 working-tree untracked。

---

## 8. 未做 / 後續事項

- [ ] 在有 Go toolchain 的環境執行 `go build ./... && go vet ./...` 驗證 router 變更
- [ ] 部署前檢查 `sys_role.data_scope` 欄位分佈（避免管理員角色被意外限制）
- [ ] 評估是否將 `Permission()` `default: return db` 改為 fail-closed（`return db.Where("1=0")`）以提升 unknown scope 的安全性
- [ ] 評估清除 `app/admin/models/datascope.go` dead code（與 `common/actions/permission.go` 重複）
- [ ] 評估 graphify 圖中提示的弱連結節點（`SysRoleMenu`, `SysApiOrder`, `SysConfigOrder` 等 59 個）— 是文件缺口還是設計上孤立？
- [ ] `data-scope-enable-20260506` 分支若需開 PR 再人工觸發
- [ ] 若需重跑 graphify：`/graphify --update`（cache 已建立，下次只會抽變動檔）

---

## 9. 工具備忘

- graphify 套件名為 `graphifyy`（雙 y，PyPI），預先以 pipx 安裝在 `/home/anew/.local/share/pipx/venvs/graphifyy/`。
- 每個 graphify 階段最後會清理 `graphify-out/.graphify_*.json` 暫存檔，但 `cache/` 與 `manifest.json` 保留供下次 update 使用。
- 沙箱限制：本 session 中遇過 `pip install graphifyy`（已預裝可直接用）與 `rm` 預存檔（datascope.go）兩處被攔，需明確授權才能放行。
