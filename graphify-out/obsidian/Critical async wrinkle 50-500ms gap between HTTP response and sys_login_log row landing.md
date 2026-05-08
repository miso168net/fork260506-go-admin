---
source_file: "docs/superpowers/specs/2026-05-07-login-trace.md"
type: "rationale"
community: "Community 44"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_44
---

# Critical async wrinkle: 50-500ms gap between HTTP response and sys_login_log row landing

## Connections
- [[Layer 5.10 - LoginLog async write 3 files (auth.go77 enqueue, server.go70 register, sys_login_log.go46 consumer)]] - `rationale_for` [EXTRACTED]
- [[Recipe (c) FR-012cFR-011 - sys_login_log row check via sqlite3 with sleep 1 retry loop]] - `rationale_for` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_44