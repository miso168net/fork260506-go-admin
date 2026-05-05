# SECURITY_NOTES — go-admin data scope (數據權限)

來源：2026-05-06 graphify 圖譜分析後對 `IndexAction` cross-community bridge 的深入追蹤。

## 摘要

`enabledp` 開關 + `PermissionAction()` middleware + `Permission()` GORM scope 三者構成 go-admin 的 row-level data scope 機制。原始狀態下這套機制在大多數路徑上**雙重關閉**：

1. `config/settings.yml` 等 3 份 yml 將 `enabledp` 預設為 `false`。
2. 13 個 admin 路由群組（`/sys-api`, `/config`, `/dept`, `/dict`, `/sys-login-log`, `/menu`, `/sys-opera-log`, `/post`, `/role` 等）沒有掛 `actions.PermissionAction()` middleware，因此 `c.Set(PermissionKey, p)` 從未執行。
3. `app/admin/apis/sys_*.go` 內部呼叫 `actions.GetPermissionFromContext(c)` 拿到的會是 zero-value `*DataPermission{DataScope:""}`，傳入 `Permission(tableName, p)` 後落入 `switch` 的 `default` 分支，等同**不過濾**。

加上 `enabledp:false`，所有 user 看到所有資料。

## 機制鏈路

```
Router middleware:  authMiddleware → AuthCheckRole → PermissionAction()
                                                            │  user.GetUserIdStr(c)
                                                            ▼
                                          newDataPermission(db, userId)
                                            (JOIN sys_user × sys_role)
                                                            │
                                                            ▼ c.Set(PermissionKey, p)
                                                       gin.Context
                                                            │
API handler / service:                                      ▼
   p := actions.GetPermissionFromContext(c)  ──── (若無上方 middleware → zero-value)
   service.GetPage(req, p, ...)
                                                            │
                                                            ▼
                          db.Scopes(MakeCondition, Paginate, Permission(table, p))
                                                            │
                                            switch p.DataScope:
                                              "1" / 預設 → 不過濾
                                              "2" → role_dept JOIN
                                              "3" → 本部門
                                              "4" → 本部門 + 子層 (dept_path LIKE)
                                              "5" → 僅本人 (create_by = user_id)
```

`Permission()`（`common/actions/permission.go:62`）尚有最外層 `if !config.ApplicationConfig.EnableDP { return db }` 的全域 kill-switch。

## 已套用的修補

### 1. 開啟 `enabledp` 預設值

| 檔案 | 變更 |
| --- | --- |
| `config/settings.yml` | `enabledp: false` → `true` |
| `config/settings.full.yml` | `enabledp: false` → `true` |
| `config/settings.sqlite.yml` | `enabledp: false` → `true` |
| `config/settings.demo.yml` | （原本就是 `true`，未動） |

### 2. 為 9 個 admin router group 補上 `.Use(actions.PermissionAction())`

每個檔案同時新增 import `"go-admin/common/actions"`：

| 檔案 | 路由 group |
| --- | --- |
| `app/admin/router/sys_api.go` | `/sys-api` |
| `app/admin/router/sys_config.go` | `/config` |
| `app/admin/router/sys_dept.go` | `/dept` |
| `app/admin/router/sys_dict.go` | `/dict` |
| `app/admin/router/sys_login_log.go` | `/sys-login-log` |
| `app/admin/router/sys_menu.go` | `/menu` |
| `app/admin/router/sys_opera_log.go` | `/sys-opera-log` |
| `app/admin/router/sys_post.go` | `/post` |
| `app/admin/router/sys_role.go` | `/role` |

未動到的 sub-group（保留無權限過濾）：
- `sys_config.go`：`/configKey`、`/app-config`、`/set-config`（key 查詢、APP 公開設定）。
- `sys_dept.go`：`/deptTree`（樹形查詢介面）。
- `sys_dict.go`：`/dict-data`（option-select 選項用）。
- `sys_menu.go`：`/menurole`（菜單角色查詢）。
- `sys_role.go`：`/role-status`、`/roledatascope`（角色狀態與 scope 設定）。

這些是 read-only 列舉或設定端點，預設不掛 row-level scope；如有需要再評估。

### 3. 待處理：刪除 dead code `app/admin/models/datascope.go`

該檔內定義了第二份 `DataPermission` struct（`app/admin/models/datascope.go:12`，與 `common/actions/permission.go:15` 完全相同欄位），以及 `(*DataPermission).GetDataScope` 方法（67 行）。grep 結果顯示**整個 repo 沒有任何外部 caller**，且檔末還有一段被註解掉的 `DataScopes()` 函式。

> 此檔在沙箱環境中暫未刪除（自動環境拒絕 `rm` 預存檔的動作），需用戶以明確指令授權移除。

## 風險與後續

### 行為改變

啟用 `enabledp:true` 後，**任何角色 `data_scope` 欄位為 `"2"`-`"5"` 的 user 都會被過濾資料**。若資料庫中歷史角色的 `data_scope` 欄位值並非預期：
- `"1"` 或空字串會落入 `Permission()` 的 `default` 分支 → 不過濾。
- `"2"`-`"5"` 將套用對應 SQL where 子句。

部署到既有環境前應先檢查 `sys_role.data_scope` 欄位的分佈，避免管理員角色被意外限制。

### `Permission()` 的 default 行為

`common/actions/permission.go:76-78` 的 `default: return db` 表示**未知 DataScope 值 = 不過濾**。從安全角度這是 fail-open，若需要 fail-closed（unknown scope = 拒絕資料）應改成 `return db.Where("1=0")`。本次未變更該行為以避免破壞 admin（DataScope `"1"` 通常表示「全部」）。

### 兩個 `DataPermission` struct 的清理

刪除 `app/admin/models/datascope.go` 後，僅保留 `common/actions/permission.go` 的 canonical 版本。對 build / runtime 無影響（已驗證零外部 caller）。

## 驗證

本機環境無 Go toolchain（`go: command not found`），未執行 `go build ./...` 驗證。建議在有 Go 1.x 的機器上：
```
go build ./...
go vet ./...
```
若 build 過，再以 `mode: dev` 啟動 server，登入不同角色實測 `/api/v1/sys-api`、`/api/v1/role` 等清單接口的資料範圍。
