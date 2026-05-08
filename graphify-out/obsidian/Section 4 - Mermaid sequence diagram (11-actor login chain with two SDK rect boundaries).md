---
source_file: "docs/superpowers/specs/2026-05-07-login-trace.md"
type: "rationale"
community: "Community 28"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_28
---

# Section 4 - Mermaid sequence diagram (11-actor login chain with two SDK rect boundaries)

## Connections
- [[2026-05-07-login-trace.md (POST apiv1login walkthrough, 805 lines)]] - `references` [EXTRACTED]
- [[Layer 5.1 - Route registration v1.POST(login) (sys_router.go71)]] - `references` [EXTRACTED]
- [[Layer 5.10 - LoginLog async write 3 files (auth.go77 enqueue, server.go70 register, sys_login_log.go46 consumer)]] - `references` [EXTRACTED]
- [[Layer 5.11 - Response assembly via SDK default LoginResponse (5-key) vs LogOutUnauthorized (2-key)]] - `references` [EXTRACTED]
- [[Layer 5.2 - gin-jwt LoginHandler SDK boundary (stub, FR-010)]] - `references` [EXTRACTED]
- [[Layer 5.3 - Authenticator (auth.go63-107) - first project callback, defer LoginLogToDB]] - `references` [EXTRACTED]
- [[Layer 5.4 - Captcha verify outer if !=dev (auth.go88) clear-after-verify flag true]] - `references` [EXTRACTED]
- [[Layer 5.5 - Login struct + Login.GetUser sys_user query (login.go9-14, 16-33)]] - `references` [EXTRACTED]
- [[Layer 5.6 - pkg.CompareHashAndPassword bcrypt compare (login.go22) - the only CPU-bound step]] - `references` [EXTRACTED]
- [[Layer 5.7 - GORM role lookup sys_role (login.go27) - 2nd independent SQL round-trip]] - `references` [EXTRACTED]
- [[Layer 5.8 - PayloadFunc flatten (user, role) - jwt.MapClaims (auth.go21-35)]] - `references` [EXTRACTED]
- [[Layer 5.9 - JWT TokenGenerator HS256 SDK boundary (stub, FR-010)]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_28