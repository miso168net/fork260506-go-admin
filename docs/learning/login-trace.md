# go-admin 登入鏈路學習導讀

> 一條 `POST /api/v1/login` 從 client 進到 JWT 出來的完整追蹤
>
> 本文件由 SpecKit feature `001-learning-trace-login` 產出。
> Spec 與相關設計檔在 `specs/001-learning-trace-login/`。
>
> **語言**：zh-TW 散文 + 原始程式碼／註解保留原文。

## 目錄

1. [學習目標 & 前置條件](#1-學習目標--前置條件)
2. [用 Docker Compose 把後端跑起來](#2-用-docker-compose-把後端跑起來)
3. [用 curl 登入並拿到 JWT](#3-用-curl-登入並拿到-jwt)
4. [登入鏈路序列圖](#4-登入鏈路序列圖)
5. [逐層追蹤每一段 code](#5-逐層追蹤每一段-code)
6. [Runtime 驗證 recipe](#6-runtime-驗證-recipe)
7. [SDK 邊界](#7-sdk-邊界)
8. [下一步](#8-下一步)

---

## 1. 學習目標 & 前置條件

### 1.1 你讀完這份文件之後可以做到

- 用**一條** Docker 指令把 go-admin 後端跑起來，不需要本機裝 Go、不需要 MySQL、也不需要 `go-admin-ui` 前端。
- 用 `curl` 拿到 `admin/123456` 對應的 JWT，並理解 dev-mode captcha bypass 為什麼能通過。
- 講得出 `POST /api/v1/login` 從 router 進來、到 JWT 出去之間每一層的角色，並指得出對應的 `file:line`。
- 用 runtime 證據（GORM 的 SQL 日誌、JWT claims、`sys_login_log` 資料列）反向驗證自己讀程式碼讀出來的預測。

### 1.2 前置條件（核心只有一項：Docker）

- **Docker**：Docker Desktop（macOS）、Docker Desktop + WSL2（Windows）或 Docker Engine + `compose` plugin（Linux / WSL2）。本文件實測過 Linux 與 WSL2，未驗證純 Windows 路徑。
- 終端機（多數 OS 內建 `curl`，或自行安裝）。
- 數百 MB 磁碟空間放 `golang:1.24-alpine` builder image、`alpine` runtime image、以及 builder 的 module cache。

```text
required: docker (with compose plugin), curl
```

### 1.3 不需要的東西（特別澄清）

專案 README「Ready to work」一節列了 Go、MySQL、Node、`go-admin-ui` 等項目，看起來像必備。**對這份學習文件而言通通不需要**：

- 不需要在本機安裝 **Go toolchain**（多階段 Dockerfile 在 builder stage 內編譯）。
- 不需要 **MySQL**（compose 檔走 SQLite，DB 檔已內嵌進 image）。
- 不需要 **Node / `go-admin-ui` 前端**（直接用 `curl` 打 API）。

### 1.4 時間預算

- 第一次 `docker compose ... up --build`：5–10 分鐘，主要花在抓 base image 與 Go module（網速決定）。
- 之後每次重啟：~30 秒。第一次成功之後，從敲指令到拿到 JWT ≤ 5 分鐘（SC-005）。
- 整份 walkthrough（讀完 + 跑完 + 驗證）：≤ 30 分鐘（SC-001）。

### 1.5 不在本文範圍

Casbin / data-scope 過濾、寫入路徑的 operation log、code-gen 等主題都**不在這份文件**——它們是後續獨立 spec 的題目，第 8 節會列出指引。

---

## 2. 用 Docker Compose 把後端跑起來

### 2.1 一條指令把後端跑起來

在專案根目錄（`fork260506-go-admin/`）下執行：

```bash
docker compose -f docker-compose.learning.yml up --build
```

需要背景執行（讓 terminal 騰出來打 `curl`）就加 `-d`：

```bash
docker compose -f docker-compose.learning.yml up --build -d
```

`-f docker-compose.learning.yml` 是必要的——倉庫根目錄的預設 `docker-compose.yml` 假設你已經 build 好 `go-admin:latest` 並會掛載 `config/settings.yml`（預設指向 MySQL `127.0.0.1:3306`），跑下去會掉進 default-MySQL-config 陷阱。學習流程改用 `docker-compose.learning.yml`，它走 SQLite 不需要外部 DB。

### 2.2 第一次 vs 之後重啟的時間預算

- **第一次 `up --build`**（無快取）：實測約 **2 分鐘** 完成 image 建置與容器啟動，主要花在抓 `golang:1.24-alpine` builder image 與 `go mod download`。網速慢或拉取受限環境（中國大陸 / 公司 proxy）可能拉長到 **5–10 分鐘**，這是 spec 給的最壞情況上限。
- **之後 `up`（有 layer cache）**：約 **30 秒** 內完成，容器本身啟動只要幾秒。
- 啟動成功時 stdout 會出現經典的 banner：

  ```text
  go-admin 2.2.0
  Server run at: http://localhost:8000/
  ```

  看到這兩行就代表 Gin 已經 listen 在 `:8000`。

> 注意：multi-stage build 的 builder stage 需要 CGO（SQLite driver `mattn/go-sqlite3` 依賴 C 編譯器），Dockerfile 已用 `golang:1.24-alpine` + `apk add gcc g++ libc6-compat` 處理好；本機**不需要** `gcc` / `CGO_ENABLED=1`。

### 2.3 Sanity check：HTTP 200 from Swagger UI

容器起來後，另開一個 terminal 確認 8000 port 真的有回應：

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8000/swagger/admin/index.html
```

預期輸出：

```text
200
```

拿到 `200` 就代表 router 已經註冊、static asset server 正常，可以進到第 3 節做 `POST /api/v1/login`。如果拿到 connection refused，先回頭看 compose 那邊的 build log 有沒有報錯。

### 2.4 FR-015 Swagger UI callout（替代但非主線）

go-admin 內建一份 Swagger UI，可以用瀏覽器直接逛所有 endpoint：

- URL：`http://localhost:8000/swagger/admin/index.html`
- 註冊位置：`app/admin/router/sys_router.go:59`（`r.GET("/swagger/admin/*any", ...)`）

它是 **curl 之外的替代探索工具**，但**不是這份學習文件的主要教學介面**——後續第 3、第 5、第 6 節仍以 `curl` 作為標準動作，因為 curl 的請求／回應對照與後面要追的 SQL log、JWT claims 對得起來，Swagger UI 的 form 互動會把 `Authorization` header、captcha 欄位這些細節藏在 UI 之後，不利於「逐層追蹤」。

### 2.5 收掉容器

學習段落結束後，停掉並清掉容器：

```bash
docker compose -f docker-compose.learning.yml down
```

### 2.6 預設不掛 volume：每次 `up` 都是乾淨狀態

`docker-compose.learning.yml` **預設只 bind-mount `./temp` 給 log 用**，SQLite DB 檔留在 container layer 裡，所以每次 `down` + `up --build` 都是全新狀態，跟第 6 節 (`US3`) 的驗證 recipe 假設一致。

如果想讓 `sys_login_log` 之類的資料列在重啟之後仍然留著，把 compose 檔裡那行被註解掉的 `# - ./go-admin-db.db:/go-admin-db.db`（見檔案 11–15 行的 OPT-IN 區塊）打開即可——這是 opt-in 行為，預設關閉。

---

## 3. 用 curl 登入並拿到 JWT

### 3.1 標準登入 curl（dev-mode captcha bypass）

容器跑起來後用這條 **canonical curl** 拿 token——`code:"0"` 與 `uuid:"0"` 是 dev-mode 的 captcha bypass sentinel（FR-007），不需要先打 `GET /api/v1/captcha`；四個欄位都是 `binding:"required"`，少一個 gin 會在進 authenticator 前回 `400`：

```bash
curl -s -X POST http://localhost:8000/api/v1/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"123456","code":"0","uuid":"0"}'
```

### 3.2 實際 response shape（5 個 top-level keys）

`contracts/post-login.md` 只列了 `code` / `expire` / `token`；實測（T004 證據）還多出 `currentAuthority` 與 `success`，**共 5 個 top-level keys**，本節以實測為準：

```json
{
  "code": 200,
  "currentAuthority": "<JWT>",
  "expire": "2126-04-13T12:08:37+08:00",
  "success": true,
  "token": "<JWT>"
}
```

`currentAuthority` 與 `token` 內容相同（antd-pro 風格前端的相容欄位）。**後續所有 `Authorization: Bearer ...` 一律用 `token`**——它是 contract 唯一保證會有的欄位，也是第 5、第 6 節追的對象。

### 3.3 把 JWT 拆開來看 claims

JWT 是三段 base64url 字串用 `.` 連接（header / payload / signature），把第二段解 base64 即可。注意 JWT 用 **URL-safe base64**（`-`/`_` 取代 `+`/`/`，不補 `=`），`base64 -d` 偶爾會抱怨，必要時自己補 `=` 或改用 `tr '_-' '/+' | base64 -d`：

```bash
TOKEN=$(curl -s -X POST http://localhost:8000/api/v1/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"123456","code":"0","uuid":"0"}' | jq -r .token)
echo "$TOKEN" | cut -d. -f2 | base64 -d 2>/dev/null
```

### 3.4 實際 claim shape（8 個 keys）

實測解出來的 payload 如下（claim 對照見 `data-model.md §E4`，6 個業務 claim + 2 個 gin-jwt envelope claim）：

```json
{
  "datascope": "",
  "exp": 4931726917,
  "identity": 1,
  "nice": "admin",
  "orig_iat": 1778090917,
  "roleid": 1,
  "rolekey": "admin",
  "rolename": "系统管理员"
}
```

特別注意 `datascope` 是**空字串**（不是 quickstart 預測的 `"1"`）——`admin` role 的 `data_scope` 欄位在 seed data 裡就是空，下游 data-scope filter 會當「無限制」處理。

### 3.5 為什麼 `code:"0", uuid:"0"` 會通過？

source-of-truth 是 `common/middleware/handler/auth.go:49–62` 的 swagger 註解（FR-017 verbatim）：

```go
// Authenticator 获取token
// @Summary 登陆
// @Description 获取token
// @Description LoginHandler can be used by clients to get a jwt token.
// @Description Payload needs to be json in the form of {"username": "USERNAME", "password": "PASSWORD"}.
// @Description Reply will be of the form {"token": "TOKEN"}.
// @Description dev mode：It should be noted that all fields cannot be empty, and a value of 0 can be passed in addition to the account password
// @Description 注意：开发模式：需要注意全部字段不能为空，账号密码外可以传入0值
// @Tags 登陆
// @Accept  application/json
// @Product application/json
// @Param account body Login  true "account"
// @Success 200 {string} string "{"code": 200, "expire": "2019-08-07T12:45:48+08:00", "token": ".eyJleHAiOjE1NjUxNTMxNDgsImlkIjoiYWRtaW4iLCJvcmlnX2lhdCI6MTU2NTE0OTU0OH0.-zvzHvbg0A" }"
// @Router /api/v1/login [post]
```

兩個條件同時成立才有 bypass：(a) `config/settings.yml` 的 `application.mode` 是 `dev`（compose 檔內嵌設定就是 `dev`）；(b) request 同時帶 `code:"0"` 與 `uuid:"0"`。只中一個會走正常 captcha 驗證並失敗。production 部署請改 `mode: prod`，bypass 自動關閉。

> 想看這條 login 背後打了哪些 SQL（`sys_user` 查詢、`sys_role` lookup、`sys_login_log` 寫入）？第 6 節 `Runtime 驗證 recipe` 會用 GORM SQL log 反向驗證。

---

## 4. 登入鏈路序列圖

> 本節將在 T011 由 `[US2]` 任務填寫。

---

## 5. 逐層追蹤每一段 code

> 本節將在 T012 由 `[US2]` 任務填寫。

---

## 6. Runtime 驗證 recipe

> 本節將在 T016 由 `[US3]` 任務填寫。

---

## 7. SDK 邊界

> 本節將在 T013 由 `[US2]` 任務填寫。

---

## 8. 下一步

> 本節將在 T014 由 `[US2]` 任務填寫。
