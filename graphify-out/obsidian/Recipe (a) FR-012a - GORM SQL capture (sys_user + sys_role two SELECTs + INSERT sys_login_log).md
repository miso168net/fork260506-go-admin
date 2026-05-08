---
source_file: "docs/superpowers/specs/2026-05-07-login-trace.md"
type: "rationale"
community: "Community 28"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_28
---

# Recipe (a) FR-012a - GORM SQL capture (sys_user + sys_role two SELECTs + INSERT sys_login_log)

## Connections
- [[Layer 5.5 - Login struct + Login.GetUser sys_user query (login.go9-14, 16-33)]] - `references` [EXTRACTED]
- [[Layer 5.7 - GORM role lookup sys_role (login.go27) - 2nd independent SQL round-trip]] - `references` [EXTRACTED]
- [[Section 6 - Runtime verification recipes (predict-execute-compare)]] - `references` [EXTRACTED]
- [[enableddb flip (Dockerfile.learning sed false-true) baked into runtime image]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_28