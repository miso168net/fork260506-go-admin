---
source_file: "docs/superpowers/specs/2026-05-07-login-trace.md"
type: "rationale"
community: "Community 28"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_28
---

# Layer 5.8 - PayloadFunc flatten (user, role) -> jwt.MapClaims (auth.go:21-35)

## Connections
- [[Layer 5.7 - GORM role lookup sys_role (login.go27) - 2nd independent SQL round-trip]] - `references` [EXTRACTED]
- [[Layer 5.9 - JWT TokenGenerator HS256 SDK boundary (stub, FR-010)]] - `references` [EXTRACTED]
- [[Recipe (b) FR-012b - JWT claim decode (8-key shape verification)]] - `references` [EXTRACTED]
- [[Section 4 - Mermaid sequence diagram (11-actor login chain with two SDK rect boundaries)]] - `references` [EXTRACTED]
- [[commonmiddlewarehandlerauth.go (AuthenticatorPayloadFuncIdentityHandlerAuthorizatorLoginLogToDBUnauthorizedLogOut)]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_28