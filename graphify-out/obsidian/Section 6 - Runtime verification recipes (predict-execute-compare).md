---
source_file: "docs/superpowers/specs/2026-05-07-login-trace.md"
type: "rationale"
community: "Community 44"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_44
---

# Section 6 - Runtime verification recipes (predict-execute-compare)

## Connections
- [[2026-05-07-login-trace.md (POST apiv1login walkthrough, 805 lines)]] - `references` [EXTRACTED]
- [[Recipe (a) FR-012a - GORM SQL capture (sys_user + sys_role two SELECTs + INSERT sys_login_log)]] - `references` [EXTRACTED]
- [[Recipe (b) FR-012b - JWT claim decode (8-key shape verification)]] - `references` [EXTRACTED]
- [[Recipe (c) FR-012cFR-011 - sys_login_log row check via sqlite3 with sleep 1 retry loop]] - `references` [EXTRACTED]
- [[Recipe (d) FR-013 - Failure path wrong password gets HTTP 200+code400, msg='登录失败' only in DB row]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_44