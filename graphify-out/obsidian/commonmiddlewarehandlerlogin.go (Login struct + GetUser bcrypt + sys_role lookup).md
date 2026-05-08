---
source_file: "x_fork.tree-20260506.md"
type: "rationale"
community: "Community 28"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_28
---

# common/middleware/handler/login.go (Login struct + GetUser bcrypt + sys_role lookup)

## Connections
- [[2026-05-07-login-trace.md (POST apiv1login walkthrough, 805 lines)]] - `cites` [EXTRACTED]
- [[God-nodes catalogue (Model, getPermissionFromContext, Authenticator, Login.GetUser, LoginLogToDB, PayloadFunc, SaveLoginLog, Api)]] - `references` [EXTRACTED]
- [[Layer 5.5 - Login struct + Login.GetUser sys_user query (login.go9-14, 16-33)]] - `references` [EXTRACTED]
- [[Layer 5.6 - pkg.CompareHashAndPassword bcrypt compare (login.go22) - the only CPU-bound step]] - `references` [EXTRACTED]
- [[Layer 5.7 - GORM role lookup sys_role (login.go27) - 2nd independent SQL round-trip]] - `references` [EXTRACTED]
- [[commonmiddleware (Gin middlewares + handler login chain core)]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_28