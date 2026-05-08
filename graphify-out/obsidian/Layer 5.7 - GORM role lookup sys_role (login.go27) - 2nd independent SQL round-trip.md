---
source_file: "docs/superpowers/specs/2026-05-07-login-trace.md"
type: "rationale"
community: "Community 28"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_28
---

# Layer 5.7 - GORM role lookup sys_role (login.go:27) - 2nd independent SQL round-trip

## Connections
- [[Layer 5.6 - pkg.CompareHashAndPassword bcrypt compare (login.go22) - the only CPU-bound step]] - `references` [EXTRACTED]
- [[Layer 5.8 - PayloadFunc flatten (user, role) - jwt.MapClaims (auth.go21-35)]] - `references` [EXTRACTED]
- [[Recipe (a) FR-012a - GORM SQL capture (sys_user + sys_role two SELECTs + INSERT sys_login_log)]] - `references` [EXTRACTED]
- [[Section 4 - Mermaid sequence diagram (11-actor login chain with two SDK rect boundaries)]] - `references` [EXTRACTED]
- [[commonmiddlewarehandlerlogin.go (Login struct + GetUser bcrypt + sys_role lookup)]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_28