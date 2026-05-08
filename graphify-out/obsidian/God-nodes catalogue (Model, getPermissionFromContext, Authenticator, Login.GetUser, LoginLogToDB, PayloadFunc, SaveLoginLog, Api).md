---
source_file: "x_fork.tree-20260506.md"
type: "rationale"
community: "Community 49"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_49
---

# God-nodes catalogue (Model, getPermissionFromContext, Authenticator, Login.GetUser, LoginLogToDB, PayloadFunc, SaveLoginLog, Api)

## Connections
- [[appadminmodelssys_login_log.go SaveLoginLog queue consumer]] - `references` [EXTRACTED]
- [[commonapisapi.go (base Api struct MakeContextBindCustomErrorMakeOrm)]] - `references` [EXTRACTED]
- [[commonmiddlewarehandlerauth.go (AuthenticatorPayloadFuncIdentityHandlerAuthorizatorLoginLogToDBUnauthorizedLogOut)]] - `references` [EXTRACTED]
- [[commonmiddlewarehandlerlogin.go (Login struct + GetUser bcrypt + sys_role lookup)]] - `references` [EXTRACTED]
- [[commonmodels (BaseModelControlByModel audit + PK base)]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_49