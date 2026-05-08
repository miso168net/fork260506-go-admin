---
type: community
cohesion: 0.20
members: 12
---

# Community 44

**Cohesion:** 0.20 - loosely connected
**Members:** 12 nodes

## Members
- [[Constitution  current plan reference (in SpecKit block)]] - rationale - CLAUDE.md
- [[Constitution Principle V - login log is product feature not debug aid (positive + negative path)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Critical async wrinkle 50-500ms gap between HTTP response and sys_login_log row landing]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[JWT claims 8-key shape 6 business (identityroleidrolekeynicedatascoperolename) + 2 envelope (exporig_iat)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[JWT decode procedure (URL-safe base64; cut -d. -f2)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Layer 5.10 - LoginLog async write 3 files (auth.go77 enqueue, server.go70 register, sys_login_log.go46 consumer)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Recipe (b) FR-012b - JWT claim decode (8-key shape verification)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Recipe (c) FR-012cFR-011 - sys_login_log row check via sqlite3 with sleep 1 retry loop]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Recipe (d) FR-013 - Failure path wrong password gets HTTP 200+code400, msg='登录失败' only in DB row]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Section 6 - Runtime verification recipes (predict-execute-compare)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[SpecKit STARTEND marker block]] - rationale - CLAUDE.md
- [[Track B - write-path CRUD with sys_operation_log (global.OperateLog topic)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/Community_44
SORT file.name ASC
```

## Connections to other communities
- 6 edges to [[_COMMUNITY_Community 28]]
- 3 edges to [[_COMMUNITY_Community 53]]
- 1 edge to [[_COMMUNITY_Community 78]]
- 1 edge to [[_COMMUNITY_Community 69]]

## Top bridge nodes
- [[Layer 5.10 - LoginLog async write 3 files (auth.go77 enqueue, server.go70 register, sys_login_log.go46 consumer)]] - degree 10, connects to 3 communities
- [[Section 6 - Runtime verification recipes (predict-execute-compare)]] - degree 5, connects to 2 communities
- [[Recipe (b) FR-012b - JWT claim decode (8-key shape verification)]] - degree 3, connects to 1 community
- [[SpecKit STARTEND marker block]] - degree 2, connects to 1 community
- [[Track B - write-path CRUD with sys_operation_log (global.OperateLog topic)]] - degree 2, connects to 1 community