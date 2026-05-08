---
source_file: "x_fork.tree-20260506.md"
type: "rationale"
community: "Community 53"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_53
---

# Recommended reading path: main.go -> cmd/api/server.go -> sys_router.go -> handler/auth.go

## Connections
- [[appadminroutersys_router.go71 (loginrefresh_tokenlogoutswagger registration)]] - `references` [EXTRACTED]
- [[cmdapiserver.go (ginmiddlewarerouterCasbinqueueJWT wiring; line 70 registers LoginLog queue consumer)]] - `references` [EXTRACTED]
- [[commonmiddlewarehandlerauth.go (AuthenticatorPayloadFuncIdentityHandlerAuthorizatorLoginLogToDBUnauthorizedLogOut)]] - `references` [EXTRACTED]
- [[main.go (tiny entrypoint, defers to cmd.Execute)]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_53