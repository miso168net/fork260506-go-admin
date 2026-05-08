---
source_file: "docs/superpowers/specs/2026-05-07-login-trace.md"
type: "rationale"
community: "Community 28"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_28
---

# Layer 5.6 - pkg.CompareHashAndPassword bcrypt compare (login.go:22) - the only CPU-bound step

## Connections
- [[Layer 5.5 - Login struct + Login.GetUser sys_user query (login.go9-14, 16-33)]] - `references` [EXTRACTED]
- [[Layer 5.7 - GORM role lookup sys_role (login.go27) - 2nd independent SQL round-trip]] - `references` [EXTRACTED]
- [[Section 4 - Mermaid sequence diagram (11-actor login chain with two SDK rect boundaries)]] - `references` [EXTRACTED]
- [[commonmiddlewarehandlerlogin.go (Login struct + GetUser bcrypt + sys_role lookup)]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_28