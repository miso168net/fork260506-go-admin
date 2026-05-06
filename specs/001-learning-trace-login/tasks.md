---
description: "Task list for 001-learning-trace-login"
---

# Tasks: Learning Walkthrough — Trace `POST /api/v1/login` End-to-End

**Input**: Design documents from `specs/001-learning-trace-login/`
**Prerequisites**: plan.md (required), spec.md (required), research.md, data-model.md, contracts/, quickstart.md (all present)

**Tests**: Per Constitution Principle III, tests would be MANDATORY for any new auth/Casbin/JWT/persistence code path. **This feature adds zero such code** — it ships only Docker artifacts and a learning document. Therefore no test tasks are generated. The spec's FR-012 / FR-013 verification recipes are *manual acceptance* steps captured as tasks below; they verify *existing* production behaviour, not new logic.

**Organization**: Tasks are grouped by user story so each story can be implemented and validated independently. Within each user-story phase, tasks are sequential because they edit shared files (`docs/learning/login-trace.md`); the only parallelizable point is the Foundational phase, where Dockerfile and compose live in different files.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies on incomplete tasks)
- **[Story]**: Which user story this task belongs to (US1, US2, US3); omitted for Setup, Foundational, and Polish phases
- File paths are exact and absolute-from-repo-root

## Path Conventions

This feature does not introduce a `src/`+`tests/` source tree. Deliverables are:
- Repo-root container artifacts: `docker-compose.learning.yml`, `Dockerfile.learning`
- Documentation: `docs/learning/login-trace.md`
- SpecKit artifacts: `specs/001-learning-trace-login/` (already populated)

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Create the directory the learning document will live in.

- [ ] T001 Create `docs/learning/` directory at the repository root (`mkdir -p docs/learning`); confirm `docs/` does not already contain a `learning/` subdirectory before running so prior work is not clobbered.

**Checkpoint**: `docs/learning/` exists; ready for any phase that writes the learning document.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Produce the runnable container artifacts that every later user story needs in order to verify acceptance against a live server.

**⚠️ CRITICAL**: No user-story phase can be acceptance-tested until Foundational is complete. The artifacts below also satisfy spec FR-001 through FR-004.

- [ ] T002 [P] Write `Dockerfile.learning` at the repository root following the multi-stage layout codified in `specs/001-learning-trace-login/research.md` §R5: builder = `golang:1.24-alpine` with `gcc g++ libc6-compat`, runs `CGO_ENABLED=1 go build -tags sqlite3 -ldflags="-w -s" -o /out/go-admin .`; runtime = `alpine` with `ca-certificates tzdata libc6-compat`, `ENV TZ=Asia/Shanghai`, `COPY` the built binary, `COPY config/settings.sqlite.yml /config/settings.yml`, `COPY go-admin-db.db /go-admin-db.db`, `EXPOSE 8000`, `CMD ["/go-admin","server","-c","/config/settings.yml"]`. Apply the `enableddb: true` toggle per research.md §R3 (either via `RUN sed -i 's/enableddb: false/enableddb: true/' /config/settings.yml` after the `COPY`, or by shipping a separate `config/settings.learning.yml` overlay — implementer's choice; both satisfy FR-012(a)).
- [ ] T003 [P] Write `docker-compose.learning.yml` at the repository root following research.md §R6: single service `go-admin-learning` that `build`s from `Dockerfile.learning`, exposes `8000:8000`, bind-mounts `./temp:/temp` for log writes, and includes a *commented-out* volume block (`# - ./go-admin-db.db:/go-admin-db.db`) with a one-line comment explaining when to uncomment per Q3 clarification. Do **not** add a `version:` key; do **not** add `restart:`; do **not** add a MySQL service (Q3 locked SQLite-only).
- [ ] T004 Build verification — from a clean docker state (`docker rmi go-admin-learning 2>/dev/null || true`), run `docker compose -f docker-compose.learning.yml up --build -d` and tail logs (`docker compose -f docker-compose.learning.yml logs -f go-admin-learning`); confirm a line like `listening and serving HTTP on :8000` appears within 10 minutes (first build, network-bound). Then `docker compose -f docker-compose.learning.yml down` and ensure the container is gone. This is the gate that proves T002+T003 produced a runnable system.

**Checkpoint**: A clean `docker compose -f docker-compose.learning.yml up --build` produces a server listening on port 8000. All later acceptance tasks can now run.

---

## Phase 3: User Story 1 — Bring backend up + obtain JWT via curl (Priority: P1) 🎯 MVP

**Goal**: A learner with only Docker installed can run one command and obtain a valid JWT via curl, no frontend, no Go.

**Independent Test**: On a clean host (Docker only), execute `docker compose -f docker-compose.learning.yml up --build`, then the documented curl, then verify the response contains a non-empty `token`. Use that token against `GET /api/v1/getinfo`; HTTP 200 = pass.

### Implementation for User Story 1

- [ ] T005 [US1] Create the learning document skeleton at `docs/learning/login-trace.md`: title in zh-TW, the eight top-level section headers from FR-005 (Goal & Prerequisites; Bring-up via Docker Compose; Login curl example with response; Sequence diagram of the login chain; Layer-by-layer trace; Runtime verification recipes; Known SDK boundaries; Next steps), a brief table of contents linking to each, and a one-line note that the document was generated from `specs/001-learning-trace-login/`.
- [ ] T006 [US1] Write Section 1 "Goal & Prerequisites" in `docs/learning/login-trace.md` (target ≈30 lines) — zh-TW prose distilled from spec.md US1 + Assumptions: what the learner will be able to do after this doc, the Docker-only prerequisite, the explicit non-prerequisites (Go, MySQL, `go-admin-ui` frontend), and the time budget (≤30 min total, ≤5 min after first build). Per FR-017, code/config snippets and quoted source comments stay verbatim.
- [ ] T007 [US1] Write Section 2 "Bring-up via Docker Compose" in `docs/learning/login-trace.md` (target ≈80 lines) — copy-paste-ready `docker compose -f docker-compose.learning.yml up --build` invocation, the expected first-run vs cached-run wall-clock budget, a sanity-check curl against `/swagger/admin/index.html` (HTTP 200), and the FR-015 callout that Swagger UI is the built-in interactive explorer (alternative to curl, but not the canonical learning surface). Cite `app/admin/router/sys_router.go:59` for where Swagger UI is registered.
- [ ] T008 [US1] Write Section 3 "Login curl example with response" in `docs/learning/login-trace.md` (target ≈60 lines) — the canonical curl using `code:"0", uuid:"0"` per FR-007, the response shape with `code` / `expire` / `token` keys (cite `contracts/post-login.md`), a one-line note on how to base64-decode the JWT to peek at claims, and a verbatim quote of the swagger comment block at `common/middleware/handler/auth.go:51–66` that authorizes the dev-mode bypass.
- [ ] T009 [US1] MVP acceptance — manually execute the curl from Section 3 against the container brought up by T004, paste the returned token into a `Authorization: Bearer …` header, and call `GET http://localhost:8000/api/v1/getinfo`; record HTTP status and the wall-clock time from `up --build` start to the protected endpoint returning 200. SC-001 passes if the wall-clock is ≤30 minutes; SC-005 passes if the time-from-built-image to JWT is ≤5 minutes.

**Checkpoint**: At this point the MVP is shippable: a learner can bring up the system and reach a protected endpoint. Sections 4–8 are still placeholders.

---

## Phase 4: User Story 2 — Trace the login request through every layer of code (Priority: P2)

**Goal**: After reading Sections 4 + 5 + 7 + 8 once, the learner can name every layer of the login chain in order and cite a `file:line` for ≥80% of them without re-reading.

**Independent Test**: Hand the doc to a target reader (yourself with fresh eyes, or a colleague unfamiliar with go-admin); ask them to (a) describe each layer's role in their own words, (b) point to the `file:line` for each layer, (c) reproduce the sequence-diagram arrows from memory at coarse granularity. SC-002 passes if ≥80% of `file:line` cites land correctly.

### Implementation for User Story 2

- [ ] T010 [US2] Re-verify the R1 layer-catalogue line numbers against the current branch tip — for each of the 11 entries in `specs/001-learning-trace-login/research.md` §R1, run `sed -n 'Np' <file>` (where N is the cited line) and confirm the cited symbol or substring still lives there. If any line drifted (e.g. someone edited `auth.go` between branch creation and now), update research.md §R1 with the correct line and explain the drift in a one-line note.
- [ ] T011 [US2] Write Section 4 "Sequence diagram of the login chain" in `docs/learning/login-trace.md` (target ≈50 lines) — a Mermaid `sequenceDiagram` block per FR-008 covering: `Client` → `Router` → `gin-jwt LoginHandler (SDK)` → `Authenticator` → `captcha.Verify` → `Login.GetUser`. Inside `GetUser`, depict **two distinct DB round-trips** (one to `sys_user`, one to `sys_role`) with `pkg.CompareHashAndPassword` between them. After `GetUser` returns: → `PayloadFunc` → `JWT TokenGenerator (SDK)` → `LoginLog hook` → `Response`. Mark the two SDK boundaries visually (note over participant or distinct color).
- [ ] T012 [US2] Write Section 5 "Layer-by-layer trace" in `docs/learning/login-trace.md` (target ≈350 lines, ≈35 per layer × 10 layers — the two SDK boundaries get short stubs) — for each of the 11 entries in research.md §R1: a zh-TW one-paragraph role explanation, the exact `file:line`, a code excerpt of ≤15 lines from that file, and a brief commentary tying the layer to the next. Layer #2 (gin-jwt LoginHandler) and Layer #9 (JWT TokenGenerator) MUST cite `fork260506-go-admin-core/sdk/pkg/jwtauth/` per FR-010 and explicitly stop without tracing into the SDK. Layer #10 (LoginLog) MUST disclose the async queue wrinkle from research.md §R2 — the HTTP response can return before the row is persisted.
- [ ] T013 [US2] Write Section 7 "Known SDK boundaries" in `docs/learning/login-trace.md` (target ≈30 lines) — list the two boundaries (gin-jwt `LoginHandler`, JWT `TokenGenerator`), one paragraph each on what they conceptually do, why this spec stops at each (FR-010), and a pointer to the sibling repo `fork260506-go-admin-core/` for learners who want to dive deeper.
- [ ] T014 [US2] Write Section 8 "Next steps" in `docs/learning/login-trace.md` (target ≈30 lines) — point to two follow-on chains as future SpecKit specs per FR-014: (a) a typical authenticated GET CRUD chain (e.g. `GET /api/v1/sys-user`) to learn Casbin enforcement and data-scope filtering, (b) a typical write-path CRUD to learn operation-log emission. Note that opening either is `/speckit-specify` followed by the brainstorming flow.
- [ ] T015 [US2] US2 acceptance — read Sections 4 + 5 + 7 + 8 once with fresh eyes (close the file, take a 10-min break, then open it). For each `file:line` reference, click through (or use `code` / `vim` to jump) and confirm the citation lands on a meaningful line. Record the % of cites that landed correctly; SC-002 passes at ≥80%. Render the Mermaid diagram in a preview tool (VS Code Markdown preview or mermaid.live) and confirm it renders without syntax errors.

**Checkpoint**: At this point a reader who completed US1 can also understand *why* it works. Section 6 (verification) is still placeholder.

---

## Phase 5: User Story 3 — Verify understanding with runtime evidence (Priority: P3)

**Goal**: The learner converts code-derived predictions into running-system verifications: predicted SQL matches GORM's actual log lines, predicted JWT claims match the decoded token, predicted `sys_login_log` row appears.

**Independent Test**: Walk the learner through Section 6's three positive recipes plus one negative recipe; record predicted vs actual for each. SC-003 passes if all substantive comparisons match (cosmetic differences allowed).

### Implementation for User Story 3

- [ ] T016 [US3] Write Section 6 "Runtime verification recipes" in `docs/learning/login-trace.md` (target ≈150 lines) covering FR-012 (a, b, c) and at least one FR-013 negative-path recipe:
   - **(a) GORM SQL capture (FR-012a)**: confirm `enableddb: true` is baked into the runtime image (cite `Dockerfile.learning` line that sets it), provide `docker compose -f docker-compose.learning.yml logs go-admin-learning | grep -E 'SELECT|INSERT|UPDATE'`, list the *predicted* SQL for the two `GetUser` queries (`SELECT * FROM sys_user WHERE username = ? AND status = '2'` and `SELECT * FROM sys_role WHERE role_id = ?`), and instruct the learner to compare to the actual log output.
   - **(b) JWT claim decode (FR-012b)**: provide `echo "$TOKEN" | cut -d. -f2 | base64 -d` (note the URL-safe base64 caveat — pad with `==` if needed), and the predicted claim shape from `data-model.md` §E4. Encourage cross-referencing `PayloadFunc` at `auth.go:21–37`.
   - **(c) `sys_login_log` row inspection (FR-012c)**: a `docker compose -f docker-compose.learning.yml exec go-admin-learning sqlite3 /go-admin-db.db 'SELECT id, username, status, msg, login_time FROM sys_login_log ORDER BY id DESC LIMIT 1;'` recipe; **MUST disclose the async queue wrinkle** from research.md §R2 and instruct `sleep 1` before re-querying, OR re-poll until the row appears (max 3 retries). Cite that this verifies Constitution Principle V.
   - **(negative, FR-013)**: at least one negative path. Recommended: send a wrong password (`"password":"wrong"`), record the predicted 401 response and the predicted login-log row with `status='2'` and `msg='密码错误'` (or whatever the source string is — implementer to confirm by reading auth.go). Optional second negative: omit one of the four required fields and observe the gin binding 400.
- [ ] T017 [US3] US3 acceptance — manually run all four recipes from Section 6 against the container brought up by T004. For each recipe, fill in the actual output and compare to the predicted output. Cosmetic differences (timestamps, whitespace, column ordering) are allowed; substantive differences are not. SC-003 passes if all three positive-path comparisons match and the negative-path produces the predicted 4xx status. If any substantive mismatch is found, fix the prediction in the doc (the goal is teaching what the system actually does, not what the implementer wishes it did).

**Checkpoint**: All three user stories complete. Document is functionally finished; remaining tasks are polish and final acceptance.

---

## Phase N: Polish & Cross-Cutting Concerns

**Purpose**: Final sanity gates that the spec's quantitative SCs are met and the boundary discipline (FR-016 / SC-006) was honored.

- [ ] T018 Length budget check (SC-004) — run `wc -l docs/learning/login-trace.md`; if N < 600, expand the most-anemic section (likely Section 1 or 7) with concrete examples; if N > 1000, prune duplicative prose without removing any FR-mandated content. Iterate until 600 ≤ N ≤ 1000.
- [ ] T019 Boundary-discipline check (SC-006) — run `git diff --stat main..HEAD`; the only paths shown MUST be under `specs/001-learning-trace-login/`, `docs/learning/`, `docker-compose.learning.yml`, `Dockerfile.learning`, and (if updated) `CLAUDE.md`. Any `app/`, `common/`, `cmd/`, `config/`, `main.go`, `docker-compose.yml` (without the `.learning` suffix), or `Dockerfile` (without the `.learning` suffix) entry is a violation; revert that change and explain in a commit note.
- [ ] T020 End-to-end acceptance from research.md §R8 — execute the four-step shell protocol verbatim: (1) `docker compose -f docker-compose.learning.yml up --build -d`, (2) wait, (3) `curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8000/swagger/admin/index.html` MUST print `200`, (4) the documented login curl MUST return a non-empty `token`, (5) `docker compose -f docker-compose.learning.yml logs go-admin-learning | grep -E 'SELECT.*sys_user|SELECT.*sys_role' | wc -l` MUST be ≥ 2. All five MUST pass before this task is checked off.
- [ ] T021 Update `CLAUDE.md` SPECKIT block — change the line that currently says `tasks.md — created by /speckit-tasks (not yet present)` to reflect that `tasks.md` is now present and active. Optional but keeps the agent-context file accurate.
- [ ] T022 Run `graphify update .` from the repository root (per the project's CLAUDE.md graphify rule) so the knowledge graph picks up the three new files (`Dockerfile.learning`, `docker-compose.learning.yml`, `docs/learning/login-trace.md`) plus the SpecKit artifacts. AST-only update, no API cost.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; T001 can start immediately.
- **Foundational (Phase 2)**: Depends on Setup. T002 + T003 are parallelizable; T004 sequential after both.
- **User Story 1 (Phase 3)**: Depends on Foundational. T005 → T006 → T007 → T008 → T009 sequential within the phase (all touch the same file).
- **User Story 2 (Phase 4)**: Depends on Foundational; logically benefits from US1 being done (so the learner who reads has actually run the system) but is not technically blocked by US1's doc sections — T010 → T011 → T012 → T013 → T014 → T015 sequential within phase.
- **User Story 3 (Phase 5)**: Depends on Foundational + US2 (T010's verified file:line index). T016 → T017 sequential within phase.
- **Polish (Phase N)**: T018–T020 depend on all user-story phases being complete; T021–T022 are housekeeping that can run any time after T020.

### Within Each User Story

- All US phases edit the same file (`docs/learning/login-trace.md`), so [P] markers do not appear within a phase.
- The sequence within each US is: **skeleton/section drafting → per-layer or per-recipe filling → acceptance verification of that story's independent test**.

### Parallel Opportunities

- **Phase 2 only**: T002 (Dockerfile.learning) and T003 (docker-compose.learning.yml) are different files with no shared content; run them concurrently.
- All other phases are inherently sequential because they build up a single Markdown file.

---

## Parallel Example: Phase 2 (Foundational)

```bash
# Two independent contributors — or one contributor in two terminals — can do:
Task: "Write Dockerfile.learning at repo root per research.md §R5"     # T002
Task: "Write docker-compose.learning.yml at repo root per §R6"          # T003
# Then converge on T004 once both files exist.
```

---

## Implementation Strategy

### MVP First (User Story 1 only)

1. Complete Phase 1 (Setup) — `mkdir -p docs/learning`.
2. Complete Phase 2 (Foundational) — produce `Dockerfile.learning` + `docker-compose.learning.yml`, verify the build.
3. Complete Phase 3 (US1) — produce Sections 1–3 of the learning doc and run acceptance.
4. **STOP and VALIDATE**: Hand the doc + compose pair to someone unfamiliar with go-admin; ask them to run it. SC-001 + SC-005 + the parts of SC-006 about new files = MVP shipped.

At this point you have a usable artifact: a Docker-based bring-up with curl-based JWT acquisition, documented step-by-step. If you stop here, the work is still net-positive.

### Incremental Delivery

1. MVP (Phase 1 + 2 + 3) → ship.
2. Add Phase 4 (US2) → ship a learner who can also *understand* the chain.
3. Add Phase 5 (US3) → ship a learner who has *verified* the chain.
4. Phase N polish → SC-004 / SC-006 / R8 acceptance gates.

### Single-Contributor Strategy (this repo's reality)

Since this is a one-developer fork, the parallel-team strategy in the
template is moot. The realistic order is just the linear: Phase 1 →
Phase 2 (T002 + T003 in two terminals if you like; otherwise sequential)
→ Phase 3 → Phase 4 → Phase 5 → Phase N. Estimated wall-clock: ~4–6
hours of focused work spread across docs and Docker, plus ~10 minutes
of first-time Docker build.

---

## Notes

- [P] tasks = different files, no dependencies on incomplete tasks.
- [Story] label maps task to spec.md US1/US2/US3 for traceability.
- Each user-story phase is independently testable per its Independent Test criterion in spec.md.
- No "tests-must-fail-first" tasks because no production code is added (Constitution III is N/A).
- Verify the existing `docker-compose.yml` and `Dockerfile` are unchanged after every task touching the docker artifacts; this is the FR-016 / SC-006 invariant.
- Commit between logical groups: after T004 (foundational built), after T009 (MVP shipped), after T015 (US2 done), after T017 (US3 done), after T020 (final acceptance).
- Avoid: editing existing application source; introducing a MySQL/Redis sidecar (Q3 locked SQLite-only); tracing into `fork260506-go-admin-core/` from inside this doc (FR-010 SDK boundary).
