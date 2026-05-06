# Phase 0 Research — `001-learning-trace-login`

This file resolves every "needs investigation" item the spec implicitly raises.
Each section follows the **Decision / Rationale / Alternatives considered**
shape mandated by the plan template. Findings here become the source of truth
that `/speckit-tasks` consumes.

---

## R1 — Login chain layer catalogue (file:line index)

**Decision**: The learning document MUST present the login chain in this exact
order, citing these exact file paths. Line numbers cited below are
authoritative as of commit `85b91b6` on branch `001-learning-trace-login`; the
implementer MUST re-verify each before pasting into `docs/learning/login-trace.md`
because line drift will happen if upstream rebases.

| # | Layer | Path | Symbol | Note |
|---|-------|------|--------|------|
| 1 | Route registration | `app/admin/router/sys_router.go:71` | `v1.POST("/login", authMiddleware.LoginHandler)` | Inside `sysCheckRoleRouterInit`, but `/login` is registered on the v1 group BEFORE the `MiddlewareFunc()` wrap on `v1auth`, so login itself is unauthenticated — this is itself worth pointing out. |
| 2 | gin-jwt entry | `fork260506-go-admin-core/sdk/pkg/jwtauth/...` (sibling repo) | `LoginHandler` (SDK) | **SDK boundary** — call FR-010. Document with one sentence + a pointer; do not trace into the SDK. |
| 3 | Authenticator | `common/middleware/handler/auth.go:~60–105` | `func authenticator(c *gin.Context) (interface{}, error)` (or named per repo — implementer to confirm exact symbol) | Reads request body into `Login`, calls `captcha.Verify`, calls `Login.GetUser`, calls `LoginLogToDB` on both success and failure paths. |
| 4 | Captcha verify | `common/middleware/handler/auth.go:88` | `captcha.Verify(loginVals.UUID, loginVals.Code, true)` | The `true` flag = **clear-after-verify** (one-time use). Dev-mode bypass is documented in the swagger comment block at `auth.go:51–66`. |
| 5 | Login struct + GORM user lookup | `common/middleware/handler/login.go:9–17` (struct), `:17–39` (`GetUser` method) | `Login{Username, Password, Code, UUID}` + `GetUser(tx *gorm.DB)` | First query: `SELECT * FROM sys_user WHERE username = ? AND status = '2'`. Status `'2'` means *active* (NOT a typo for boolean). |
| 6 | Bcrypt password compare | `common/middleware/handler/login.go:24` | `pkg.CompareHashAndPassword(user.Password, u.Password)` | bcrypt internals are out of scope (see FR-016 + spec Out-of-scope). |
| 7 | GORM role lookup | `common/middleware/handler/login.go:30` | `tx.Table("sys_role").Where("role_id = ?", user.RoleId).First(&role)` | Second query: `SELECT * FROM sys_role WHERE role_id = ?`. |
| 8 | PayloadFunc (build claims) | `common/middleware/handler/auth.go:21–37` | `PayloadFunc(data interface{}) jwt.MapClaims` | Claims: `identity`, `roleid`, `rolekey`, `nice`, `datascope`, `rolename`. `IdentityKey` etc. are constants from gin-jwt SDK. |
| 9 | JWT TokenGenerator | `fork260506-go-admin-core/sdk/pkg/jwtauth/...` (sibling repo) | gin-jwt internal | **SDK boundary** — same treatment as #2. |
| 10 | LoginLog emission | `common/middleware/handler/auth.go:77` (queue push) → `cmd/api/server.go:70` (queue consumer registration) → `app/admin/models/sys_login_log.go:SaveLoginLog` (actual DB write) | `LoginLogToDB(c, status, msg, username)` | **Critical detail: this is async via the project's queue (`sdk.Runtime.GetStreamMessage`)**. The user-facing response can return BEFORE the row hits SQLite. The learning doc MUST disclose this so US3-3 verification is not confused by "I got the JWT but the row hasn't appeared yet" race conditions. Mitigation: the doc instructs `sleep 1` (or simply re-query) before checking `sys_login_log`. |
| 11 | Response assembly | `common/middleware/handler/auth.go:155` (success) / `:178` (alt) | `c.JSON(http.StatusOK, gin.H{...})` inside `LoginResponse` and adjacent handlers | Response shape: `{code: 200, data: {token, expire}, ...}` — exact shape implementer to confirm by reading the function bodies. |

**Rationale**: Codifying the layer catalogue in research.md (rather than only
in the eventual `docs/learning/login-trace.md`) gives `/speckit-tasks` a stable
list to generate "read file X:Y, paste into doc section Z" tasks in
deterministic order, and lets the implementer validate the chain against the
real codebase before prose is written.

**Alternatives considered**:
- *Embed all layer details directly in the learning doc and skip research.md*:
  rejected — research.md is the single source of truth that survives doc
  rewrites; pinning layer mappings here means the prose can be regenerated
  without re-discovering the chain.
- *Trace into the SDK for layers 2 / 9 / 11*: rejected by clarify-pass FR-010.
  Out of scope; learners who want SDK depth open a separate spec scoped to
  the sibling repo.

---

## R2 — LoginLog write path is asynchronous via the project queue

**Decision**: The learning document MUST explicitly call out that
`LoginLogToDB` (`auth.go:77`, `:109–...`) does **not** execute a `gorm.DB.Create`
inline. It serializes the log row into a queue message via
`sdk.Runtime.GetStreamMessage("", global.LoginLog, l)` and the queue consumer
`models.SaveLoginLog` (registered at `cmd/api/server.go:70`) performs the
actual INSERT later. The doc MUST instruct learners to (a) wait briefly or
(b) re-poll `sys_login_log` rather than expect the row to be present
synchronously with the login response.

**Rationale**: Without this disclosure, a learner running US3-3 verification
will get a flake — login responds 200, table is empty for ~50–500ms, then a
row appears. They will conclude the verification is broken when in fact the
queue is doing exactly what it should. This wrinkle is invisible if you only
read `auth.go:77` and assume `LoginLogToDB` is synchronous.

**Alternatives considered**:
- *Switch the consumer from queue to inline write for "learning simplicity"*:
  forbidden — that would modify production code, violating FR-016.
- *Hide this detail and let the learner be surprised*: rejected — Constitution
  Principle V demands the log be "load-bearing"; the delivery mechanism
  matters for verification correctness.

---

## R3 — `enableddb: true` toggle behaviour

**Decision**: Setting `settings.logger.enableddb: true` in the active
`settings.yml` causes go-admin (via go-admin-core's logger setup) to wire a
GORM logger that writes every executed SQL statement to the configured
log sink. With `settings.logger.stdout: ''` (empty string) and `mode: dev`,
that sink is the container's **stdout**, observable via `docker compose logs`.

**Rationale**: This was the user's Q2 selection (zero code change, zero proxy,
zero new tooling) and matches FR-016. The flag already ships set to `false`
in `config/settings.sqlite.yml` and `config/settings.yml`; flipping to `true`
is a one-line edit baked into `Dockerfile.learning`'s baked config copy.

**Alternatives considered**:
- *Leave `enableddb: false` and instruct learners to attach a SQL proxy*:
  rejected — extra dependency, extra complexity for marginal gain.
- *Keep `enableddb: false` and rely on `enabledp: true` (the data-permission
  flag)*: rejected — `enabledp` is unrelated; it's the data-scope feature
  toggle, not SQL logging.

**Implementation note for `/speckit-implement`**: The Dockerfile.learning
COPY step that places the SQLite config at `/config/settings.yml` MUST either
(a) `COPY` `settings.sqlite.yml` and then `RUN sed -i 's/enableddb: false/enableddb: true/' /config/settings.yml`, or (b) ship a small overlay file
`config/settings.learning.yml` that the Dockerfile copies instead. Option (a)
keeps the new file count minimal; option (b) keeps the change explicit. The
implementer should pick whichever they find clearer; both satisfy FR-012(a).

---

## R4 — Captcha bypass: dev-mode "0 / 0" sentinel

**Decision**: In dev mode (`settings.application.mode: dev`), the captcha
verification at `auth.go:88` accepts `code: "0"` and `uuid: "0"` as a
sentinel that bypasses the real captcha check. This is documented in the
swagger comment block immediately above the `Authenticator` function
(`common/middleware/handler/auth.go:51–66`):

> dev mode：It should be noted that all fields cannot be empty, and a value of 0 can be passed in addition to the account password
> 注意：开发模式：需要注意全部字段不能为空，账号密码外可以传入0值

The learning doc cites that exact comment so the bypass is authoritative
(documented in source) rather than a clever discovery.

**Rationale**: Without the bypass, the curl path requires fetching
`GET /api/v1/captcha`, decoding the base64 PNG, OCR'ing or eyeballing the
digit string, and round-tripping the UUID. Possible, but pedagogically noisy.
The bypass is intentional and explicitly designed for dev/test workflows.

**Alternatives considered**:
- *Run a 2-step curl flow — fetch captcha first, then login*: rejected for
  MVP. May appear later as a "want to learn the captcha endpoint too?"
  side-quest in the doc's "Next steps" section.
- *Run with `mode: prod` and document a non-bypass path*: rejected —
  contradicts the spec's dev-mode assumption and would force config gymnastics.

---

## R5 — Multi-stage Dockerfile.learning structure

**Decision**: Dockerfile.learning will follow this exact two-stage layout:

```dockerfile
# ---------- builder ----------
FROM golang:1.24-alpine AS builder
RUN apk add --no-cache gcc g++ libc6-compat
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=1 go build -tags sqlite3 -ldflags="-w -s" -o /out/go-admin .

# ---------- runtime ----------
FROM alpine:3
RUN apk add --no-cache ca-certificates tzdata libc6-compat
ENV TZ=Asia/Shanghai
COPY --from=builder /out/go-admin /go-admin
COPY config/settings.sqlite.yml /config/settings.yml
COPY go-admin-db.db /go-admin-db.db
EXPOSE 8000
CMD ["/go-admin", "server", "-c", "/config/settings.yml"]
```

with the `enableddb: true` toggle applied per R3.

**Rationale**:
- Builder stage matches `go.mod` (`go 1.24`) and the existing
  `make build-sqlite` target's flag set (`CGO_ENABLED=1`, `-tags sqlite3`,
  `-ldflags="-w -s"`).
- Runtime stage mirrors the existing `Dockerfile`'s package list
  (`ca-certificates`, `tzdata`, `libc6-compat`, `TZ=Asia/Shanghai`) — proven to
  work for this project. We deliberately keep parity to reduce surprise.
- `go mod download` runs after copying just `go.mod` + `go.sum` so the
  module download layer stays cached across edits to other files.
- Two `COPY` statements after the build stage place the config and DB
  exactly where the existing `Dockerfile`'s comment ("/config/settings.yml")
  expects them, keeping the runtime entrypoint identical.

**Alternatives considered**:
- *Distroless runtime base (`gcr.io/distroless/static`)*: rejected — incompatible
  with CGO `mattn/go-sqlite3`. distroless/cc would work but adds an unfamiliar
  base; alpine matches the existing repo flavor.
- *Single-stage build (run `go build` directly in the runtime stage)*:
  rejected — bloats the runtime image with the full Go toolchain (~600 MB vs
  ~30 MB for the two-stage approach).
- *Buildx + multi-arch*: out of scope for a learning artifact targeting one
  developer machine.

---

## R6 — Compose file structure

**Decision**: `docker-compose.learning.yml` ships with this structure:

```yaml
services:
  go-admin-learning:
    build:
      context: .
      dockerfile: Dockerfile.learning
    container_name: go-admin-learning
    ports:
      - "8000:8000"
    volumes:
      - ./temp:/temp     # log path bind-mount per FR-001
    # --- OPT-IN: persist the SQLite database across `down`/`up --build` cycles ---
    # Uncomment the lines below if you want sys_login_log and other rows to survive
    # container recreates. Default behaviour (commented) gives a clean DB each run,
    # which is what the US3-3 verification recipe assumes.
    #  - ./go-admin-db.db:/go-admin-db.db
```

Notes:
- The `version:` key is omitted intentionally — it's deprecated in Compose v2.
- No `networks:` block — Docker provides a default bridge network adequate
  for single-service compose.
- No `restart:` policy — learning context, not production; failure should
  surface to the learner immediately.

**Rationale**: Minimum surface area that satisfies FR-001, FR-003, FR-004
(coexistence with existing `docker-compose.yml`), and Q3 default (no DB volume,
opt-in instructions inline).

**Alternatives considered**:
- *Add a MySQL service to the compose for fidelity to "real" deployment*:
  rejected by Q3-default. The clarify pass locked SQLite-only.
- *Use `image: go-admin:latest` (the existing flow)*: rejected — defeats
  the goal of "host needs zero Go".
- *Add `healthcheck:`*: rejected for v1 — adds learner-cognitive cost; can
  be revisited if the doc grows a "production-ish hardening" appendix.

---

## R7 — Learning document outline and length budget

**Decision**: `docs/learning/login-trace.md` ships with these eight top-level
sections (matches FR-005), with target line counts that sum to 600–1000 lines
(SC-004):

| Section | Target lines | Source |
|---------|-------------:|--------|
| 1. Goal & Prerequisites | 30 | spec §Assumptions, §Edge Cases |
| 2. Bring-up via Docker Compose | 80 | FR-006, R5, R6 |
| 3. Login curl example with response | 60 | FR-007, R4 |
| 4. Sequence diagram of the login chain | 50 | FR-008 + R1 |
| 5. Layer-by-layer trace (10 sub-sections) | 350 | FR-009 + R1, R2 |
| 6. Runtime verification recipes | 150 | FR-012, FR-013, R3 |
| 7. Known SDK boundaries | 30 | FR-010, R1 |
| 8. Next steps | 30 | FR-014 |
| **Subtotal** | **780** | (within 600–1000) |

**Rationale**: Section 5 (the layer-by-layer trace) is the largest because
it is the actual learning payload — every layer needs ~35 lines (paragraph +
file:line + ≤15-line excerpt + brief commentary). Sections 2 and 6 (bring-up
and verification) are the runnable parts and need enough copy-paste density
to be self-sufficient. Sections 1, 7, 8 stay tight.

**Alternatives considered**:
- *Skip the sequence diagram*: rejected by FR-008. The diagram is also the
  fastest reference for someone re-reading later.
- *Inline the SDK boundary discussion into Section 5 layer notes*: rejected
  — making it a separate Section 7 keeps the boundary discipline visible.

---

## R8 — Acceptance verification protocol

**Decision**: The implementer (in `/speckit-implement`) MUST run the following
end-to-end check before marking the feature complete:

1. From the feature branch tip, on a clean worktree:
   ```bash
   docker compose -f docker-compose.learning.yml up --build -d
   sleep 60   # allow first build (network-bound)
   curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8000/swagger/admin/index.html
   # MUST print 200
   curl -s -X POST http://localhost:8000/api/v1/login \
     -H "Content-Type: application/json" \
     -d '{"username":"admin","password":"123456","code":"0","uuid":"0"}' \
     | tee /tmp/login.json
   # MUST contain a non-empty "token" field
   docker compose -f docker-compose.learning.yml logs go-admin-learning \
     | grep -E 'SELECT.*sys_user|SELECT.*sys_role' | wc -l
   # MUST be ≥ 2 (the two GetUser queries)
   ```

2. `git diff --stat main..HEAD` MUST show changes only under
   `specs/001-learning-trace-login/`, `docs/learning/`,
   `docker-compose.learning.yml`, and `Dockerfile.learning` — proves SC-006.

3. `wc -l docs/learning/login-trace.md` MUST be 600 ≤ N ≤ 1000 — proves
   SC-004.

**Rationale**: Captures the spec's quantifiable SCs as automatable shell
commands. `/speckit-implement` will turn each into a task with these exact
exit conditions.

**Alternatives considered**:
- *Skip end-to-end docker run during implement and rely on dry static checks*:
  rejected — defeats the spec's "edge case: build cache cold start" guarantee
  and "captcha bypass works" claim. Run-once is non-negotiable.
- *Run the full US3 verification suite (SQL × 2 prediction match, JWT decode
  match, login log row check) during implement*: deferred to learner's
  own use; the implementer only proves the *infrastructure* is correct, not
  that every verification recipe matches the predictions (that's a learner
  task).
