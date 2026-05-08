---
source_file: "x_fork.tree-20260506.md"
type: "rationale"
community: "Community 53"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_53
---

# cmd/api/server.go (gin/middleware/router/Casbin/queue/JWT wiring; line 70 registers LoginLog queue consumer)

## Connections
- [[2026-05-07-login-trace.md (POST apiv1login walkthrough, 805 lines)]] - `cites` [EXTRACTED]
- [[Layer 5.10 - LoginLog async write 3 files (auth.go77 enqueue, server.go70 register, sys_login_log.go46 consumer)]] - `references` [EXTRACTED]
- [[Recommended reading path main.go - cmdapiserver.go - sys_router.go - handlerauth.go]] - `references` [EXTRACTED]
- [[appadminrouter (route registration, mounted by cmdapiserver.go)]] - `references` [EXTRACTED]
- [[cmd (Cobra subcommand entry points; main.go enters here)]] - `references` [EXTRACTED]
- [[main.go (tiny entrypoint, defers to cmd.Execute)]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_53