---
source_file: "docs/superpowers/specs/2026-05-07-login-trace.md"
type: "rationale"
community: "Community 28"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_28
---

# Layer 5.9 - JWT TokenGenerator HS256 SDK boundary (stub, FR-010)

## Connections
- [[FR-010 - cross-repo SDK code requires separate spec (no deep-trace into core)]] - `rationale_for` [EXTRACTED]
- [[Layer 5.10 - LoginLog async write 3 files (auth.go77 enqueue, server.go70 register, sys_login_log.go46 consumer)]] - `references` [EXTRACTED]
- [[Layer 5.8 - PayloadFunc flatten (user, role) - jwt.MapClaims (auth.go21-35)]] - `references` [EXTRACTED]
- [[Section 4 - Mermaid sequence diagram (11-actor login chain with two SDK rect boundaries)]] - `references` [EXTRACTED]
- [[Section 7 - 3 SDK boundaries LoginHandler, TokenGenerator, default LoginResponse]] - `references` [EXTRACTED]
- [[settings.jwt.secret default literal 'go-admin' (HS256 signing key)]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_28