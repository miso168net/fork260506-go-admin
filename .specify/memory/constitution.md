<!--
Sync Impact Report
==================
Version change: TEMPLATE (placeholders) → 1.0.0 (initial ratification)
Modified principles: N/A (first concrete fill — no prior named principles)
Added sections:
  - Core Principles I–V (concrete content)
  - Technology Stack & Constraints (Section 2)
  - Development Workflow (Section 3)
  - Governance (with amendment procedure, versioning policy, compliance review)
Removed sections: none
Templates requiring updates:
  - .specify/templates/plan-template.md (✅ aligned — Constitution Check gate is
    a placeholder filled at /speckit-plan time and references this file)
  - .specify/templates/spec-template.md (✅ aligned — no constitution-driven
    mandatory sections changed)
  - .specify/templates/tasks-template.md (✅ updated — added a Principle III
    carve-out: tests are MANDATORY for auth/authz/JWT/data-scope/persistence
    paths even when otherwise OPTIONAL)
  - CLAUDE.md (✅ aligned — already referenced as runtime guidance; no edits
    required since Governance simply declares it canonical)
Follow-up TODOs:
  - None blocking. RATIFICATION_DATE recorded as today (2026-05-07); if a
    different historical adoption date is preferred, raise as a PATCH amendment.
-->

# go-admin (fork260506) Constitution

## Core Principles

### I. Upstream Compatibility & Fork Discipline

Every change to forked code MUST be traceable against the upstream
`go-admin-team/go-admin` lineage recorded in `x_fork.branch-origin.md`. Local
modifications SHOULD be isolated (by commit prefix, path, or branch) so a
future rebase against upstream can apply cleanly. Diverging from upstream APIs
without a recorded reason is forbidden.

**Rationale**: This repository is a fork, not a hard divergence. Losing the
ability to pull upstream security and bug fixes is the worst outcome we can
inflict on ourselves; structural traceability is what protects us.

### II. API & Permission Contract Stability

Public REST endpoints under `app/admin/router/`, the Casbin policy schema, and
the JWT claim shape are CONTRACTS. Backward-incompatible changes (removed
endpoints, changed response field types, altered permission semantics) MUST be
flagged in `spec.md`, MUST ship with a migration note under `docs/`, and MUST
bump the project's externally-visible version (MINOR for deprecation, MAJOR
for removal). Additive changes do not require a version bump but SHOULD carry
tests per Principle III.

**Rationale**: go-admin is consumed by separate front-ends (vue2, vue3, antd)
and downstream tools. Silent breakage is expensive across that surface.

### III. Test Discipline for Production Logic (NON-NEGOTIABLE)

Any code path that handles authentication, authorization (Casbin), data scope
filtering, JWT validation, or persistence MUST land with at least one test
covering the success path and one covering the rejection path. Code-generation
output and pure config wiring are exempt. New tests MUST be written and FAIL
before the implementation lands — Red → Green → Refactor.

**Rationale**: Upstream README still lists "TODO: unit test". This fork closes
that gap deliberately for the surfaces where bugs do not surface until
production: auth, permissions, and data scope.

### IV. Spec-Driven Change for Non-Trivial Work

Any change that adds a new module under `app/`, alters a database migration,
or modifies router/middleware composition MUST go through the SpecKit flow:
`/speckit-specify` → `/speckit-clarify` (when `NEEDS CLARIFICATION` markers
exist) → `/speckit-plan` → `/speckit-tasks` → `/speckit-implement`. One-line
bug fixes, typo corrections, and dependency bumps are exempt and may go
direct.

**Rationale**: SpecKit is the project's chosen workflow
(`.specify/init-options.json` records `speckit_version: 0.8.5`). Skipping it
discards the audit trail and Constitution Check gate the workflow provides.

### V. Observability & Audit Logs Are Load-Bearing

Operation log, login log, and structured request logging are PRODUCT features,
not debugging aids. New write-path handlers MUST emit an operation log entry
on both success and failure. New auth-path handlers MUST emit a login log
entry. Removing, short-circuiting, or sampling-down these emissions is
forbidden without an explicit constitution amendment.

**Rationale**: These logs are advertised features in the README and the
primary mechanism by which operators detect abuse. Treating them as optional
breaks the product, not just telemetry.

## Technology Stack & Constraints

- **Language**: Go (`go.mod` declares `go 1.24`). Contributors MUST NOT
  introduce constructs requiring a newer toolchain without first bumping
  `go.mod` and noting the change in the relevant plan.
- **HTTP framework**: Gin. New routes MUST register through the existing
  router composition in `app/admin/router/`. Introducing a parallel HTTP
  framework requires an amendment.
- **ORM**: GORM with the project's chosen drivers (mysql, postgres, sqlite,
  sqlserver). Raw SQL is allowed only for migrations and reporting queries.
- **AuthN / AuthZ**: JWT for authentication, Casbin for authorization. No
  alternative auth schemes without an amendment.
- **API documentation**: Swagger via `swaggo`. New endpoints MUST include
  swag annotations sufficient to regenerate the docs.
- **CLI**: Cobra-based multi-command pattern under `cmd/`. New commands MUST
  follow the existing structure (server, migrate, version, etc.).
- **Knowledge graph**: `graphify-out/` is regenerated via `graphify update .`
  after code changes. Contributors SHOULD run it before opening a PR that
  touches more than a handful of files (per the project `CLAUDE.md`).
- **Companion repos**: `go-admin-core`, `go-admin-ui`, `gorm-adapter`,
  `redis-watcher`, `redisqueue`, and the docs site are sibling forks under
  the same workspace. Cross-repo changes MUST be coordinated, not silently
  desynchronized.

## Development Workflow

- **Runtime guidance**: `CLAUDE.md` (repository root and per-subdirectory) is
  the authoritative runtime guide for AI-assisted contributors. When this
  constitution and `CLAUDE.md` conflict, the constitution wins; `CLAUDE.md`
  MUST be updated to match.
- **Branching**: SpecKit feature branches use sequential numbering
  (`.specify/init-options.json` declares `branch_numbering: sequential`).
  Manual branches MUST follow the same convention or carry a clear prefix
  (`fix/`, `chore/`, `docs/`).
- **Commit messages**: SHOULD use conventional prefixes (`feat:`, `fix:`,
  `docs:`, `chore:`, `refactor:`) — established by recent history (e.g.
  `d191ed3 chore: integrate SpecKit toolkit`).
- **Push to remote**: Pushing to `origin` is a user-confirmed action. AI
  contributors MUST NOT push without explicit approval, mirroring the user's
  global guidance.
- **Code review**: PRs MUST verify each touched principle via the
  Constitution Check gate in `.specify/templates/plan-template.md`.
  Violations either MUST be removed or MUST be justified in the plan's
  Complexity Tracking table — silent waivers are prohibited.
- **SpecKit hooks**: Pre/post hooks in `.specify/extensions.yml` are
  executed as configured. Contributors MUST NOT disable the mandatory
  `before_constitution` git initialization hook without an amendment.

## Governance

This constitution supersedes ad-hoc practices in this repository. When the
constitution conflicts with any other project document (READMEs, code
comments, prior conventions), the constitution wins until the conflicting
document is updated.

**Amendment procedure**:

1. Open an amendment draft (typically via `/speckit-constitution`).
2. State the version bump and reasoning (MAJOR / MINOR / PATCH per the
   versioning policy below).
3. Run the consistency propagation pass on `.specify/templates/*` and
   `CLAUDE.md`; record results in the Sync Impact Report at the top of this
   file.
4. Land via PR with explicit reviewer sign-off on the principle change.

**Versioning policy**:

- **MAJOR**: Removed or redefined principle; backward-incompatible governance
  change.
- **MINOR**: Added principle, materially expanded section, or new mandatory
  workflow step.
- **PATCH**: Wording, typo, clarification, or non-semantic refinement.

**Compliance review**: Every `/speckit-plan` run MUST execute the Constitution
Check gate against this file. Violations either MUST be removed during
planning or MUST be justified in the plan's Complexity Tracking table.

**Runtime guidance file**: `CLAUDE.md` (repository root) is the canonical
runtime guidance referenced by this constitution.

**Version**: 1.0.0 | **Ratified**: 2026-05-07 | **Last Amended**: 2026-05-07
