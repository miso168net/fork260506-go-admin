---
source_file: "docs/superpowers/specs/2026-05-07-login-trace.md"
type: "document"
community: "Community 53"
tags:
  - graphify/document
  - graphify/EXTRACTED
  - community/Community_53
---

# 2026-05-07-login-trace.md (POST /api/v1/login walkthrough, 805 lines)

## Connections
- [[Section 1 - Learning goals & prerequisites (Docker-only, no GoMySQLNode)]] - `references` [EXTRACTED]
- [[Section 2 - Bring up backend with `docker compose -f docker-compose.learning.yml up --build`]] - `references` [EXTRACTED]
- [[Section 3 - Canonical login curl with code0uuid0 dev-mode bypass]] - `references` [EXTRACTED]
- [[Section 4 - Mermaid sequence diagram (11-actor login chain with two SDK rect boundaries)]] - `references` [EXTRACTED]
- [[Section 6 - Runtime verification recipes (predict-execute-compare)]] - `references` [EXTRACTED]
- [[Section 7 - 3 SDK boundaries LoginHandler, TokenGenerator, default LoginResponse]] - `references` [EXTRACTED]
- [[SpecKit feature 001-learning-trace-login (parent spec of this doc)]] - `cites` [EXTRACTED]
- [[Track A - GET apiv1sys-user follow-up (Casbin + data-scope filter)]] - `references` [EXTRACTED]
- [[Track B - write-path CRUD with sys_operation_log (global.OperateLog topic)]] - `references` [EXTRACTED]
- [[appadminmodelssys_login_log.go SaveLoginLog queue consumer]] - `cites` [EXTRACTED]
- [[appadminroutersys_router.go71 (loginrefresh_tokenlogoutswagger registration)]] - `cites` [EXTRACTED]
- [[cmdapiserver.go (ginmiddlewarerouterCasbinqueueJWT wiring; line 70 registers LoginLog queue consumer)]] - `cites` [EXTRACTED]
- [[commonmiddlewarehandlerauth.go (AuthenticatorPayloadFuncIdentityHandlerAuthorizatorLoginLogToDBUnauthorizedLogOut)]] - `cites` [EXTRACTED]
- [[commonmiddlewarehandlerlogin.go (Login struct + GetUser bcrypt + sys_role lookup)]] - `cites` [EXTRACTED]

#graphify/document #graphify/EXTRACTED #community/Community_53