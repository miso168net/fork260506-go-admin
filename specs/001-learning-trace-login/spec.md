# Feature Specification: Learning Walkthrough — Trace `POST /api/v1/login` End-to-End

**Feature Branch**: `001-learning-trace-login`
**Created**: 2026-05-07
**Status**: Draft
**Input**: Distilled from a `superpowers:brainstorming` session whose intent was *"build initial spec-kit documents for this fork; learner does not maintain the project"*. Selections: learning-oriented (not customization), depth = one full request chain, chain = login (`POST /api/v1/login`), method = run-and-read, runtime = custom Docker Compose with multi-stage Dockerfile (host needs zero Go).

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Bring the backend up with one Docker command and obtain a JWT (Priority: P1) — MVP

As a learner who has just cloned this fork and has Docker installed (but not necessarily Go), I want to start the entire go-admin backend with a single command, then use `curl` to exchange the default credentials for a JWT — without needing the `go-admin-ui` frontend, without installing Go, and without provisioning MySQL.

**Why this priority**: This is the irreducible MVP. Without it, every later step is blocked. Once a learner has a JWT in hand and a running server they can hit, the other stories become incremental — but if startup fails, nothing else matters.

**Independent Test**: On a clean machine that has Docker but no Go toolchain and no preexisting `go-admin` image, run the documented startup command and the documented login `curl`. Success = receive HTTP 200 with a JSON payload containing a non-empty `token` field, AND that token decodes to a JWT with the expected claims (`identity`, `rolekey`, `datascope`, …).

**Acceptance Scenarios**:

1. **Given** a clean host with only Docker installed, **When** the learner runs `docker compose -f docker-compose.learning.yml up --build`, **Then** the server logs report listening on port 8000 within 10 minutes (first build) or 30 seconds (subsequent runs), and `GET http://localhost:8000/swagger/admin/index.html` returns 200.
2. **Given** the server is running, **When** the learner sends `POST /api/v1/login` with body `{"username":"admin","password":"123456","code":"0","uuid":"0"}`, **Then** the response is HTTP 200 with `data.token` populated.
3. **Given** a JWT obtained in scenario 2, **When** the learner sends a protected request (e.g. `GET /api/v1/getinfo`) with header `Authorization: Bearer <token>`, **Then** the response is HTTP 200.

---

### User Story 2 — Trace the login request through every layer of code (Priority: P2)

As a learner who can already get a JWT (US1 is complete), I want a written walkthrough that explains exactly what happens between the moment my `curl` hits the server and the moment the JWT comes back — naming each layer, pointing at the precise file and line, and showing a sequence diagram so I can hold the whole flow in my head.

**Why this priority**: This is the actual learning payload. US1 proves the system works; US2 explains *how* it works. Without US2 the learner has run a black box. With US2 they understand the auth path well enough to reason about other endpoints.

**Independent Test**: After reading the produced walkthrough once (without re-reading), the learner can (a) name every layer of the login chain in order, (b) point to the file path and line for ≥80% of the layers without consulting the doc, and (c) reproduce the sequence diagram from memory at a coarse level (each layer + arrow direction).

**Acceptance Scenarios**:

1. **Given** the produced learning document, **When** the learner reads it once, **Then** they can describe each of the following layers in their own words and cite a file:line for each: (a) router registration, (b) gin-jwt `LoginHandler` entry (SDK boundary), (c) `Authenticator` in `common/middleware/handler/auth.go`, (d) captcha verification, (e) `Login.GetUser` GORM query against `sys_user` and `sys_role`, (f) bcrypt password comparison, (g) `PayloadFunc` claims construction, (h) JWT signing (SDK boundary), (i) login log emission, (j) response assembly.
2. **Given** the produced learning document, **When** the learner is asked "where in the codebase does Casbin/data-scope filtering happen for the login endpoint?", **Then** they correctly answer "it doesn't — login is the auth issuance step and bypasses Casbin", with a one-sentence rationale.
3. **Given** the sequence diagram in the document, **When** the learner is shown a stripped-down version with arrows missing, **Then** they can fill in the direction of every arrow correctly.

---

### User Story 3 — Verify understanding with runtime evidence (Priority: P3)

As a learner who has read the walkthrough (US2 is complete), I want to confirm my understanding by comparing my code-derived predictions against what the running system actually does — same approach engineers use to debug for real.

**Why this priority**: Passive reading creates an illusion of understanding. Forcing predictions and checking them is what converts "I read the words" into durable knowledge. P3 because it depends on US1+US2; not in MVP because the MVP can ship without verification rituals if time is short.

**Independent Test**: For each verification target the learner records (a) what they predicted from reading code and (b) what actually happened. Substantive matches = pass. Cosmetic differences (timestamps, whitespace, ordering) are ignored.

**Acceptance Scenarios**:

1. **Given** US1 is running and US2 has been read, **When** the learner predicts the SQL that GORM will execute for the user lookup and the role lookup, **Then** the actual SQL captured from server logs (or a transient `enableddb: true` config) matches the predicted shape (table, where-clause columns, parameters) — only cosmetic differences allowed.
2. **Given** a JWT obtained from a successful login, **When** the learner predicts the JSON claims and decodes the token (e.g. via `cut -d. -f2 | base64 -d` or jwt.io), **Then** every predicted claim key appears with the predicted value type; values for `identity`, `rolekey`, `datascope` match the seeded admin user.
3. **Given** the `sys_login_log` table state before a login attempt, **When** the learner sends a successful login then queries the table, **Then** a new row exists with the expected fields (username, login result, timestamp), confirming Constitution Principle V applies to this endpoint.
4. **Given** the running server, **When** the learner intentionally sends a wrong password, an empty captcha field, and an expired token (three separate failure cases), **Then** each produces the predicted error code and the predicted log line.

---

### Edge Cases

- **Captcha bypass not understood**: Learner sees `code` and `uuid` fields and assumes they need to fetch a captcha image first. Document MUST explicitly call out the dev-mode bypass (`code:"0", uuid:"0"`) with a citation to `common/middleware/handler/auth.go:51-66` swagger comments.
- **Default config targets MySQL**: `config/settings.yml` ships pointing at MySQL `127.0.0.1:3306`. Without intervention, the learner gets an unhelpful "connection refused" error. The Docker Compose path MUST bake `config/settings.sqlite.yml` as the active config so the SQLite-based path is the default.
- **CGO requirement for SQLite**: `mattn/go-sqlite3` requires `CGO_ENABLED=1` and `-tags sqlite3`. The custom Dockerfile MUST set both (graphify report flagged this in its `CGO SQLite Build Note` community).
- **Default credentials missing or rotated**: If `admin/123456` does not work, document a one-liner to re-seed via the project's migration command, OR cite the `db.sql` seed file path so the learner can confirm the seeded password hash.
- **SDK trace going off-cliff**: Two layers (`gin-jwt LoginHandler`, JWT signing) live in `fork260506-go-admin-core/`, not in this repo. Document MUST mark those as SDK-boundary stops and tell the learner where to go if they want to keep tracing — without actually tracing into the SDK in this spec.
- **Build cache cold start time**: First `docker compose up --build` may take 5–10 min on a slow network. Document MUST set this expectation upfront so learners do not abort.
- **Existing `docker-compose.yml` collision**: The project already ships a `docker-compose.yml` and `Dockerfile` for production-style deploy. The custom learning artifacts MUST be named distinctly (`docker-compose.learning.yml`, `Dockerfile.learning`) so existing flows are untouched.

## Requirements *(mandatory)*

### Functional Requirements

#### Runtime artifacts (deliverables that produce a runnable system)

- **FR-001**: A new file `docker-compose.learning.yml` MUST be added at the repository root, defining a single service `go-admin-learning` that builds from `Dockerfile.learning`, exposes port `8000:8000`, and bind-mounts at minimum a writable log path (e.g. `./temp:/temp`).
- **FR-002**: A new file `Dockerfile.learning` MUST be added at the repository root as a multi-stage build: a `builder` stage based on `golang:1.24-alpine` with `gcc`, `g++`, and `libc6-compat` available, running `CGO_ENABLED=1 go build -tags sqlite3 -ldflags="-w -s" -o /out/go-admin .`; and a `runtime` stage based on `alpine` with `ca-certificates`, `tzdata`, and `libc6-compat` only, copying the built binary, `config/settings.sqlite.yml` as `/config/settings.yml`, and `go-admin-db.db` into the image.
- **FR-003**: The runtime container MUST start the API server with `["/go-admin", "server", "-c", "/config/settings.yml"]` and listen on `0.0.0.0:8000`.
- **FR-004**: The existing `docker-compose.yml` and `Dockerfile` MUST remain unmodified. The new artifacts MUST coexist alongside them without breaking `make build`, `make build-linux`, or `make run`.

#### Learning document (the actual learning payload)

- **FR-005**: A new file `docs/learning/login-trace.md` MUST be created. It MUST contain, in this order: (1) Goal & Prerequisites, (2) Bring-up via Docker Compose, (3) Login curl example with response, (4) Sequence diagram of the login chain, (5) Layer-by-layer trace with `file:line` references, (6) Runtime verification recipes, (7) Known SDK boundaries, (8) Next steps.
- **FR-006**: Section 2 (Bring-up) MUST give the exact command (`docker compose -f docker-compose.learning.yml up --build`), expected first-run build time, and a sanity check that opens `http://localhost:8000/swagger/admin/index.html` to confirm the server is up.
- **FR-007**: Section 3 (Login curl) MUST contain the exact curl command including the dev-mode captcha bypass (`code:"0", uuid:"0"`) and explain *why* it works, citing `common/middleware/handler/auth.go:51-66` (swagger comments documenting the bypass).
- **FR-008**: Section 4 (Sequence diagram) MUST be authored as Mermaid (`sequenceDiagram` block). It MUST show: client → router → gin-jwt LoginHandler (SDK) → Authenticator → captcha.Verify → Login.GetUser. Inside `Login.GetUser` the diagram MUST depict two distinct database round-trips (one to `sys_user`, one to `sys_role`) with `pkg.CompareHashAndPassword` between them. After `GetUser` returns: PayloadFunc → JWT TokenGenerator (SDK) → LoginLog hook → response.
- **FR-009**: Section 5 (Layer-by-layer trace) MUST list every layer named in FR-008 as its own subsection, each containing: a single-paragraph plain-language role description, an exact `file:line` reference, and a minimal code excerpt (≤15 lines) from that file.
- **FR-010**: Section 5 MUST mark gin-jwt `LoginHandler` and JWT `TokenGenerator` as SDK boundaries, naming the sibling-repo path `fork260506-go-admin-core/sdk/pkg/jwtauth/` and stating explicitly that this spec does not trace into the SDK; learners wanting more depth must open a separate spec.
- **FR-011**: Section 5 MUST identify the line where the login log is emitted and verify (in the verification section) that a row appears in `sys_login_log` after a successful login, satisfying Constitution Principle V for the login path.
- **FR-012**: Section 6 (Verification) MUST include three concrete recipes: (a) capture the actual SQL emitted by GORM and compare to predicted SQL, (b) decode the issued JWT and compare claims to predicted claims, (c) inspect `sys_login_log` after a successful login.
- **FR-013**: Section 6 MUST include at least one negative-path recipe (wrong password, missing captcha field, or expired token) showing the predicted vs actual error response and log line.
- **FR-014**: Section 8 (Next steps) MUST point at two follow-on chains as separate future specs: (a) a typical authenticated GET CRUD chain (e.g. `GET /api/v1/sys-user`) which exposes Casbin and data-scope filtering, (b) a typical write-path CRUD which exposes operation log emission.
- **FR-015**: The document MUST identify the built-in Swagger UI route (`http://localhost:8000/swagger/admin/index.html`) as an alternative interactive endpoint explorer, but MUST NOT make Swagger UI the primary teaching surface — the curl path remains canonical.

#### Boundary discipline

- **FR-016**: The learning document MUST NOT modify, replace, or instruct the learner to modify any existing source file. It is read-only with respect to `app/`, `common/`, `cmd/`, `config/`, `main.go`, and the existing Docker artifacts. The only new files this spec authorizes are: `docker-compose.learning.yml` and `Dockerfile.learning` at the repository root, the document itself under `docs/learning/`, and the SpecKit artifacts under `specs/001-learning-trace-login/` (spec.md, plan.md, tasks.md, etc.).

### Key Entities

This feature does not introduce new domain entities; it documents existing ones. References are nonetheless cataloged so the learning doc can cite them precisely.

- **`sys_user`**: existing GORM model representing an admin user. Login query reads `username`, `status`, `password` (hash), `role_id`. Path: `app/admin/models/sys_user.go` (referenced — do not modify).
- **`sys_role`**: existing GORM model representing a role. Login query reads `role_id`, `role_key`, `data_scope`, `role_name`. Path: `app/admin/models/sys_role.go` (referenced — do not modify).
- **`sys_login_log`**: existing GORM model recording each login attempt. Verification reads new rows after a login. Path: `app/admin/models/sys_login_log.go` (referenced — do not modify).
- **JWT claims (in-memory only)**: keys `identity`, `roleid`, `rolekey`, `nice`, `datascope`, `rolename` constructed by `PayloadFunc` (`common/middleware/handler/auth.go`).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A learner unfamiliar with go-admin who has Docker installed but no Go toolchain can complete US1 in ≤30 minutes, including the first cold Docker build (network-dependent).
- **SC-002**: After reading the learning document once, a learner can correctly cite a `file:line` for ≥80% of the layers named in FR-008 without re-reading.
- **SC-003**: In US3 verification, all substantive comparisons (SQL × 2, JWT claims, `sys_login_log` row) match the learner's predictions; only cosmetic differences (timestamps, whitespace, column ordering) are present.
- **SC-004**: The final `docs/learning/login-trace.md` is between 600 and 1000 lines — comprehensive enough to teach, terse enough not to bloat.
- **SC-005**: Once `docker compose -f docker-compose.learning.yml up --build` has finished its first build, a learner can obtain a working JWT via the documented curl in ≤5 minutes (i.e. the dev-mode captcha bypass and the credential pair are clearly enough documented).
- **SC-006**: The work introduced by this spec changes zero lines in `app/`, `common/`, `cmd/`, `config/`, `main.go`, the existing `docker-compose.yml`, and the existing `Dockerfile`. Verified by `git diff --stat` against `main` showing only additions under `docker-compose.learning.yml`, `Dockerfile.learning`, `docs/learning/`, and `specs/001-learning-trace-login/`.

## Assumptions

- The learner has Docker (Docker Desktop or Docker Engine + the `compose` plugin) installed and runnable. Other tooling (Go, MySQL client, Node) is **not** assumed.
- The learner accepts a 5–10 minute first-build cost for the multi-stage Dockerfile in exchange for not needing Go locally.
- The default seeded credentials `admin/123456` work against the bundled `go-admin-db.db`. If they do not (e.g. the SQLite file has been wiped), the document's edge-case section provides recovery guidance rather than re-seeding being part of MVP.
- The two SDK boundaries (gin-jwt `LoginHandler`, JWT `TokenGenerator`) are intentionally treated as opaque in this spec. A learner who wants to trace deeper opens a separate spec scoped to the SDK repo (`fork260506-go-admin-core/`).
- Constitution Principle V (Observability & Audit Logs) governs the login path: a successful login MUST emit a `sys_login_log` row. This spec verifies that property; it does not introduce or relocate the logging code.
- The existing fork is on Go 1.24 (per `go.mod`); the Dockerfile builder stage matches.
