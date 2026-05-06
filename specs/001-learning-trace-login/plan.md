# Implementation Plan: Learning Walkthrough — Trace `POST /api/v1/login` End-to-End

**Branch**: `001-learning-trace-login` | **Date**: 2026-05-07 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-learning-trace-login/spec.md`

## Summary

Produce three artifacts that let a learner who has only Docker installed (no Go,
no MySQL) bring up the go-admin backend and trace the full `POST /api/v1/login`
chain end-to-end. The artifacts are: (a) a `docker-compose.learning.yml` +
multi-stage `Dockerfile.learning` pair so `docker compose up --build` produces
a running server with SQLite, (b) a zh-TW learning document at
`docs/learning/login-trace.md` that walks each layer of the login request with
exact `file:line` references, a Mermaid sequence diagram, and runtime
verification recipes (GORM SQL via `enableddb: true` toggle, JWT claim decode,
`sys_login_log` row inspection on a freshly-recreated container), and (c) the
SpecKit artifacts under `specs/001-learning-trace-login/` that document this
plan. Zero changes to existing source files (FR-016 / SC-006).

## Technical Context

**Language/Version**:
- Builder image: `golang:1.24-alpine` (matches `go.mod` declaration `go 1.24`)
- Runtime image: `alpine` (latest)
- Documentation: Markdown + Mermaid (`sequenceDiagram`), zh-TW prose per FR-017

**Primary Dependencies**:
- Build-stage tooling: `gcc`, `g++`, `libc6-compat` (CGO toolchain required by
  `mattn/go-sqlite3`)
- Runtime tooling: `ca-certificates`, `tzdata`, `libc6-compat`
- Application-level dependencies are pre-existing in `go.mod` and unchanged:
  Gin, GORM, Casbin, gin-jwt (via `go-admin-core` SDK), `mattn/go-sqlite3`,
  Cobra, swaggo

**Storage**: SQLite via `mattn/go-sqlite3` (CGO build tag `sqlite3`). The
seeded `go-admin-db.db` from the repo root is `COPY`'d into the runtime image.
No host volume mount in the default compose definition; a commented-out volume
block is provided per Q3 clarification.

**Testing**: Manual acceptance per spec FRs and SCs. This is a documentation +
container-artifacts feature; there are no unit tests to author. Acceptance is
binary per FR clause + recipe execution per FR-012 / FR-013.

**Target Platform**:
- Docker host: Linux / macOS / WSL2 (per Assumptions; native Windows out of
  scope, deferred from clarify pass)
- Container OS: Alpine Linux x86_64

**Project Type**: Documentation + container deliverables (no new application
code). The "source code" of this feature is the multi-stage Dockerfile and the
zh-TW learning prose.

**Performance Goals** (per spec SC-001 / SC-005):
- First `docker compose up --build` ≤ 10 minutes wall-clock (network-bound on
  base-image pull and module download)
- Subsequent `docker compose up` ≤ 30 seconds (Docker layer cache)
- Time-to-JWT after first build ≤ 5 minutes (curl + dev-mode captcha bypass
  is canonical path)

**Constraints**:
- FR-016 / SC-006: zero edits to existing `app/`, `common/`, `cmd/`, `config/`,
  `main.go`, `docker-compose.yml`, `Dockerfile`. Verifiable via
  `git diff --stat main..HEAD`.
- FR-017: zh-TW prose; code excerpts and quoted comments preserved verbatim.
- SDK boundary discipline (FR-010): gin-jwt `LoginHandler` and JWT
  `TokenGenerator` are NOT traced; pointers to `fork260506-go-admin-core/`
  serve as exit ramps.
- Single-service compose (no MySQL / Redis side cars) per Q3-default.

**Scale/Scope**: Single learner. Three deliverables (one compose file, one
Dockerfile, one Markdown document). Document target length 600–1000 lines
(SC-004).

## Constitution Check

*Gate evaluation against `.specify/memory/constitution.md` v1.0.0.*

| # | Principle | Status | Justification |
|---|-----------|--------|---------------|
| I | Upstream Compatibility & Fork Discipline | **PASS** | Zero edits to fork code (FR-016 enforces). All new files live at root or under `docs/learning/` and `specs/`. Future upstream rebase is unaffected. |
| II | API & Permission Contract Stability | **N/A** | No REST endpoint, Casbin policy, or JWT claim shape is introduced or modified. The contracts in `contracts/` are *documentation of existing behaviour* for learning purposes, not new contracts. |
| III | Test Discipline (NON-NEGOTIABLE) | **N/A — auth-path verification preserved** | Principle III mandates tests for new auth/Casbin/JWT/persistence code. This feature adds **zero** such code. The existing login path's behaviour is *verified* by FR-012 / FR-013 recipes (manual but explicit) — confirming test discipline rather than violating it. |
| IV | Spec-Driven Change for Non-Trivial Work | **PASS** | This very plan is the SpecKit flow (`/speckit-specify` → `/speckit-clarify` → `/speckit-plan` → `/speckit-tasks` → `/speckit-implement`). Compliance is met by following the workflow itself. |
| V | Observability & Audit Logs Are Load-Bearing | **PASS — actively verified** | FR-011 + FR-012(c) require the plan / learning doc to *verify* `sys_login_log` row emission on successful login. We are not removing, sampling-down, or short-circuiting any log emission. Recipe (c) on a freshly-recreated container is the empirical check. |

**No violations. Complexity Tracking section omitted.**

## Project Structure

### Documentation (this feature)

```text
specs/001-learning-trace-login/
├── plan.md              # This file (/speckit-plan output)
├── spec.md              # /speckit-specify output (already in branch)
├── research.md          # Phase 0 output — login-chain layer catalogue, LoginLog queue note, Dockerfile.learning research
├── data-model.md        # Phase 1 output — sys_user / sys_role / sys_login_log shape + JWT claim shape (referenced, not added)
├── quickstart.md        # Phase 1 output — minimal "from clone to JWT" walkthrough
├── contracts/
│   ├── post-login.md    # request/response contract for POST /api/v1/login (documented, not new)
│   └── get-captcha.md   # request/response contract for GET /api/v1/captcha (documented, not new)
└── tasks.md             # Phase 2 output — created by /speckit-tasks (NOT by /speckit-plan)
```

### Source Code (repository root)

```text
# Existing files — UNCHANGED (FR-016, SC-006)
app/                     # admin endpoints, models, services, routers (read-only this feature)
common/middleware/       # auth.go, login.go (read-only; quoted in learning doc)
cmd/                     # cobra commands (read-only)
config/                  # settings.yml, settings.sqlite.yml, db.sql, ... (read-only)
docker-compose.yml       # existing production-style compose (UNCHANGED)
Dockerfile               # existing single-stage Dockerfile (UNCHANGED)
go-admin-db.db           # seeded SQLite file (UNCHANGED; COPY'd into Dockerfile.learning runtime stage)
main.go                  # entrypoint (read-only)

# New files added by /speckit-implement
docker-compose.learning.yml          # NEW — single-service compose, port 8000, no DB volume by default, commented-out volume hint
Dockerfile.learning                  # NEW — multi-stage: golang:1.24-alpine builder (CGO + sqlite3 tag) + alpine runtime
docs/
└── learning/
    └── login-trace.md               # NEW — zh-TW learning document (600–1000 lines target)
```

**Structure Decision**: This feature does not introduce a conventional
`src/` + `tests/` source layout because it adds zero application code. The
"source" of this feature is the multi-stage build description plus prose. The
deliverables are placed at canonical locations (root for Docker artifacts,
`docs/learning/` for prose) so they sit beside but never replace the existing
production-style `Dockerfile` / `docker-compose.yml`.

## Complexity Tracking

> **All Constitution gates PASS or are N/A. No violations to justify.**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| _(none)_ | _(n/a)_ | _(n/a)_ |
