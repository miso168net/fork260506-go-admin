---
source_file: "docs/superpowers/specs/2026-05-07-login-trace.md"
type: "rationale"
community: "Community 28"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_28
---

# Layer 5.5 - Login struct + Login.GetUser sys_user query (login.go:9-14, :16-33)

## Connections
- [[Layer 5.4 - Captcha verify outer if !=dev (auth.go88) clear-after-verify flag true]] - `references` [EXTRACTED]
- [[Layer 5.6 - pkg.CompareHashAndPassword bcrypt compare (login.go22) - the only CPU-bound step]] - `references` [EXTRACTED]
- [[Recipe (a) FR-012a - GORM SQL capture (sys_user + sys_role two SELECTs + INSERT sys_login_log)]] - `references` [EXTRACTED]
- [[Section 4 - Mermaid sequence diagram (11-actor login chain with two SDK rect boundaries)]] - `references` [EXTRACTED]
- [[commonmiddlewarehandlerlogin.go (Login struct + GetUser bcrypt + sys_role lookup)]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_28