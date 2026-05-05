# Graph Report - .  (2026-05-06)

## Corpus Check
- 196 files · ~62,915 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 1725 nodes · 5422 edges · 73 communities detected
- Extraction: 77% EXTRACTED · 23% INFERRED · 0% AMBIGUOUS · INFERRED: 1256 edges (avg confidence: 0.8)
- Token cost: 15,100 input · 5,000 output

## Community Hubs (Navigation)
- [[_COMMUNITY_Frontend Parser Internals|Frontend Parser Internals]]
- [[_COMMUNITY_Admin REST APIs|Admin REST APIs]]
- [[_COMMUNITY_Expression Parser Helpers|Expression Parser Helpers]]
- [[_COMMUNITY_Admin Models & Service Layer|Admin Models & Service Layer]]
- [[_COMMUNITY_Init & File Routing|Init & File Routing]]
- [[_COMMUNITY_Comment & Declaration Parser|Comment & Declaration Parser]]
- [[_COMMUNITY_Admin Routers & Permission|Admin Routers & Permission]]
- [[_COMMUNITY_Expression Operator Parser|Expression Operator Parser]]
- [[_COMMUNITY_Admin Sys Models|Admin Sys Models]]
- [[_COMMUNITY_Common Service & Form Builder|Common Service & Form Builder]]
- [[_COMMUNITY_Project Documentation|Project Documentation]]
- [[_COMMUNITY_Job Scheduling|Job Scheduling]]
- [[_COMMUNITY_User DTOs|User DTOs]]
- [[_COMMUNITY_Scope Tracker Parser|Scope Tracker Parser]]
- [[_COMMUNITY_Role DTOs|Role DTOs]]
- [[_COMMUNITY_Config DTOs|Config DTOs]]
- [[_COMMUNITY_Auth & Login Middleware|Auth & Login Middleware]]
- [[_COMMUNITY_Menu DTOs|Menu DTOs]]
- [[_COMMUNITY_Common DTO Forms|Common DTO Forms]]
- [[_COMMUNITY_API DTOs|API DTOs]]
- [[_COMMUNITY_Department DTOs|Department DTOs]]
- [[_COMMUNITY_Dict Data DTOs|Dict Data DTOs]]
- [[_COMMUNITY_Dict Type DTOs|Dict Type DTOs]]
- [[_COMMUNITY_Post DTOs|Post DTOs]]
- [[_COMMUNITY_Server Monitor|Server Monitor]]
- [[_COMMUNITY_Operation Log DTOs|Operation Log DTOs]]
- [[_COMMUNITY_Login Log DTOs|Login Log DTOs]]
- [[_COMMUNITY_Form Parser Example|Form Parser Example]]
- [[_COMMUNITY_Comment Processor|Comment Processor]]
- [[_COMMUNITY_Huawei OBS Storage|Huawei OBS Storage]]
- [[_COMMUNITY_Response Helpers|Response Helpers]]
- [[_COMMUNITY_Form Generator Logo|Form Generator Logo]]
- [[_COMMUNITY_Go Gopher Asset|Go Gopher Asset]]
- [[_COMMUNITY_Casbin Rule Model|Casbin Rule Model]]
- [[_COMMUNITY_DB Columns Tool|DB Columns Tool]]
- [[_COMMUNITY_DB Tables Tool|DB Tables Tool]]
- [[_COMMUNITY_File Store Interface|File Store Interface]]
- [[_COMMUNITY_Middleware User Handler|Middleware User Handler]]
- [[_COMMUNITY_File Router|File Router]]
- [[_COMMUNITY_RoleDept Migration|RoleDept Migration]]
- [[_COMMUNITY_SysColumns Migration|SysColumns Migration]]
- [[_COMMUNITY_DictData Migration|DictData Migration]]
- [[_COMMUNITY_DictType Migration|DictType Migration]]
- [[_COMMUNITY_SysTables Migration|SysTables Migration]]
- [[_COMMUNITY_TbDemo Migration|TbDemo Migration]]
- [[_COMMUNITY_Role Middleware|Role Middleware]]
- [[_COMMUNITY_Config Extend|Config Extend]]
- [[_COMMUNITY_Code Generation Tests|Code Generation Tests]]
- [[_COMMUNITY_GoAdmin Banner Endpoint|GoAdmin Banner Endpoint]]
- [[_COMMUNITY_RoleMenu Service|RoleMenu Service]]
- [[_COMMUNITY_SysTable Service DTO|SysTable Service DTO]]
- [[_COMMUNITY_API Jobs Init|API Jobs Init]]
- [[_COMMUNITY_API Other Init|API Other Init]]
- [[_COMMUNITY_Migration BaseModel|Migration BaseModel]]
- [[_COMMUNITY_Migrations Local Init|Migrations Local Init]]
- [[_COMMUNITY_Settings UrlInfo|Settings UrlInfo]]
- [[_COMMUNITY_Ping Handler|Ping Handler]]
- [[_COMMUNITY_Swagger Docs Init|Swagger Docs Init]]
- [[_COMMUNITY_DB Columns API|DB Columns API]]
- [[_COMMUNITY_DB Tables API|DB Tables API]]
- [[_COMMUNITY_Actions Type|Actions Type]]
- [[_COMMUNITY_Database Open|Database Open]]
- [[_COMMUNITY_SQLite Open|SQLite Open]]
- [[_COMMUNITY_Global Admin|Global Admin]]
- [[_COMMUNITY_Global Logo|Global Logo]]
- [[_COMMUNITY_Global Topic|Global Topic]]
- [[_COMMUNITY_Common Menu Model|Common Menu Model]]
- [[_COMMUNITY_TinyMCE Example Bundle|TinyMCE Example Bundle]]
- [[_COMMUNITY_CGO SQLite Build Note|CGO SQLite Build Note]]
- [[_COMMUNITY_Empty Upload Log|Empty Upload Log]]
- [[_COMMUNITY_Generate ObjectById DTO|Generate ObjectById DTO]]
- [[_COMMUNITY_Pagination Struct|Pagination Struct]]
- [[_COMMUNITY_Model Audit Base|Model Audit Base]]

## God Nodes (most connected - your core abstractions)
1. `Ke` - 87 edges
2. `Be` - 79 edges
3. `Model` - 47 edges
4. `de()` - 47 edges
5. `Fe` - 33 edges
6. `flowParsePrimaryType()` - 24 edges
7. `tsParseNonArrayType()` - 23 edges
8. `Me` - 23 edges
9. `getPermissionFromContext()` - 21 edges
10. `flowParseTypeParameterDeclaration()` - 21 edges

## Surprising Connections (you probably didn't know these)
- `go-admin Project (Chinese README)` --semantically_similar_to--> `go-admin Project (English README)`  [INFERRED] [semantically similar]
  README.Zh-cn.md → README.md
- `settings.yml configuration schema (Chinese annotated)` --semantically_similar_to--> `config/settings.yml configuration file`  [INFERRED] [semantically similar]
  config/READMEN.md → README.md
- `main()` --calls--> `Execute()`  [INFERRED]
  main.go → cmd/cobra.go
- `run()` --calls--> `InitJob()`  [INFERRED]
  cmd/version/server.go → app/jobs/examples.go
- `Setup()` --calls--> `Setup()`  [INFERRED]
  app/jobs/jobbase.go → common/storage/initialize.go

## Hyperedges (group relationships)
- **go-admin core technology stack** — readme_gin_framework, readme_gorm, readme_casbin, readme_jwt_auth, readme_swaggo, readme_viper_config, readme_vue_frontend [EXTRACTED 0.90]
- **settings.yml top-level subsections** — readmen_application_section, readmen_log_section, readmen_jwt_section, readmen_database_section [EXTRACTED 1.00]
- **Fork branch lineage (upstream -> master -> main)** — x_fork_upstream_repo, x_fork_master_branch, x_fork_main_branch, x_fork_head_a5cc0a9 [EXTRACTED 1.00]

## Communities

### Community 0 - "Frontend Parser Internals"
Cohesion: 0.03
Nodes (211): assertModuleNodeAllowed(), Be, checkExport(), checkProto(), directiveToStmt(), estreeParseBigIntLiteral(), estreeParseLiteral(), estreeParseRegExpLiteral() (+203 more)

### Community 1 - "Admin REST APIs"
Cohesion: 0.05
Nodes (31): Api, SysApi, SysConfig, SysDept, SysDictData, SysDictType, SysJob, SysLoginLog (+23 more)

### Community 2 - "Expression Parser Helpers"
Cohesion: 0.04
Nodes (60): canHaveLeadingDecorator(), checkLVal(), checkParams(), checkReservedWord(), eatExportStar(), finishArrowValidation(), finishPlaceholder(), flowParseDeclare() (+52 more)

### Community 3 - "Admin Models & Service Layer"
Cohesion: 0.03
Nodes (37): CustomError(), GetDataScope (method, dead code), GeneralDelDto, GeneralGetDto, ObjectById, Pagination, SysJobById, SysJobControl (+29 more)

### Community 4 - "Init & File Routing"
Cohesion: 0.03
Nodes (51): _1599190683659Tables(), init(), _1653638869132Test(), init(), File, FileResponse, AuthInit(), InitJob() (+43 more)

### Community 5 - "Comment & Declaration Parser"
Cohesion: 0.05
Nodes (41): addComment(), at, checkDeclaration(), checkGetterSetterParams(), checkNotUnderscore(), checkReservedType(), ct(), de() (+33 more)

### Community 6 - "Admin Routers & Permission"
Cohesion: 0.04
Nodes (37): DataPermission, DataPermission (struct, app/admin/models), DemoEvn(), registerDBRouter(), registerSysTableRouter(), sysNoCheckRoleRouter(), InitMiddleware(), GetClientIP() (+29 more)

### Community 7 - "Expression Operator Parser"
Cohesion: 0.06
Nodes (49): _(), A(), Ae(), b(), C(), ce, D(), _e (+41 more)

### Community 8 - "Admin Sys Models"
Cohesion: 0.03
Nodes (15): ControlBy (audit by-fields), ActiveRecord, SysApi, SysConfig, SysDept, SysDictData, SysDictType, SysLoginLog (+7 more)

### Community 9 - "Common Service & Form Builder"
Cohesion: 0.05
Nodes (42): _(), a(), B(), buildAttributes(), buildBeforeUpload(), buildData(), buildexport(), buildOptionMethod() (+34 more)

### Community 10 - "Project Documentation"
Cohesion: 0.07
Nodes (34): go-admin-team (copyright holder), MIT License (go-admin), Casbin RBAC Access Control, Code Generation Tool, Docker Build & Deploy Workflow, Form Builder, Gin Web Framework, go-admin Project (English README) (+26 more)

### Community 11 - "Job Scheduling"
Cohesion: 0.09
Nodes (10): AddJob(), Setup(), ExecJob, HttpJob, Job, JobCore, JobExec, SysJob (+2 more)

### Community 12 - "User DTOs"
Cohesion: 0.08
Nodes (10): DeptJoin, PassWord, ResetSysUserPwdReq, SysUserById, SysUserGetPageReq, SysUserInsertReq, SysUserOrder, SysUserUpdateReq (+2 more)

### Community 13 - "Scope Tracker Parser"
Cohesion: 0.13
Nodes (2): ee, ie

### Community 14 - "Role DTOs"
Cohesion: 0.1
Nodes (10): DeptIdList, RoleDataScopeReq, SysRoleByName, SysRoleDeleteReq, SysRoleGetPageReq, SysRoleGetReq, SysRoleInsertReq, SysRoleOrder (+2 more)

### Community 15 - "Config DTOs"
Cohesion: 0.11
Nodes (10): GetSetSysConfigReq, GetSysConfigByKEYForServiceResp, SysConfigByKeyReq, SysConfigControl, SysConfigDeleteReq, SysConfigGetPageReq, SysConfigGetReq, SysConfigGetToSysAppReq (+2 more)

### Community 16 - "Auth & Login Middleware"
Cohesion: 0.15
Nodes (5): Authenticator(), LoginLogToDB(), LogOut(), Login, BaseUser

### Community 17 - "Menu DTOs"
Cohesion: 0.12
Nodes (8): MenuLabel, MenuRole, SelectRole, SysMenuDeleteReq, SysMenuGetPageReq, SysMenuGetReq, SysMenuInsertReq, SysMenuUpdateReq

### Community 18 - "Common DTO Forms"
Cohesion: 0.14
Nodes (9): AutoForm, Config, Control, Field, Index, Option, Slot, Style (+1 more)

### Community 19 - "API DTOs"
Cohesion: 0.14
Nodes (6): SysApiDeleteReq, SysApiGetPageReq, SysApiGetReq, SysApiInsertReq, SysApiOrder, SysApiUpdateReq

### Community 20 - "Department DTOs"
Cohesion: 0.14
Nodes (6): DeptLabel, SysDeptDeleteReq, SysDeptGetPageReq, SysDeptGetReq, SysDeptInsertReq, SysDeptUpdateReq

### Community 21 - "Dict Data DTOs"
Cohesion: 0.14
Nodes (6): SysDictDataDeleteReq, SysDictDataGetAllResp, SysDictDataGetPageReq, SysDictDataGetReq, SysDictDataInsertReq, SysDictDataUpdateReq

### Community 22 - "Dict Type DTOs"
Cohesion: 0.14
Nodes (6): SysDictTypeDeleteReq, SysDictTypeGetPageReq, SysDictTypeGetReq, SysDictTypeInsertReq, SysDictTypeOrder, SysDictTypeUpdateReq

### Community 23 - "Post DTOs"
Cohesion: 0.14
Nodes (5): SysPostDeleteReq, SysPostGetReq, SysPostInsertReq, SysPostPageReq, SysPostUpdateReq

### Community 24 - "Server Monitor"
Cohesion: 0.3
Nodes (10): ServerMonitor, getCPUInfo(), getDiskInfo(), GetHourDiffer(), getMemoryInfo(), getNetworkInfo(), getOSInfo(), getSwapInfo() (+2 more)

### Community 25 - "Operation Log DTOs"
Cohesion: 0.18
Nodes (5): SysOperaLogControl, SysOperaLogDeleteReq, SysOperaLogGetPageReq, SysOperaLogGetReq, SysOperaLogOrder

### Community 26 - "Login Log DTOs"
Cohesion: 0.22
Nodes (5): SysLoginLogControl, SysLoginLogDeleteReq, SysLoginLogGetPageReq, SysLoginLogGetReq, SysLoginLogOrder

### Community 27 - "Form Parser Example"
Cohesion: 0.25
Nodes (0): 

### Community 28 - "Comment Processor"
Cohesion: 0.4
Nodes (3): lt(), Mt(), pt

### Community 29 - "Huawei OBS Storage"
Cohesion: 0.4
Nodes (1): HuaWeiOBS

### Community 30 - "Response Helpers"
Cohesion: 0.4
Nodes (2): Page, Response

### Community 31 - "Form Generator Logo"
Cohesion: 0.6
Nodes (5): Form Generator module, Green rounded pebble background, Stylized letter V, Form Generator Logo (V mark), vue-form-making / form-generator project

### Community 32 - "Go Gopher Asset"
Cohesion: 0.6
Nodes (5): go-admin Project, Go Gopher (Go language mascot), Go Programming Language, Go Gopher Mascot Image, Static Uploadfile Assets

### Community 33 - "Casbin Rule Model"
Cohesion: 0.5
Nodes (1): CasbinRule

### Community 34 - "DB Columns Tool"
Cohesion: 0.5
Nodes (1): DBColumns

### Community 35 - "DB Tables Tool"
Cohesion: 0.5
Nodes (1): DBTables

### Community 36 - "File Store Interface"
Cohesion: 0.5
Nodes (3): ClientOption, DriverType, FileStoreType

### Community 37 - "Middleware User Handler"
Cohesion: 0.5
Nodes (1): SysUser

### Community 38 - "File Router"
Cohesion: 0.67
Nodes (0): 

### Community 39 - "RoleDept Migration"
Cohesion: 0.67
Nodes (1): SysRoleDept

### Community 40 - "SysColumns Migration"
Cohesion: 0.67
Nodes (1): SysColumns

### Community 41 - "DictData Migration"
Cohesion: 0.67
Nodes (1): DictData

### Community 42 - "DictType Migration"
Cohesion: 0.67
Nodes (1): DictType

### Community 43 - "SysTables Migration"
Cohesion: 0.67
Nodes (1): SysTables

### Community 44 - "TbDemo Migration"
Cohesion: 0.67
Nodes (1): TbDemo

### Community 45 - "Role Middleware"
Cohesion: 0.67
Nodes (1): SysRole

### Community 46 - "Config Extend"
Cohesion: 0.67
Nodes (2): AMap, Extend

### Community 47 - "Code Generation Tests"
Cohesion: 0.67
Nodes (0): 

### Community 48 - "GoAdmin Banner Endpoint"
Cohesion: 1.0
Nodes (0): 

### Community 49 - "RoleMenu Service"
Cohesion: 1.0
Nodes (1): SysRoleMenu

### Community 50 - "SysTable Service DTO"
Cohesion: 1.0
Nodes (1): SysTableSearch

### Community 51 - "API Jobs Init"
Cohesion: 1.0
Nodes (0): 

### Community 52 - "API Other Init"
Cohesion: 1.0
Nodes (0): 

### Community 53 - "Migration BaseModel"
Cohesion: 1.0
Nodes (1): BaseModel

### Community 54 - "Migrations Local Init"
Cohesion: 1.0
Nodes (0): 

### Community 55 - "Settings UrlInfo"
Cohesion: 1.0
Nodes (1): UrlInfo

### Community 56 - "Ping Handler"
Cohesion: 1.0
Nodes (0): 

### Community 57 - "Swagger Docs Init"
Cohesion: 1.0
Nodes (0): 

### Community 58 - "DB Columns API"
Cohesion: 1.0
Nodes (0): 

### Community 59 - "DB Tables API"
Cohesion: 1.0
Nodes (0): 

### Community 60 - "Actions Type"
Cohesion: 1.0
Nodes (0): 

### Community 61 - "Database Open"
Cohesion: 1.0
Nodes (0): 

### Community 62 - "SQLite Open"
Cohesion: 1.0
Nodes (0): 

### Community 63 - "Global Admin"
Cohesion: 1.0
Nodes (0): 

### Community 64 - "Global Logo"
Cohesion: 1.0
Nodes (0): 

### Community 65 - "Global Topic"
Cohesion: 1.0
Nodes (0): 

### Community 66 - "Common Menu Model"
Cohesion: 1.0
Nodes (0): 

### Community 67 - "TinyMCE Example Bundle"
Cohesion: 1.0
Nodes (0): 

### Community 68 - "CGO SQLite Build Note"
Cohesion: 1.0
Nodes (1): CGO + go-sqlite3 build issue (Windows)

### Community 69 - "Empty Upload Log"
Cohesion: 1.0
Nodes (1): Empty upload log placeholder (static/uploadfile/log.txt)

### Community 70 - "Generate ObjectById DTO"
Cohesion: 1.0
Nodes (1): ObjectById (DTO)

### Community 71 - "Pagination Struct"
Cohesion: 1.0
Nodes (1): Pagination (struct)

### Community 72 - "Model Audit Base"
Cohesion: 1.0
Nodes (1): Model (PK)

## Ambiguous Edges - Review These
- `Form Generator module` → `vue-form-making / form-generator project`  [AMBIGUOUS]
  static/form-generator/img/logo.e1bc3747.png · relation: derived_from
- `Go Gopher Mascot Image` → `go-admin Project`  [AMBIGUOUS]
  static/uploadfile/77cfc1dd-535c-4e60-b34a-5909e2cf5ed0.jpg · relation: used_as_sample_upload_in
- `GetDataScope (method, dead code)` → `GetDataScope (method, dead code)`  [AMBIGUOUS]
  app/admin/models/datascope.go · relation: rationale_for

## Knowledge Gaps
- **63 isolated node(s):** `SysRoleMenu`, `SysApiOrder`, `SysConfigOrder`, `UpdateSetSysConfigReq`, `GetSysConfigByKEYForServiceResp` (+58 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **Thin community `GoAdmin Banner Endpoint`** (2 nodes): `go_admin.go`, `GoAdmin()`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `RoleMenu Service`** (2 nodes): `sys_role_menu.go`, `SysRoleMenu`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `SysTable Service DTO`** (2 nodes): `sys_tables.go`, `SysTableSearch`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `API Jobs Init`** (2 nodes): `jobs.go`, `init()`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `API Other Init`** (2 nodes): `other.go`, `init()`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Migration BaseModel`** (2 nodes): `model.go`, `BaseModel`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Migrations Local Init`** (2 nodes): `doc.go`, `init()`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Settings UrlInfo`** (2 nodes): `settings.go`, `UrlInfo`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Ping Handler`** (2 nodes): `ping.go`, `Ping()`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Swagger Docs Init`** (2 nodes): `init()`, `admin_docs.go`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `DB Columns API`** (1 nodes): `db_columns.go`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `DB Tables API`** (1 nodes): `db_tables.go`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Actions Type`** (1 nodes): `type.go`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Database Open`** (1 nodes): `open.go`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `SQLite Open`** (1 nodes): `open_sqlite3.go`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Global Admin`** (1 nodes): `adm.go`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Global Logo`** (1 nodes): `logo.go`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Global Topic`** (1 nodes): `topic.go`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Common Menu Model`** (1 nodes): `menu.go`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `TinyMCE Example Bundle`** (1 nodes): `tinymce-example.641995ab.js`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `CGO SQLite Build Note`** (1 nodes): `CGO + go-sqlite3 build issue (Windows)`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Empty Upload Log`** (1 nodes): `Empty upload log placeholder (static/uploadfile/log.txt)`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Generate ObjectById DTO`** (1 nodes): `ObjectById (DTO)`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Pagination Struct`** (1 nodes): `Pagination (struct)`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Model Audit Base`** (1 nodes): `Model (PK)`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **What is the exact relationship between `Form Generator module` and `vue-form-making / form-generator project`?**
  _Edge tagged AMBIGUOUS (relation: derived_from) - confidence is low._
- **What is the exact relationship between `Go Gopher Mascot Image` and `go-admin Project`?**
  _Edge tagged AMBIGUOUS (relation: used_as_sample_upload_in) - confidence is low._
- **What is the exact relationship between `GetDataScope (method, dead code)` and `GetDataScope (method, dead code)`?**
  _Edge tagged AMBIGUOUS (relation: rationale_for) - confidence is low._
- **Why does `IndexAction()` connect `Admin Models & Service Layer` to `Admin REST APIs`, `Expression Parser Helpers`, `Init & File Routing`, `Admin Routers & Permission`, `Admin Sys Models`?**
  _High betweenness centrality (0.090) - this node is a cross-community bridge._
- **Why does `ActiveRecord` connect `Admin Sys Models` to `Admin Models & Service Layer`?**
  _High betweenness centrality (0.061) - this node is a cross-community bridge._
- **Why does `n()` connect `Expression Operator Parser` to `Frontend Parser Internals`, `Admin REST APIs`, `Expression Parser Helpers`?**
  _High betweenness centrality (0.039) - this node is a cross-community bridge._
- **Are the 45 inferred relationships involving `Model` (e.g. with `.GetPage()` and `.Get()`) actually correct?**
  _`Model` has 45 INFERRED edges - model-reasoned connections that need verification._