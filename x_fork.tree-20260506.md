# 專案檔案結構與用途註解(2026-05-06 snapshot)

本檔案對應 `tree` 命令在 `fork260506-go-admin/` 根目錄的輸出,
為每個檔案/目錄補上一行用途註解,作為閱讀程式碼前的索引。

- 命名沿用 `x_fork.graphify-20260506.md` 的「snapshot 日期」慣例。
- `graphify-out/cache/` 與 `graphify-out/obsidian/` 分別有約 190 與 1500
  個自動生成的檔案,於目錄層級一句話帶過,不逐檔列出。
- `static/form-generator/js/*.js` 為前端 form-builder 的預編譯 vendor bundle
  (~1600 個 function,graphify god nodes 都在這),不展開內部結構。
- 13 個 admin 資源(api/config/dept/dict_data/dict_type/login_log/menu/
  opera_log/post/role/user 等)各有 4 層 sys_*.go 檔(apis/models/router/
  service[/dto]),逐一附註用途。

```
.
├── CLAUDE.md                                    # 給 Claude Code 看的專案指令(SpecKit START/END 區塊 + graphify 規則)
├── Dockerfile                                   # 上游單階段 Dockerfile(假設 ./main 已 build,COPY 進 alpine)
├── Dockerfilebak                                # 舊版 Dockerfile 備份(歷史殘留,不參與流程)
├── LICENSE.md                                   # MIT License (Copyright go-admin-team)
├── Makefile                                     # build / build-sqlite / build-linux / run / stop targets
├── README.Zh-cn.md                              # 上游簡中 README(功能介紹 + 上手教學連結)
├── README.md                                    # 上游英文 README(同上鏡像)
├── _config.yml                                  # Jekyll/GitHub Pages 設定(go-admin.dev 站台用)
│
├── app/                                         # 業務模組(admin / jobs / other)
│   ├── admin/                                   # 後台管理:13 個資源 × 4 層架構
│   │   ├── apis/                                # gin handler 層(MakeContext → Bind → service → Custom)
│   │   │   ├── captcha.go                       # GET /api/v1/captcha 圖形驗證碼產生
│   │   │   ├── go_admin.go                      # GET /go-admin/info 專案 banner endpoint
│   │   │   ├── sys_api.go                       # /api/v1/sys-api CRUD handler
│   │   │   ├── sys_config.go                    # /api/v1/sys-config CRUD handler
│   │   │   ├── sys_dept.go                      # /api/v1/sys-dept CRUD handler(部門樹)
│   │   │   ├── sys_dict_data.go                 # /api/v1/sys-dict-data CRUD handler(字典資料)
│   │   │   ├── sys_dict_type.go                 # /api/v1/sys-dict-type CRUD handler(字典類型)
│   │   │   ├── sys_login_log.go                 # /api/v1/sys-login-log read-only handler
│   │   │   ├── sys_menu.go                      # /api/v1/sys-menu CRUD handler(選單樹)
│   │   │   ├── sys_opera_log.go                 # /api/v1/sys-opera-log 操作日誌 read-only
│   │   │   ├── sys_post.go                      # /api/v1/sys-post CRUD handler(崗位)
│   │   │   ├── sys_role.go                      # /api/v1/sys-role CRUD handler(角色 + 權限指派)
│   │   │   └── sys_user.go                      # /api/v1/sys-user CRUD handler(使用者 + 重設密碼)
│   │   ├── models/                              # GORM model 層(對應 sys_* tables)
│   │   │   ├── casbin_rule.go                   # casbin_rule table model(runtime 版本,Casbin policy 來源)
│   │   │   ├── datascope.go                     # data-scope 過濾 helper(部門範圍計算)
│   │   │   ├── initdb.go                        # InitDb() — DB 初始化 / migration 入口
│   │   │   ├── sys_api.go                       # sys_api table 對應 GORM struct
│   │   │   ├── sys_config.go                    # sys_config table 對應 GORM struct
│   │   │   ├── sys_dept.go                      # sys_dept table 對應 GORM struct
│   │   │   ├── sys_dict_data.go                 # sys_dict_data table 對應 GORM struct
│   │   │   ├── sys_dict_type.go                 # sys_dict_type table 對應 GORM struct
│   │   │   ├── sys_login_log.go                 # sys_login_log table 對應 GORM struct + SaveLoginLog queue consumer
│   │   │   ├── sys_menu.go                      # sys_menu table 對應 GORM struct
│   │   │   ├── sys_opera_log.go                 # sys_opera_log table 對應 GORM struct
│   │   │   ├── sys_post.go                      # sys_post table 對應 GORM struct
│   │   │   ├── sys_role.go                      # sys_role table 對應 GORM struct(含 DataScope 欄位)
│   │   │   └── sys_user.go                      # sys_user table 對應 GORM struct(含 Encrypt() bcrypt)
│   │   ├── router/                              # 路由註冊層(把 handler 掛到 v1 group)
│   │   │   ├── init_router.go                   # 集中註冊所有 admin 路由(被 cmd/api/server.go 呼叫)
│   │   │   ├── router.go                        # gin engine 共用 helper
│   │   │   ├── sys_api.go                       # registerSysApiRouter()
│   │   │   ├── sys_config.go                    # registerSysConfigRouter()
│   │   │   ├── sys_dept.go                      # registerSysDeptRouter()
│   │   │   ├── sys_dict.go                      # dict_data + dict_type 兩資源「共用」的路由檔
│   │   │   ├── sys_login_log.go                 # registerSysLoginLogRouter()
│   │   │   ├── sys_menu.go                      # registerSysMenuRouter()
│   │   │   ├── sys_opera_log.go                 # registerSysOperaLogRouter()
│   │   │   ├── sys_post.go                      # registerSyPostRouter()(注意 typo SyPost)
│   │   │   ├── sys_role.go                      # registerSysRoleRouter()
│   │   │   ├── sys_router.go                    # 系統級路由:login(line 71) / refresh_token / logout / swagger UI / ws
│   │   │   └── sys_user.go                      # registerSysUserRouter()
│   │   └── service/                             # 商業邏輯層(GORM query + 聚合)
│   │       ├── dto/                             # JSON DTOs(GetReq/InsertReq/UpdateReq/DeleteReq/GetPageReq)
│   │       │   ├── sys_api.go                   # SysApi 五個 Req DTOs
│   │       │   ├── sys_config.go                # SysConfig 五個 Req DTOs
│   │       │   ├── sys_dept.go                  # SysDept 五個 Req DTOs
│   │       │   ├── sys_dict_data.go             # SysDictData 五個 Req DTOs
│   │       │   ├── sys_dict_type.go             # SysDictType 五個 Req DTOs
│   │       │   ├── sys_login_log.go             # SysLoginLog 五個 Req DTOs
│   │       │   ├── sys_menu.go                  # SysMenu 五個 Req DTOs
│   │       │   ├── sys_opera_log.go             # SysOperaLog 五個 Req DTOs
│   │       │   ├── sys_post.go                  # SysPost 五個 Req DTOs
│   │       │   ├── sys_role.go                  # SysRole 五個 Req DTOs
│   │       │   └── sys_user.go                  # SysUser 五個 Req DTOs(含 ResetSysUserPwdReq)
│   │       ├── sys_api.go                       # SysApi service(GORM CRUD + 業務分支)
│   │       ├── sys_config.go                    # SysConfig service(含 GetWithKey 快取讀)
│   │       ├── sys_dept.go                      # SysDept service(部門樹遞迴)
│   │       ├── sys_dict_data.go                 # SysDictData service
│   │       ├── sys_dict_type.go                 # SysDictType service
│   │       ├── sys_login_log.go                 # SysLoginLog service(read-only)
│   │       ├── sys_menu.go                      # SysMenu service(選單樹遞迴 + 角色過濾)
│   │       ├── sys_opera_log.go                 # SysOperaLog service(read-only)
│   │       ├── sys_post.go                      # SysPost service
│   │       ├── sys_role.go                      # SysRole service(指派 menu / dept / api 多對多)
│   │       ├── sys_role_menu.go                 # 跨表 service:role × menu 多對多管理
│   │       └── sys_user.go                      # SysUser service(註冊 / 重設密碼 / Profile / GetInfo)
│   ├── jobs/                                    # 排程任務模組(robfig/cron 包裝)
│   │   ├── apis/
│   │   │   └── sys_job.go                       # /api/v1/sys-job CRUD + SysJobControl 開關
│   │   ├── examples.go                          # 兩個示範 job(ExamplesOne/ExamplesTwo)
│   │   ├── jobbase.go                           # Job 基底:InitJob/AddJob/HttpJob/ExecJob worker 框架
│   │   ├── models/
│   │   │   └── sys_job.go                       # sys_job table model(cron expression + invoke target + status)
│   │   ├── router/
│   │   │   ├── int_router.go                    # 內部 jobs 路由(注意 typo:int_ 不是 init_)
│   │   │   ├── router.go                        # jobs 路由 helper
│   │   │   └── sys_job.go                       # registerSysJobRouter()
│   │   ├── service/
│   │   │   ├── dto/
│   │   │   │   └── sys_job.go                   # SysJob 五個 Req DTOs
│   │   │   └── sys_job.go                       # SysJob service(同步 cron scheduler)
│   │   └── type.go                              # Job 相關 type / JobCore enum
│   └── other/                                   # 周邊功能(file / monitor / code-gen)
│       ├── apis/
│       │   ├── file.go                          # 檔案上傳 handler(local/OSS/OBS/KODO 多後端)
│       │   ├── sys_server_monitor.go            # /api/v1/server-monitor — gopsutil 抓系統資訊
│       │   └── tools/                           # code-gen 工具 API
│       │       ├── db_columns.go                # 列出 DB table 欄位 metadata
│       │       ├── db_tables.go                 # 列出 DB 所有 tables
│       │       ├── gen.go                       # code-gen 主入口(依樣板生 model/service/router)
│       │       └── sys_tables.go                # code-gen 設定的 CRUD
│       ├── models/
│       │   └── tools/
│       │       ├── db_columns.go                # DB columns metadata struct
│       │       ├── db_tables.go                 # DB tables metadata struct
│       │       ├── sys_columns.go               # sys_columns table model(code-gen 欄位設定)
│       │       └── sys_tables.go                # sys_tables table model(code-gen 表設定)
│       ├── router/
│       │   ├── file.go                          # registerFileRouter()
│       │   ├── gen_router.go                    # code-gen 路由(/sys-tables、/db/*、/api/v1/captcha)
│       │   ├── init_router.go                   # other 模組路由集中註冊
│       │   ├── monitor.go                       # Prometheus / pprof / Sentinel 觀測路由
│       │   ├── router.go                        # router helper
│       │   └── sys_server_monitor.go            # registerSysServerMonitorRouter()
│       └── service/
│           └── dto/
│               └── sys_tables.go                # SysTables 五個 Req DTOs(code-gen 設定)
│
├── cmd/                                         # Cobra 多指令入口(main.go 進這)
│   ├── api/                                     # `go-admin server` — HTTP API 啟動
│   │   ├── jobs.go                              # 啟動時掛 cron jobs
│   │   ├── other.go                             # 啟動時掛 other 模組路由
│   │   └── server.go                            # gin/middleware/router/Casbin/queue/JWT 全部組起來;line 70 註冊 LoginLog queue consumer
│   ├── app/
│   │   └── server.go                            # `go-admin app` — generated app 啟動入口
│   ├── cobra.go                                 # Cobra root command 註冊(掛所有子命令)
│   ├── config/
│   │   └── server.go                            # `go-admin config` — 印 / 驗證 / 加密 config
│   ├── migrate/                                 # `go-admin migrate` — DB 遷移
│   │   ├── migration/
│   │   │   ├── init.go                          # 集中註冊所有 version migrations
│   │   │   ├── models/                          # migration 階段的 schema struct(與 runtime model 分離避免 import 環)
│   │   │   │   ├── by.go                        # ControlBy(audit by-fields)migration 版
│   │   │   │   ├── casbin_rule.go               # casbin_rule schema(migration 版)
│   │   │   │   ├── initdb.go                    # migration init helper
│   │   │   │   ├── model.go                     # BaseModel(PK + ModelTime)migration 版
│   │   │   │   ├── role_dept.go                 # role × dept 多對多 join table schema
│   │   │   │   ├── sys_api.go                   # sys_api schema
│   │   │   │   ├── sys_columns.go               # sys_columns schema(code-gen)
│   │   │   │   ├── sys_config.go                # sys_config schema
│   │   │   │   ├── sys_dept.go                  # sys_dept schema
│   │   │   │   ├── sys_dict_data.go             # sys_dict_data schema
│   │   │   │   ├── sys_dict_type.go             # sys_dict_type schema
│   │   │   │   ├── sys_job.go                   # sys_job schema
│   │   │   │   ├── sys_login_log.go             # sys_login_log schema
│   │   │   │   ├── sys_menu.go                  # sys_menu schema
│   │   │   │   ├── sys_opera_log.go             # sys_opera_log schema
│   │   │   │   ├── sys_post.go                  # sys_post schema
│   │   │   │   ├── sys_role.go                  # sys_role schema
│   │   │   │   ├── sys_tables.go                # sys_tables schema(code-gen)
│   │   │   │   ├── sys_user.go                  # sys_user schema
│   │   │   │   └── tb_demo.go                   # 範例 demo table(學習用)
│   │   │   ├── version/                         # 依時戳排序的 migration 檔
│   │   │   │   ├── 1599190683659_tables.go      # 第一次 migration(2020-09-04):建所有核心 tables
│   │   │   │   └── 1653638869132_migrate.go     # 第二次 migration(2022-05-27):schema patch
│   │   │   └── version-local/                   # 留給 fork 自加 migration 不污染上游
│   │   │       └── doc.go                       # 占位 doc(避免目錄為空)
│   │   └── server.go                            # `go-admin migrate` 指令本體
│   └── version/
│       └── server.go                            # `go-admin version` — 印版本 + build 時間
│
├── common/                                      # 跨模組共用基礎設施
│   ├── actions/                                 # 通用 CRUD action 工廠(reflection-based)
│   │   ├── create.go                            # CreateAction() — 通用新增 handler 工廠
│   │   ├── delete.go                            # DeleteAction() — 通用刪除 handler 工廠
│   │   ├── index.go                             # IndexAction() — 通用列表 handler(分頁 + 排序)
│   │   ├── permission.go                        # PermissionAction() — 合併 Casbin + data-scope
│   │   ├── type.go                              # DataPermission 等共用 type
│   │   ├── update.go                            # UpdateAction() — 通用更新 handler 工廠
│   │   └── view.go                              # ViewAction() — 通用單筆查詢 handler 工廠
│   ├── apis/
│   │   └── api.go                               # 所有 controller 的基底 Api(MakeContext/Bind/Custom/Error/MakeOrm)
│   ├── database/
│   │   ├── initialize.go                        # Setup() — 從 settings 開 DB / 跑 migration / 註冊 Casbin
│   │   ├── open.go                              # 通用 DB open(mysql/postgres/sqlserver)
│   │   └── open_sqlite3.go                      # SQLite open(CGO + -tags sqlite3 才會編進來)
│   ├── dto/                                     # 跨模組共用 DTO
│   │   ├── auto_form.go                         # form-builder 自動產生的 DTO 包裝
│   │   ├── generate.go                          # Generate() interface — DTO 寫回 model 的標準接口
│   │   ├── order.go                             # OrderDest 排序方向 enum + helpers
│   │   ├── pagination.go                        # Pagination(pageSize/pageIndex/total)— 列表回傳統一格式
│   │   ├── search.go                            # 通用 search request(struct → GORM Where)
│   │   └── type.go                              # GeneralDelDto / GeneralGetDto / ObjectById 共用 type
│   ├── file_store/                              # 檔案儲存抽象層(local/aliyun/huawei/qiniu)
│   │   ├── initialize.go                        # 依 config 選擇 storage backend
│   │   ├── interface.go                         # FileStoreType interface(Upload/Remove)
│   │   ├── kodo.go                              # 七牛 KODO 實作
│   │   ├── kodo_test.go                         # KODO 測試(需金鑰)
│   │   ├── obs.go                               # 華為 OBS 實作
│   │   ├── obs_test.go                          # OBS 測試
│   │   ├── oss.go                               # 阿里雲 OSS 實作
│   │   └── oss_test.go                          # OSS 測試
│   ├── global/                                  # 全域 singleton
│   │   ├── adm.go                               # Admin 全域實體(Admin = adminApp{})
│   │   ├── casbin.go                            # 全域 Casbin enforcer + LoadPolicy
│   │   ├── logo.go                              # 啟動時 stdout 印的 ASCII logo
│   │   └── topic.go                             # 全域 queue topic 常數(LoginLog/OperateLog)
│   ├── ip.go                                    # GetClientIP() — 從 gin.Context 抓 IP(考慮 X-Forwarded-For)
│   ├── middleware/                              # Gin middlewares
│   │   ├── auth.go                              # AuthInit() — gin-jwt middleware 設定
│   │   ├── customerror.go                       # panic recovery + 統一錯誤 JSON
│   │   ├── db.go                                # 把 GORM connection 注入 gin.Context
│   │   ├── demo.go                              # demo 環境鎖(DemoEvn() 阻擋寫操作)
│   │   ├── handler/                             # gin-jwt callback 集合(本 fork 的登入鏈核心)
│   │   │   ├── auth.go                          # Authenticator/PayloadFunc/IdentityHandler/Authorizator/LoginLogToDB/Unauthorized/LogOut
│   │   │   ├── httpshandler.go                  # HTTPS / TLS handler
│   │   │   ├── login.go                         # Login struct + GetUser()(GORM 查 sys_user + bcrypt + 查 sys_role)
│   │   │   ├── ping.go                          # /ping 健康檢查
│   │   │   ├── role.go                          # role 相關 handler helper
│   │   │   └── user.go                          # getInfo / profile 等使用者資訊 handler
│   │   ├── header.go                            # response header(CORS / 安全 header)
│   │   ├── init.go                              # InitMiddleware() — 集中載入所有 middleware
│   │   ├── logger.go                            # 請求 log middleware(LoggerToFile())
│   │   ├── permission.go                        # AuthCheckRole() — Casbin 權限檢查 middleware
│   │   ├── request_id.go                        # RequestId() — 注入 X-Request-Id
│   │   ├── sentinel.go                          # Alibaba Sentinel 流控 middleware
│   │   ├── settings.go                          # settings 注入 ctx 的 middleware
│   │   └── trace.go                             # OpenTracing trace middleware
│   ├── models/                                  # 共用 model 基底
│   │   ├── by.go                                # ControlBy(CreatedBy/UpdatedBy)嵌入式 audit
│   │   ├── menu.go                              # menu 相關共用 type
│   │   ├── migrate.go                           # migration interface 定義
│   │   ├── response.go                          # Response struct(統一 response shape)
│   │   ├── type.go                              # BaseModel / Model(PK + ModelTime)
│   │   └── user.go                              # BaseUser 共用使用者 type
│   ├── response/
│   │   └── binding.go                           # gin binding 錯誤訊息客製化 + 統一格式
│   ├── service/
│   │   └── service.go                           # service 層基底類別(logger/orm/錯誤累積)
│   └── storage/
│       └── initialize.go                        # queue / cache 後端初始化(memory/redis)
│
├── config/                                      # 設定檔與 SQL seed
│   ├── READMEN.md                               # 中文 settings.yml 各區段說明(注意 READMEN typo)
│   ├── db-begin-mysql.sql                       # MySQL:clean DB 開始的 schema reset
│   ├── db-end-mysql.sql                         # MySQL:seed 完成後的 finalize SQL
│   ├── db-sqlserver.sql                         # SQL Server schema + seed
│   ├── db.sql                                   # 主要 MySQL schema + seed(admin/角色/選單/Casbin policy)
│   ├── extend.go                                # settings.yml 的 extend: 區段 → Go struct
│   ├── pg.sql                                   # PostgreSQL schema + seed
│   ├── settings.demo.yml                        # demo 環境設定
│   ├── settings.full.yml                        # 完整選項範本(所有可用 keys)
│   ├── settings.sqlite.yml                      # SQLite 設定(driver: sqlite3,Dockerfile.learning 用)
│   └── settings.yml                             # 預設(driver: mysql,127.0.0.1:3306,直接跑會掉進陷阱)
│
├── docker-compose.yml                           # 上游生產 compose(image: go-admin:latest,需先 make build-linux)
│
├── docs/                                        # Swagger 自動產生文件
│   └── admin/
│       ├── admin_docs.go                        # swaggo go embed(把 spec 編進 binary)
│       ├── admin_swagger.json                   # swagger spec JSON
│       └── admin_swagger.yaml                   # swagger spec YAML
│
├── go-admin-db.db                               # 預先 seeded SQLite DB(admin/123456 + sys_* tables + Casbin + menu 樹)
├── go.mod                                       # Go module:module go-admin、go 1.24、~50 直接依賴
│
├── graphify-out/                                # graphify 知識圖譜輸出(整個目錄由工具生成)
│   ├── GRAPH_REPORT.md                          # 圖譜總覽(god nodes / communities / surprising connections)
│   ├── cache/                                   # ~190 個 hash-keyed JSON 抽取快取(graphify 內部用,勿手改)
│   ├── cost.json                                # 抽取 token / cost 紀錄
│   ├── graph.html                               # 互動式圖譜瀏覽(瀏覽器開)
│   ├── graph.json                               # 原始 nodes/edges 資料
│   ├── manifest.json                            # 抽取 manifest(輸入清單與設定)
│   ├── memory/                                  # graphify save-result 寫入的 query 紀錄(下次 update 會拌進圖)
│   └── obsidian/                                # ~1500 個自動生成的 Obsidian wiki note + graph.canvas(可在 Obsidian 開資料夾為 vault,勿手改)
│
├── main.go                                      # 程式入口(極短,只 cmd.Execute() 跳到 cobra root)
├── package-lock.json                            # 殘留 npm lock(無 package.json,歷史測試遺留)
├── restart.sh                                   # shell 腳本:重啟服務(kill + 重 run)
│
├── scripts/                                     # 運維 / 部署腳本
│   ├── Dockerfile                               # 另一份備用 Dockerfile(不同階段用法)
│   └── k8s/
│       ├── deploy.yml                           # K8s Deployment 樣板
│       ├── prerun.sh                            # K8s init container 前置腳本
│       └── storage.yml                          # K8s PVC 樣板
│
├── ssh/
│   └── swag.sh                                  # swaggo 重新產生 docs 的 helper(swag init)
│
├── static/                                      # 內建前端資產(form-builder + 上傳檔)
│   ├── form-generator/                          # 預編譯 vue-form-making bundle(admin 後台 iframe 它)
│   │   ├── css/
│   │   │   ├── index.1a124643.css               # 主 CSS(webpack hash)
│   │   │   └── parser-example.69e16e51.css      # parser-example 頁的 CSS
│   │   ├── img/
│   │   │   └── logo.e1bc3747.png                # form-generator logo
│   │   ├── index.html                           # form-generator 入口頁
│   │   ├── js/                                  # 預編譯 bundle(graphify god nodes Ke/Be/de() 都在 chunk-vendors)
│   │   │   ├── chunk-vendors.971555db.js        # 第三方 vendor 大包(Babel/Flow/TS parser ~1600 functions)
│   │   │   ├── index.8e6d9f8f.js                # form-generator 主 bundle
│   │   │   ├── parser-example.ce55fa09.js       # 解析器示例頁 bundle
│   │   │   ├── preview.8ce4e0db.js              # 預覽頁 bundle
│   │   │   └── tinymce-example.641995ab.js      # TinyMCE 編輯器示例 bundle
│   │   └── preview.html                         # 預覽頁
│   └── uploadfile/                              # 上傳檔案儲存目錄(runtime 寫入)
│       ├── 77cfc1dd-535c-4e60-b34a-5909e2cf5ed0.jpg  # 上傳測試殘留圖片(git 管理進來的樣本)
│       └── log.txt                              # 上傳目錄 placeholder(保留資料夾用)
│
├── stop.sh                                      # shell 腳本:停服務
├── temp/                                        # runtime 暫存 / log 目錄(.gitignore 忽略內容)
│   └── logs/                                    # 請求 log 寫入處(空目錄)
│
├── template/                                    # code generator 樣板(被 app/other/apis/tools/gen.go 用)
│   ├── api_migrate.template                     # 產生 migration 檔的樣板
│   ├── cmd_api.template                         # 產生新 cobra subcommand 的樣板
│   ├── migrate.template                         # 通用 migration 樣板
│   ├── router.template                          # 產生 router 註冊檔的樣板
│   └── v4/                                      # v4 樣板組(預設使用)
│       ├── actions/                             # 用 common/actions 工廠的版本
│       │   ├── router_check_role.go.template    # 含權限檢查的 router 段
│       │   └── router_no_check_role.go.template # 不檢查權限的 router 段
│       ├── dto.go.template                      # DTO 檔
│       ├── js.go.template                       # 前端 JS API client
│       ├── model.go.template                    # GORM model 檔
│       ├── no_actions/                          # 不用 common/actions 的版本(手寫 CRUD)
│       │   ├── apis.go.template                 # API handler 檔
│       │   ├── router_check_role.go.template    # 含權限檢查的 router
│       │   ├── router_no_check_role.go.template # 不檢查權限的 router
│       │   └── service.go.template              # service 檔
│       └── vue.go.template                      # vue 前端頁面(給 frontpath 用,需 go-admin-ui)
│
├── test/                                        # code-gen 行為測試 + 對應樣板
│   ├── api.go.template                          # 測試樣板:API 層
│   ├── gen_test.go                              # code-gen 行為的 Go test(驗證樣板渲染正確)
│   └── model.go.template                        # 測試樣板:model 層
│
├── x_fork.branch-origin.md                      # fork 由來(從 upstream master 取出 a5cc0a9 → 改名 main)
├── x_fork.graphify-20260506.md                  # 第一次跑 graphify 的 session 紀錄
└── x_fork.tree-20260506.md                      # 本檔(tree 結構 + 全檔註解)
```

---

## 補充說明

**幾個重要慣例**:

- `sys_*.go` 命名同步出現在 `app/admin/{apis,models,router,service[/dto]}/`
  四個目錄,代表同一個資源在四層架構的對應檔。閱讀順序建議:
  router → apis → service → service/dto → models。
- `*_test.go` 一律是 Go 標準測試檔(注意 file_store 子套件的測試需要實際雲端
  金鑰才會跑)。
- migration 階段的 model(`cmd/migrate/migration/models/sys_*.go`)與 runtime
  model(`app/admin/models/sys_*.go`)是兩份,刻意分離以避免 import 環。
  改 schema 要兩邊一起改。
- `common/actions/{create,delete,index,update,view}.go` 是通用 reflection-based
  CRUD 工廠;controller 內若用 `CreateAction()` 等就走這條,自己手寫 GORM
  則繞開。
- `common/middleware/handler/auth.go` + `login.go` 是登入鏈核心,FR-016
  feature `001-learning-trace-login` 的 §5 layer trace 完整解構這條路徑。
- `static/form-generator/js/chunk-vendors.971555db.js` 是預編譯的 Babel/Flow/
  TypeScript parser bundle(form-builder 拿來剖析 vue 模板)。**它在 graphify
  圖裡產生最多 god nodes**(`Ke`、`Be`、`de()` 等),但跟 go-admin 的 Go
  業務邏輯完全無關 — 讀程式碼時可忽略 community 1-8 整批前端 parser 節點。
- `x_fork.*` 開頭的檔案都是這份 fork 的 meta 紀錄,不影響任何程式邏輯。

**最常被引用的 god nodes**(來自 graphify GRAPH_REPORT,**已排除前端 parser**):

| 節點 | 所在檔案 | 邊數 |
|---|---|---|
| `Model` | `common/models/type.go`(BaseModel) | 47 |
| `de()` | (前端 parser,可忽略) | 47 |
| `getPermissionFromContext()` | `common/middleware/permission.go` | 21 |
| `Authenticator()` | `common/middleware/handler/auth.go:63-107` | (auth 鏈核心) |
| `Login.GetUser()` | `common/middleware/handler/login.go:16-33` | (auth 鏈核心) |
| `LoginLogToDB()` | `common/middleware/handler/auth.go:77` | (audit log 入口,async via queue) |
| `PayloadFunc()` | `common/middleware/handler/auth.go:21-35` | (JWT claims 建構) |
| `SaveLoginLog()` | `app/admin/models/sys_login_log.go:46` | (queue consumer 真正寫 DB) |
| `Api`(struct) | `common/apis/api.go:18` | (所有 controller 的基底) |

讀程式碼建議從 `main.go` → `cmd/api/server.go` → `app/admin/router/sys_router.go`
→ `common/middleware/handler/auth.go` 這條主線往外爬,效率最高。
登入鏈完整逐層解構見 feature branch `001-learning-trace-login` 的
`docs/learning/login-trace.md`(main 上目前不存在,在 PR #2)。

---

*本檔案於 2026-05-08 由 Claude Code 依 `tree` 輸出生成,日期 `20260506`
沿用 fork 的 snapshot 命名慣例(對齊 `x_fork.graphify-20260506.md`)。*
