---
type: community
cohesion: 0.12
members: 18
---

# Community 26

**Cohesion:** 0.12 - loosely connected
**Members:** 18 nodes

## Members
- [[Dockerfile (upstream single-stage; assumes .main pre-built)]] - rationale - x_fork.tree-20260506.md
- [[Dockerfile.learning (multi-stage golang1.24-alpine builder; flips enableddb to true via sed)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Makefile (build  build-sqlite  build-linux  run  stop targets)]] - rationale - x_fork.tree-20260506.md
- [[Repo root fork260506-go-admin]] - rationale - x_fork.tree-20260506.md
- [[Section 2 - Bring up backend with `docker compose -f docker-compose.learning.yml up --build`]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[config (settings.yml family + SQL seed)]] - rationale - x_fork.tree-20260506.md
- [[configsettings.sqlite.yml (driver sqlite3; used by Dockerfile.learning)]] - rationale - x_fork.tree-20260506.md
- [[configsettings.yml (default mysql 127.0.0.13306 - default-MySQL trap)]] - rationale - x_fork.tree-20260506.md
- [[docker-compose.learning.yml (SQLite-backed learning compose, opt-in DB persistence)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[docker-compose.yml (upstream production compose; needs make build-linux)]] - rationale - x_fork.tree-20260506.md
- [[docsadmin (swaggo-generated swagger spec embedded in binary)]] - rationale - x_fork.tree-20260506.md
- [[enableddb flip (Dockerfile.learning sed false-true) baked into runtime image]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[staticform-generator (precompiled vue-form-making bundle; ~1600 god-node functions in chunk-vendors.js)]] - rationale - x_fork.tree-20260506.md
- [[template (code-generator templates used by appotherapistoolsgen.go)]] - rationale - x_fork.tree-20260506.md
- [[test (code-gen behavior tests + corresponding templates)]] - rationale - x_fork.tree-20260506.md
- [[x_fork.branch-origin.md (fork lineage from upstream a5cc0a9)]] - rationale - x_fork.tree-20260506.md
- [[x_fork.graphify-20260506.md (first graphify session log)]] - rationale - x_fork.tree-20260506.md
- [[x_fork.tree-20260506.md (annotated repo tree)]] - document - x_fork.tree-20260506.md

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/Community_26
SORT file.name ASC
```

## Connections to other communities
- 2 edges to [[_COMMUNITY_Community 53]]
- 1 edge to [[_COMMUNITY_Community 54]]
- 1 edge to [[_COMMUNITY_Community 69]]
- 1 edge to [[_COMMUNITY_Community 49]]
- 1 edge to [[_COMMUNITY_Community 70]]
- 1 edge to [[_COMMUNITY_Community 28]]

## Top bridge nodes
- [[Repo root fork260506-go-admin]] - degree 16, connects to 5 communities
- [[Section 2 - Bring up backend with `docker compose -f docker-compose.learning.yml up --build`]] - degree 3, connects to 1 community
- [[enableddb flip (Dockerfile.learning sed false-true) baked into runtime image]] - degree 3, connects to 1 community