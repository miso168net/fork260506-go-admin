---
source_file: "docs/superpowers/specs/2026-05-07-login-trace.md"
type: "rationale"
community: "Community 28"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_28
---

# Layer 5.11 - Response assembly via SDK default LoginResponse (5-key) vs LogOut/Unauthorized (2-key)

## Connections
- [[FR-010 - cross-repo SDK code requires separate spec (no deep-trace into core)]] - `rationale_for` [EXTRACTED]
- [[Layer 5.10 - LoginLog async write 3 files (auth.go77 enqueue, server.go70 register, sys_login_log.go46 consumer)]] - `references` [EXTRACTED]
- [[Login response 5-key shape codecurrentAuthorityexpiresuccesstoken]] - `references` [EXTRACTED]
- [[Section 4 - Mermaid sequence diagram (11-actor login chain with two SDK rect boundaries)]] - `references` [EXTRACTED]
- [[Section 7 - 3 SDK boundaries LoginHandler, TokenGenerator, default LoginResponse]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_28