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

下圖把第 3 節那條 `curl` 從 client 進到 JWT 出來的全程拆成 11 層 actor，對應 `research.md §R1` 的 file:line 索引；後面 5.x 每一節對應圖上的一個 box。兩塊灰底（`rect rgb(...)`）標記出 **gin-jwt SDK** 的兩道邊界（FR-010），也是第 7 節要重點處理的「不要追進去」的部分。

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant Router
    participant LoginHandler as LoginHandler [SDK]
    participant Authenticator
    participant Captcha as captcha.Verify
    participant GetUser as Login.GetUser
    participant DB
    participant PayloadFunc
    participant TokenGen as TokenGenerator [SDK]
    participant LoginLog as LoginLogToDB

    Client ->> Router: POST /api/v1/login {username,password,code,uuid}
    Router ->> LoginHandler: gin v1 group dispatch (sys_router.go:71)

    rect rgb(235,235,250)
        Note over LoginHandler: SDK boundary (gin-jwt)
        LoginHandler ->> Authenticator: invoke project authenticator callback
    end

    Authenticator ->> Authenticator: bind JSON to Login struct (auth.go:63-107)
    Authenticator ->> Captcha: Verify(uuid, code, true) — clear-after-verify
    Captcha -->> Authenticator: pass (dev-mode 0/0 bypass)
    Authenticator ->> GetUser: GetUser(tx *gorm.DB)

    Note over GetUser,DB: 兩次獨立的 DB round-trip
    GetUser ->> DB: SELECT * FROM sys_user WHERE username = ? AND status = '2'
    DB -->> GetUser: user row
    GetUser ->> GetUser: pkg.CompareHashAndPassword(hash, plain) — bcrypt (login.go:22)
    GetUser ->> DB: SELECT * FROM sys_role WHERE role_id = ?
    DB -->> GetUser: role row

    GetUser -->> Authenticator: (user, role, err)
    Authenticator ->> PayloadFunc: build jwt.MapClaims (auth.go:21-35)
    PayloadFunc -->> Authenticator: claims map

    rect rgb(235,235,250)
        Note over TokenGen: SDK boundary (gin-jwt HS256)
        Authenticator ->> TokenGen: sign claims
        TokenGen -->> Authenticator: signed JWT
    end

    Authenticator ->> LoginLog: enqueue log message (auth.go:77)
    Note over LoginLog: async; row arrives ~50-500ms later via queue consumer (cmd/api/server.go:70)
    LoginLog -->> Authenticator: returns immediately (no inline INSERT)

    Authenticator -->> LoginHandler: (claims, nil)
    LoginHandler -->> Client: 200 {code, currentAuthority, expire, success, token}
```

幾個值得在讀圖時注意的點：

- **兩道 SDK 邊界視覺標記**：圖上兩塊灰底 `rect` 各自包住 `LoginHandler` 與 `TokenGenerator`，分別對應 `research.md §R1` 第 2 與第 9 層。本文件刻意不追進這兩塊內部（FR-010），第 7 節會一次處理它們的責任邊界。
- **`Login.GetUser` 內部兩次獨立 DB round-trip**：先 `sys_user` 撈使用者列，中間夾 `pkg.CompareHashAndPassword` 做 bcrypt 比對（`login.go:22`），通過後才打第二發查 `sys_role`。這兩段在第 6 節的 GORM SQL log 會分別印出兩條 `SELECT`。
- **LoginLog 是 async**：`Authenticator ->> LoginLog` 那一箭只是把訊息塞進 `sdk.Runtime.GetStreamMessage` 的 queue（`auth.go:77`），真正的 INSERT 由 `cmd/api/server.go:70` 註冊的 consumer 之後執行；詳見 `research.md §R2` 與第 6 節驗證 recipe 的 `sleep 1` 說明。
- 想看每一層的 code 細節，請繼續往下到第 5 節的逐層 walkthrough。

---

## 5. 逐層追蹤每一段 code

本節把第 4 節序列圖上的 11 個 actor 一個一個拆開：每一層先用 zh-TW 散文交代它在鏈路裡的角色，接著貼出**精確的 `file:line`** 與**逐字保留**的 ≤15 行程式碼節錄，最後給一段 commentary 把棒子交給下一層。所有 `file:line` 對齊 `research.md §R1` 經 T010 重新驗證後的 catalogue（HEAD `9ad098f`）；如果未來這些行號漂移，重跑 T010 驗證腳本就會抓到。

兩個 SDK 邊界（5.2 `LoginHandler` 與 5.9 `TokenGenerator`）依 FR-010 規定**不追進去**，只給簡短的 stub；要看內部的人請另外開 spec 處理 `fork260506-go-admin-core` 兄弟倉庫。所有程式碼節錄為了保住 source-of-truth 一律保留原始中／英／英中混雜註解（FR-017），不重排空白、不翻譯。

讀完整節之後可以回頭對照第 4 節的 mermaid sequence diagram——每個 5.x 子節對應序列圖上一個 box（或一道 SDK 邊界），雙向比對能幫忙在腦海裡把 11 層黏成一張完整的 control-flow 圖。最後第 6 節會用 runtime SQL log、JWT decode 與 `sys_login_log` row 三條證據反向驗證這 11 層的預測；本節是「讀程式碼預測」、第 6 節是「跑起來實證」，兩節是配對的——讀完一邊一定要去跑另一邊，否則學習迴圈不會閉合。

### 5.1 Route registration — `/api/v1/login` 進門那一行

第一層發生在 Gin engine 啟動時：`sysCheckRoleRouterInit` 把 `/api/v1/login` 註冊到 `v1` 這個 RouterGroup 上，並把 handler 直接指給 gin-jwt SDK 的 `authMiddleware.LoginHandler`。

**值得特別點出來的是：`/login` 是註冊在 `v1` 群組（line 69 開的那個），不是後面 `registerBaseRouter` 裡用 `MiddlewareFunc()` 包過的 `v1auth` 群組**——換言之 login endpoint 本身是**未驗證的（unauthenticated）**，這是合理的（你還沒拿到 token，怎麼可能帶 token 上門），但讀 router 檔時若沒注意分組順序，可能會誤以為 login 也走 JWT middleware。同檔的 `/refresh_token` 也是同理放在 `v1` 不是 `v1auth`，因為 refresh 必須允許過期但仍然合法的 token 進來。

**位置**：`app/admin/router/sys_router.go:71`

```go
	v1 := r.Group("/api/v1")
	{
		v1.POST("/login", authMiddleware.LoginHandler)
		// Refresh time can be longer than token timeout
		v1.GET("/refresh_token", authMiddleware.RefreshHandler)
	}
```

接到 `POST /api/v1/login` 的那一刻，Gin 會把 request 直接交給 `authMiddleware.LoginHandler`——但這個 `LoginHandler` 不是這個 repo 內的程式碼，它住在 sibling repo `fork260506-go-admin-core` 的 gin-jwt SDK 裡。`authMiddleware` 是 `*jwt.GinJWTMiddleware` 型別的指標，在更早一層 `cmd/api/server.go` 裡用 `middleware.AuthInit()` 建出來、把本 repo 的 `Authenticator` / `PayloadFunc` / `Authorizator` / `IdentityHandler` / `Unauthorized` 一次塞進它的 callback 欄位——也就是說 5.1 看到的這條 `v1.POST("/login", authMiddleware.LoginHandler)` 已經是一個「已組裝完成的 SDK handler」，所有專案 callback 在這時已經就位。

換句話說，5.1 這一行**只是把 entrypoint 釘到 router 上**，真正的「登入 orchestration」是 5.2 起步的——SDK 內部的 `LoginHandler` 會依序 callback 到本 repo 的 `Authenticator`、`PayloadFunc`、`LoginResponse`，每一個 callback 對應 5.3 之後的某幾層。下一層（5.2）就是這道 SDK 邊界。

### 5.2 gin-jwt LoginHandler — SDK 邊界（stub）

`authMiddleware.LoginHandler` 是 gin-jwt SDK 提供的「樣板 login orchestration」：它負責呼叫專案註冊的 `Authenticator` callback、把回傳的 `data` 餵給 `PayloadFunc` 產生 claims、把 claims 交給內部的 `TokenGenerator` 簽名，最後呼叫 `LoginResponse` callback 把 JSON 寫回 `c`。本專案不提供自己的 `LoginHandler` 實作——它直接重用 SDK 版。

**SDK boundary**：實際程式碼在 `fork260506-go-admin-core/sdk/pkg/jwtauth/`（兄弟倉庫）。本 spec 不深追進去——FR-010 規定追進去要另外開 spec。如果你想看 gin-jwt 內部，往那邊去。

下一層（5.3）就是 `LoginHandler` 從 SDK 裡 callback 回來這個 repo 的第一站：`Authenticator`。讀者只需記住 5.2 在這條鏈路裡是個「dispatcher」——把 control flow 從 router 交到專案 callback、再把專案回傳值交給 token signer。

### 5.3 Authenticator — 專案 callback 的入口

`Authenticator` 是這個專案登入鏈路的**第一段自家程式碼**——SDK 從 5.2 callback 回來就停在這裡。它的 contract 是：吃 `*gin.Context`、回傳 `(interface{}, error)`；error 為 `nil` 代表通過，回傳的 `interface{}` 之後會被丟給 `PayloadFunc`（5.8 層）轉成 claims。

它做四件事：取 GORM connection、把 JSON body bind 進 `Login` struct、（非 dev mode）跑 captcha verify、呼叫 `Login.GetUser` 撈使用者並驗 bcrypt。`defer LoginLogToDB(...)` 包住整段，確保**成功與失敗都會留下登入紀錄**——這呼應 Constitution Principle V（login log 是 product feature 不是 debug aid）。

**位置**：`common/middleware/handler/auth.go:63–107`

```go
func Authenticator(c *gin.Context) (interface{}, error) {
	log := api.GetRequestLogger(c)
	db, err := pkg.GetOrm(c)
	if err != nil {
		log.Errorf("get db error, %s", err.Error())
		response.Error(c, 500, err, "数据库连接获取失败")
		return nil, jwt.ErrFailedAuthentication
	}

	var loginVals Login
	var status = "2"
	var msg = "登录成功"
	var username = ""
	defer func() {
		LoginLogToDB(c, status, msg, username)
```

（…完整定義見 `common/middleware/handler/auth.go:63–107`，包含 deferred `}()` 收尾、`c.ShouldBind`、captcha 分支、`loginVals.GetUser` 呼叫與最終 `return`。）

注意 `status` 預設 `"2"`、`msg` 預設 `"登录成功"`——**樂觀預設**：只有失敗分支才會把它們改成 `"1"` / 失敗訊息（如 `"数据解析失败"`、`"验证码错误"`、`"登录失败"`），這代表所有成功路徑可以「什麼都不寫」就讓 deferred 的 `LoginLogToDB` 寫對。

另一個值得指出來的是 `defer LoginLogToDB(...)` 的位置——它放在 `c.ShouldBind` 之**前**：哪怕 JSON parse 就失敗、`username` 還是空字串、loginVals 完全沒填，這條 defer 還是會跑——這是 Constitution Principle V「不能省 log」的程式碼層保證。`Authenticator` 也只回 `(nil, jwt.ErrXxx)` 不回專屬錯誤型別，**所有失敗路徑對 SDK 來說等價**（差別只在 `msg`），HTTP response 都是 SDK 預設的 `Unauthorized` callback 產出的同一個 `{code, msg}`。下一層（5.4）就是 `Authenticator` 內部的第一個檢查點：captcha verify。

### 5.4 Captcha verify — `code:"0", uuid:"0"` 的 dev-mode bypass

`Authenticator` 在 bind JSON 完之後、撈 user 之前先做 captcha 驗證。關鍵是**整個 captcha 區塊被一個 `if config.ApplicationConfig.Mode != "dev"` 包住**——也就是說 dev mode 下這段**直接跳過**，這正是第 3 節 `code:"0"`、`uuid:"0"` 為什麼會通過的真正原因（不是 captcha 認得 `0`，而是這條 `if` 沒進去）。

`captcha.Verify` 第三個參數 `true` 是 **clear-after-verify** 旗標：驗證一次就把 store 裡那組 UUID/code 清掉，避免重放。失敗時把 `status` 設 `"1"`、`msg` 設 `"验证码错误"` 並回 `jwt.ErrInvalidVerificationode`；deferred 的 `LoginLogToDB` 會把這次失敗也寫成一筆 login log（Principle V 一致性）。

**位置**：`common/middleware/handler/auth.go:88`

```go
	if config.ApplicationConfig.Mode != "dev" {
		if !captcha.Verify(loginVals.UUID, loginVals.Code, true) {
			username = loginVals.Username
			msg = "验证码错误"
			status = "1"

			return nil, jwt.ErrInvalidVerificationode
		}
	}
```

（注意原始碼 typo `ErrInvalidVerificationode`——少一個 `C`，verbatim 保留。FR-016 規定不修。）兩個觀察值得記下來：第一，這個 captcha bypass 的判斷面**不在 captcha 函式內**，而是 caller 端的一個外殼 `if`——意思是 `captcha.Verify` 本身在 dev/prod 行為一致，bypass 是「不呼叫 verify」做到的，不是「verify 對 0/0 開後門」。第二，`mode` 設定值來自 `config.ApplicationConfig.Mode`，最終讀的是 `settings.yml` 的 `application.mode`——`docker-compose.learning.yml` 內嵌的設定就是 `dev`，所以本 spec 的 curl 才能跳過 captcha。production 部署 `mode: prod` 時這個 `if` 會走真實 verify、`code:"0"` 會被拒絕。下一層（5.5）就是 captcha 通過之後 `Authenticator` 真正進到「我是誰」的第一步：把 request 收進 `Login` struct、再用 `Login.GetUser` 打 GORM。

### 5.5 Login struct + Login.GetUser — request 解析與第一發 SQL

第 5 層其實是兩件相鄰的事，住在另一個檔 `login.go`。先有 `Login` struct（第 9–14 行），它是 `c.ShouldBind` 的目標型別——四個欄位都加了 `binding:"required"`，少一個會在 5.3 的 `c.ShouldBind` 那裡直接回 `400`（連 captcha 都不會跑到）。

然後是 method `GetUser(tx *gorm.DB)`（第 16–33 行）——它把整個「驗使用者」流程封裝起來：先查 `sys_user`（第一發 SQL），通過再 bcrypt 比密碼（5.6 層），最後查 `sys_role`（5.7 層）。把這段拉到 method 裡的好處是：`Authenticator` 只看到 `(user, role, err)` 三個回傳值，不需要知道內部走了幾條 SQL、bcrypt 在哪一行——對照本節要展示的「逐層」概念，這個封裝邊界正好是 5.5 / 5.6 / 5.7 三層的拆分點。

**位置**：`common/middleware/handler/login.go:9–14`（struct）與 `:16–33`（`GetUser` method）

```go
type Login struct {
	Username string `form:"UserName" json:"username" binding:"required"`
	Password string `form:"Password" json:"password" binding:"required"`
	Code     string `form:"Code" json:"code" binding:"required"`
	UUID     string `form:"UUID" json:"uuid" binding:"required"`
}

func (u *Login) GetUser(tx *gorm.DB) (user SysUser, role SysRole, err error) {
	err = tx.Table("sys_user").Where("username = ?  and status = '2'", u.Username).First(&user).Error
	if err != nil {
		log.Errorf("get user error, %s", err.Error())
		return
	}
```

（…完整定義延伸到 line 33；剩下的 bcrypt 與 role lookup 分別在 5.6、5.7 層拆。）

第一發 SQL 的 WHERE 條件 `username = ? and status = '2'` 值得標記：`status = '2'` 不是布林、不是 typo——seed data 約定 `'2'` 表示**啟用中**，停權使用者（`status = '1'`）被軟性過濾掉、連 user row 都不會回。從 caller 角度看，**「使用者不存在」與「使用者被停權」回的 error 一模一樣**（`gorm.ErrRecordNotFound`），這是刻意的設計：避免外部探測者用回應差異區別「帳號不存在 vs 帳號被停」做帳號列舉。

`First` 而非 `Find` 也很關鍵——`First` 找不到會回 `gorm.ErrRecordNotFound`，`Find` 找不到只會回空 slice 不算 error，這條鏈路要的是前者，因為錯誤分支倚賴 `if err != nil` 跳脫。另注意 `tx.Table("sys_user")` 是用顯式 table name 而非用 model 推斷；這跟本 repo 多數 repository 使用 `db.Model(&SysUser{})` 風格不同，是個小一致性瑕疵但不影響行為。下一層（5.6）就是 user row 撈出來之後的第一個檢查：bcrypt。

### 5.6 Bcrypt password compare — 沒回 row 不算錯，密碼錯才算錯

`pkg.CompareHashAndPassword` 是個 thin wrapper：它把 `user.Password`（DB 撈出來的 bcrypt hash）跟 `u.Password`（client 傳上來的明碼）比對，內部呼叫 `golang.org/x/crypto/bcrypt`。bcrypt 是 adaptive cost-factor hash，本身就是 timing-safe，不需要再加 `subtle.ConstantTimeCompare`。

注意這一行會回傳 `_, err`——第一個回傳值（rehash 後的 hash）被丟掉，只用 `err` 做控制流，**不會**把舊 hash 升級寫回 DB（這是 SDK 自己的設計取捨，超出本 spec 範圍 FR-016）。也就是說即使將來 cost factor 從 10 提升到 12，舊 user 的 hash 仍會以舊 cost 留存——他們需要重設密碼才會升級。這算個技術債，但屬於另一個 spec 的題目。

**位置**：`common/middleware/handler/login.go:22`

```go
	_, err = pkg.CompareHashAndPassword(user.Password, u.Password)
	if err != nil {
		log.Errorf("user login error, %s", err.Error())
		return
	}
```

bcrypt 比對通過之後，`err` 仍是 `nil`，`GetUser` 才會繼續往下打第二發 SQL。bcrypt 的 cost factor 由 hash 字串本身編進去（例如 `$2a$10$...` 第二個 `$` 後的 `10` 就是 cost），由建立帳號當時 `pkg.HashPassword` 決定；驗證階段會自動用同一 cost——這一層完全不知道也不在意 cost，內部由 SDK 處理。

bcrypt 的演算法細節（Blowfish-derived KDF、salt 內嵌、constant-time 比較）out of scope，FR-016 規定不深追。但要記住的設計觀察是：這個 compare 是**鏈路裡唯一的 CPU-bound 動作**——bcrypt 慢是 feature 不是 bug，cost=10 大概是 ~50ms，這就是登入吞吐率的事實上限。下一層（5.7）就是：使用者通過了，那他的角色是誰？

### 5.7 GORM role lookup — 第二發 SQL，用 user.RoleId 查 sys_role

`GetUser` 在 bcrypt 通過後立刻打第二發 GORM 查詢：`SELECT * FROM sys_role WHERE role_id = ?`。這一發跟 5.5 的 user 查詢是**兩次獨立的 round-trip**——沒做 JOIN——這對讀第 6 節 SQL log 的人很重要：你會看到兩條 `SELECT ... FROM sys_user ...` 與 `SELECT ... FROM sys_role ...`，分別對應 5.5 與 5.7。`role.DataScope` 與 `role.RoleKey` 等欄位在 5.8 PayloadFunc 會被讀進 JWT claims，所以這條 query 不只是「補資料」，它是**簽 token 的前置依賴**。

**位置**：`common/middleware/handler/login.go:27`

```go
	err = tx.Table("sys_role").Where("role_id = ? ", user.RoleId).First(&role).Error
	if err != nil {
		log.Errorf("get role error, %s", err.Error())
		return
	}
	return
}
```

（注意 `"role_id = ? "` 結尾多了一個空白，verbatim 保留。GORM 不會 care，但讀程式碼時值得標記。）

`GetUser` 走完 return 之後，回到 5.3 `Authenticator`：第 96–100 行那段 `if e == nil { return map[string]interface{}{"user": sysUser, "role": role}, nil }` 把 `(user, role)` 包成 map 傳回 SDK。**沒做 JOIN 是刻意的**：兩條 SELECT 在 SQL log 裡分開出現比較容易讀懂、bcrypt 比較失敗時也不會浪費一次 sys_role 查詢；缺點是多一個 round-trip，但 `sys_user` / `sys_role` 表都很小（dev 環境一張幾十 row），多出的延遲忽略不計。

另一個值得 flag 的是 `user.RoleId` 是個單值——本 repo 的資料模型假設**一個 user 只屬於一個 role**（不是 many-to-many）。這在多角色系統的設計取捨上是個重要簡化；要做多角色得把這條 query 改成 `Where("role_id IN ?", ids)` 並把 5.8 PayloadFunc 的 `roleid` claim 改成 array——這是另一個 spec 的題目。SDK 再把這個 map 餵給下一層（5.8）：`PayloadFunc`。

### 5.8 PayloadFunc — 把 (user, role) 攤平成 JWT MapClaims

`PayloadFunc` 是 gin-jwt SDK 在簽 token 之前 callback 的最後一站：SDK 把 5.3 `Authenticator` 回傳的 `interface{}`（實際是 `map[string]interface{}{"user":..., "role":...}`）丟進來，由本層產出 `jwt.MapClaims`——也就是最終會被 base64 編進 JWT payload 的鍵值對。

`IdentityKey` / `RoleIdKey` / `RoleKey` / `NiceKey` / `DataScopeKey` / `RoleNameKey` 這些 key 名字是 SDK 公開常數（在 `fork260506-go-admin-core` 那邊定義），對應到第 3.4 節實測解出的 `identity` / `roleid` / `rolekey` / `nice` / `datascope` / `rolename` 六個業務 claim。`nice` 看起來莫名其妙，原文是 `Nickname`／中文「昵稱」之意——這是專案歷史命名選擇，FR-016 規定不重命名。

**位置**：`common/middleware/handler/auth.go:21–35`

```go
func PayloadFunc(data interface{}) jwt.MapClaims {
	if v, ok := data.(map[string]interface{}); ok {
		u, _ := v["user"].(SysUser)
		r, _ := v["role"].(SysRole)
		return jwt.MapClaims{
			jwt.IdentityKey:  u.UserId,
			jwt.RoleIdKey:    r.RoleId,
			jwt.RoleKey:      r.RoleKey,
			jwt.NiceKey:      u.Username,
			jwt.DataScopeKey: r.DataScope,
			jwt.RoleNameKey:  r.RoleName,
		}
	}
	return jwt.MapClaims{}
}
```

注意兩個 `_` 忽略 type assertion 失敗——若上游把 user/role 包錯，這裡會塞**零值**（`UserId=0`、`RoleKey=""`）而不是噴錯。實務上 `Authenticator` 一定會回正確的 map（5.3 第 100 行那個 literal），所以 type assertion 失敗只會發生在 SDK 內部資料路徑改 contract 的時候——對 learner 而言這是個「靜默崩潰」風險點，但短期不會踩到。

也注意這六個 claim 全部都是**業務 claim**（`identity` / `roleid` / `rolekey` / `nice` / `datascope` / `rolename`），第 3.4 節解出的 `exp` 與 `orig_iat` **不在這裡**設定——它們是 SDK 簽 token 時加上的「envelope claim」，由 5.9 層處理。`PayloadFunc` 是純函式（沒有 side effect），這代表它可以被測試獨立呼叫驗 claim shape；也意味著「JWT 漏出哪些 claim」這個安全問題的 single source of truth 就是這 12 行——任何要新增／刪除 claim 的 PR 都必須改這裡。

### 5.9 JWT TokenGenerator — SDK 邊界（stub）

SDK 拿到 5.8 `PayloadFunc` 回傳的 `MapClaims` 之後，會用設定檔指定的 `SigningAlgorithm`（go-admin 預設 HS256）與 `Key`（`settings.jwt.secret`）把 claims 簽成最終的 JWT 字串，並補上 `exp`（過期時間）與 `orig_iat`（發 token 當下的 unix epoch）這兩個 envelope claim。簽完的 JWT 字串就是第 3.2 節 response 的 `token` 與 `currentAuthority` 欄位。

**SDK boundary**：實際程式碼在 `fork260506-go-admin-core/sdk/pkg/jwtauth/`（兄弟倉庫）。本 spec 不深追進去——FR-010 規定追進去要另外開 spec。如果你想看 gin-jwt 內部，往那邊去。

下一層（5.10）切回本 repo：`Authenticator` 在 return 前 deferred 跑的 `LoginLogToDB`——它是這條鏈路的**寫路徑副作用**，而且是 async。順序上這層 token 已經簽完、SDK 馬上要呼叫 `LoginResponse` 把 JSON 寫回 client，所以 5.10 的 log 寫入跟 client 收到 response 之間的時間競賽從這一刻開始。

### 5.10 LoginLog emission — 三檔聯動的 async write（critical wrinkle）

這一層是整條鏈路裡**最容易誤讀的一段**，也是第 6 節驗證 recipe 為什麼要 `sleep 1` 的根本原因。它牽涉**三個檔案、三個動作**：

1. `auth.go:77`——`Authenticator` 的 `defer` 呼叫 `LoginLogToDB`，把日誌封成 message **塞進 in-memory queue 就回**，不打 INSERT。
2. `cmd/api/server.go:70`——服務啟動時把 `models.SaveLoginLog` 註冊成 `global.LoginLog` topic 的 consumer。
3. `app/admin/models/sys_login_log.go:46`——consumer side 真正做 `db.Create(&l)` 寫 SQLite/MySQL 的地方。

**Critical wrinkle**：因為 (1) 跟 (3) 之間隔了一個 `go queue.Run()` goroutine，**HTTP response 會在 row 真正落地前返回**（實測 ~50–500ms 延遲）。沒讀過 (2)、(3) 的人光看 `auth.go:77` 會誤以為 `LoginLogToDB` 是同步 `Create`，然後在第 6 節 `SELECT * FROM sys_login_log` 抓不到 row 時懷疑驗證腳本壞了。它沒壞——是 queue 還沒 flush。

這個 async 設計呼應 Constitution **Principle V**（登入日誌是 load-bearing product feature）：登入路徑的延遲不應該被 log 寫入拖到，所以走 queue 是正確的；但 Principle V 同時要求 log 「不可省」，因此這條 queue 必須是可靠的（記憶體中 in-process，不會掉訊息）而非「best-effort fire-and-forget」——失敗會印 errorf 但不會 retry，這是個取捨點。對 learner 而言，必須事先知道這件 async 事，否則第 6 節驗證會被 race condition 騙到。

**位置 1**：`common/middleware/handler/auth.go:77`（queue 投遞）

```go
	defer func() {
		LoginLogToDB(c, status, msg, username)
	}()
```

`LoginLogToDB` 函式本體在同檔 `:109–141`，第一行就是 `if !config.LoggerConfig.EnabledDB { return }`——這意味著本機如果沒開 `enableddb: true`，這條 log 連 queue 都不進；本 spec 在 `Dockerfile.learning` 那邊已經把它預設打開（見 `research.md §R3`）。函式內部呼叫 `sdk.Runtime.GetMemoryQueue(c.Request.Host)` 拿到 queue handle、用 `sdk.Runtime.GetStreamMessage("", global.LoginLog, l)` 把 dict 包成 message，然後 `q.Append(message)` 投遞——全程沒有 `db.Create`，也沒有等 ack。如果 queue.Append 失敗（極罕見），會 `log.Errorf` 一行就略過，不影響 HTTP response。

**位置 2**：`cmd/api/server.go:70`（consumer 註冊）

```go
	//注册监听函数
	queue := sdk.Runtime.GetMemoryQueue("")
	queue.Register(global.LoginLog, models.SaveLoginLog)
	queue.Register(global.OperateLog, models.SaveOperaLog)
	queue.Register(global.ApiCheck, models.SaveSysApi)
	go queue.Run()
```

`queue.Register` 把 topic（`global.LoginLog`）跟 handler（`models.SaveLoginLog`）綁起來；`go queue.Run()` 起一個 goroutine 開始消化 queue。注意 `LoginLogToDB` 那邊用的是 `GetMemoryQueue(c.Request.Host)`（帶 host 當 prefix），而這邊註冊用的是 `GetMemoryQueue("")`（空 prefix）；它們之所以還是同一個 queue，是因為 SDK 的 `GetMemoryQueue` 對相同 host 回傳同一 instance、空 prefix 也指向同一個全域 queue（內部實作；FR-016 不深追）。

**位置 3**：`app/admin/models/sys_login_log.go:46`（consumer 真正寫 DB）

```go
// SaveLoginLog 从队列中获取登录日志
func SaveLoginLog(message storage.Messager) (err error) {
	//准备db
	db := sdk.Runtime.GetDbByKey(message.GetPrefix())
	if db == nil {
		err = errors.New("db not exist")
		log.Errorf("host[%s]'s %s", message.GetPrefix(), err.Error())
		return err
	}
```

（…完整定義到 `:72`，含 `json.Marshal` / `Unmarshal` 與最終 `db.Create(&l).Error`。）

`SaveLoginLog` 的工作流是：(a) 用 `message.GetPrefix()` 拿到對應 db handle，(b) 把 message 的 values 走一輪 `json.Marshal` → `json.Unmarshal` 灌進 `SysLoginLog` struct，(c) 走 `db.Create` 寫進 `sys_login_log` 表。**整段在 consumer goroutine 上跑**，跟 HTTP request goroutine 完全脫鉤。

第 6 節驗證 recipe 的 `sleep 1` 就是在等這段 finish——實測大約 50–500ms 內會看到 row，但「上限」沒有保證；極端高併發或 SQLite 被 lock 時可能更久。寫入失敗也只會印 log（`log.Errorf("db create error, ...")`）而不會 retry，這是**已知 trade-off**（FR-016 不修），但對 learner 重要——你若觀察到 row 沒進，先檢查 docker logs 有沒有那行 errorf。`json.Marshal` → `json.Unmarshal` 的 round-trip 看起來像 over-engineering，實際是 SDK 的 message envelope 抽象——它讓同一個 queue handler 可以處理不同 topic 的 schema，代價是每次寫 log 多兩次 JSON 編碼。

下一層（5.11）回到 `Authenticator` 真的 return 之後，SDK 把 token 與 metadata 組成 HTTP response 寫回 client 的部分。

### 5.11 Response assembly — 200 OK 與失敗時的 fallback

`Authenticator` return `(map, nil)` 之後，SDK 內部的 `LoginResponse` callback（gin-jwt 預設實作，未被本 repo override）會把簽好的 JWT 與 expire 時間組成 JSON 寫回——這對應第 3.2 節實測的 5-key response（`code` / `currentAuthority` / `expire` / `success` / `token`）。

本 repo `auth.go` 內並**沒有**自己的 `LoginResponse` 實作；同檔最接近的兩個 `c.JSON` 反而是登出（`LogOut`）成功與授權失敗（`Unauthorized`）的 fallback——它們不是 login 成功路徑的回應，但研究 catalogue 把它們列為「response assembly 的 sibling reference」，方便比對 SDK 預設與專案 override 的差異。

**位置**：`common/middleware/handler/auth.go:155`（`LogOut` 成功時的 c.JSON）

```go
func LogOut(c *gin.Context) {
	LoginLogToDB(c, "2", "退出成功", user.GetUserName(c))
	c.JSON(http.StatusOK, gin.H{
		"code": 200,
		"msg":  "退出成功",
	})

}
```

**位置**：`common/middleware/handler/auth.go:178`（`Unauthorized` fallback）

```go
func Unauthorized(c *gin.Context, code int, message string) {
	c.JSON(http.StatusOK, gin.H{
		"code": code,
		"msg":  message,
	})
}
```

兩個 `c.JSON` 都是 `{code, msg}` 的二鍵 shape——跟第 3.2 節登入成功的 5-key shape（`code` / `currentAuthority` / `expire` / `success` / `token`）**明顯不同**。

差距正好證實一件事：登入成功的 JSON **不是**從這個檔產出，而是從 5.9 那道 SDK 邊界後面、gin-jwt 的 `LoginResponse` 預設實作產出的。要看那一段，FR-010 規定請另外開 spec 進到 `fork260506-go-admin-core/sdk/pkg/jwtauth/`。

`LogOut`（`auth.go:153–160`）多做了一件事：在回 JSON 之前先呼叫 `LoginLogToDB(c, "2", "退出成功", user.GetUserName(c))`——也就是登出也算一筆 login log，跟 5.10 走同一條 async queue。`Unauthorized`（`auth.go:177–182`）則是 SDK 在驗 token 失敗時 callback 用的 fallback；它跟 login 成功路徑沒直接關係，列在這一層只是因為它是 auth.go 內最接近「response assembly 形狀」的兩段 code，方便讀者比對 SDK 預設與專案 override 的差異。

到這裡，client 收到 `200 {code, currentAuthority, expire, success, token}`，整條 `POST /api/v1/login` 從 5.1 router 到 5.11 response 的鏈路結束。第 6 節會用 GORM SQL log、JWT decode、`sys_login_log` row 三條 runtime 證據反向驗證上面 11 層每一層的預測——例如「sys_user 真的查了一發、sys_role 真的查了一發、JWT 真的有 6 個業務 claim、`sys_login_log` 真的會出現一 row（但要等 50–500ms）」這四件事。

---

## 6. Runtime 驗證 recipe

> 本節將在 T016 由 `[US3]` 任務填寫。

---

## 7. SDK 邊界

> 本節將在 T013 由 `[US2]` 任務填寫。

---

## 8. 下一步

> 本節將在 T014 由 `[US2]` 任務填寫。
