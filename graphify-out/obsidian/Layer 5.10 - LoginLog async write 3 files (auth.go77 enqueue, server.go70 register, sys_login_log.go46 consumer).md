---
source_file: "docs/superpowers/specs/2026-05-07-login-trace.md"
type: "rationale"
community: "Community 44"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_44
---

# Layer 5.10 - LoginLog async write: 3 files (auth.go:77 enqueue, server.go:70 register, sys_login_log.go:46 consumer)

## Connections
- [[Constitution Principle V - login log is product feature not debug aid (positive + negative path)]] - `rationale_for` [EXTRACTED]
- [[Critical async wrinkle 50-500ms gap between HTTP response and sys_login_log row landing]] - `rationale_for` [EXTRACTED]
- [[Layer 5.11 - Response assembly via SDK default LoginResponse (5-key) vs LogOutUnauthorized (2-key)]] - `references` [EXTRACTED]
- [[Layer 5.9 - JWT TokenGenerator HS256 SDK boundary (stub, FR-010)]] - `references` [EXTRACTED]
- [[Recipe (c) FR-012cFR-011 - sys_login_log row check via sqlite3 with sleep 1 retry loop]] - `references` [EXTRACTED]
- [[Section 4 - Mermaid sequence diagram (11-actor login chain with two SDK rect boundaries)]] - `references` [EXTRACTED]
- [[Track B - write-path CRUD with sys_operation_log (global.OperateLog topic)]] - `semantically_similar_to` [INFERRED]
- [[appadminmodelssys_login_log.go SaveLoginLog queue consumer]] - `references` [EXTRACTED]
- [[cmdapiserver.go (ginmiddlewarerouterCasbinqueueJWT wiring; line 70 registers LoginLog queue consumer)]] - `references` [EXTRACTED]
- [[commonmiddlewarehandlerauth.go (AuthenticatorPayloadFuncIdentityHandlerAuthorizatorLoginLogToDBUnauthorizedLogOut)]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_44