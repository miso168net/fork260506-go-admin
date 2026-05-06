# Quickstart — `001-learning-trace-login`

> **Audience**: A learner who has just cloned this fork, has Docker installed,
> does not necessarily have Go locally, and wants the **shortest possible
> path** from "clone" to "I have a JWT in hand."
>
> The fuller learning experience lives in `docs/learning/login-trace.md`
> (created during `/speckit-implement`). This quickstart is the 5-minute
> appetizer.

---

## Prerequisites

- Docker (Docker Desktop on Mac/Win, or Docker Engine + the `compose` plugin
  on Linux/WSL2)
- A terminal with `curl` available
- ~10 minutes for the first build (network-bound on Go base image + module
  download); ~30 seconds for subsequent runs

You do **not** need: Go toolchain, MySQL, Node, the `go-admin-ui` frontend.

---

## Step 1 — Bring the server up

From the repository root (the directory containing `go.mod` and
`go-admin-db.db`):

```bash
docker compose -f docker-compose.learning.yml up --build
```

> *First run downloads `golang:1.24-alpine` (~350 MB), `alpine`, and Go
> modules. Subsequent runs use the layer cache.*

Wait until you see a log line ending with something like
`listening and serving HTTP on :8000`.

Sanity check (in another terminal):

```bash
curl -s -o /dev/null -w "swagger UI: HTTP %{http_code}\n" \
  http://localhost:8000/swagger/admin/index.html
# Expected: swagger UI: HTTP 200
```

You can also open `http://localhost:8000/swagger/admin/index.html` in a
browser — the Swagger UI is the project's built-in interactive API explorer
and serves as a UI-shaped alternative to curl.

## Step 2 — Get a JWT via curl (dev-mode captcha bypass)

```bash
curl -s -X POST http://localhost:8000/api/v1/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"123456","code":"0","uuid":"0"}'
```

Expected response shape (token elided):

```json
{
  "code":   200,
  "expire": "2026-05-07T16:00:00Z",
  "token":  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...payload...signature"
}
```

The `code:"0", uuid:"0"` pair triggers the dev-mode captcha bypass — see the
swagger comment block at `common/middleware/handler/auth.go:51–66`. It only
works when `settings.application.mode: dev`, which is the default in
`docker-compose.learning.yml`'s baked config.

## Step 3 — Decode the JWT to see the claims

```bash
TOKEN="<paste the token from step 2>"
echo "$TOKEN" | cut -d. -f2 | base64 -d 2>/dev/null
```

(Or paste at https://jwt.io.)

Expected claims:

```json
{
  "identity":  1,
  "roleid":    1,
  "rolekey":   "admin",
  "nice":      "admin",
  "datascope": "1",
  "rolename":  "系统管理员",
  "exp":       1746636000,
  "orig_iat":  1746632400
}
```

Field meanings → see `data-model.md` §E4.

## Step 4 — Use the JWT against a protected endpoint

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  http://localhost:8000/api/v1/getinfo
```

Returns user/role/menu info for the authenticated admin. **HTTP 200 here
proves the entire chain — login → JWT → JWT middleware verifying the token —
is working end-to-end.**

## Step 5 — Tear down

```bash
docker compose -f docker-compose.learning.yml down
```

(With the default no-volume compose, this also clears the SQLite state. Next
`up --build` starts fresh.)

---

## Where to go next

- **Want to read the chain**: `docs/learning/login-trace.md` (produced by
  `/speckit-implement`).
- **Want to see GORM SQL**: `enableddb` is already `true` inside the
  container; check `docker compose logs go-admin-learning | grep -E
  'SELECT|INSERT'`.
- **Want to verify `sys_login_log`**: see `docs/learning/login-trace.md`
  §6 (verification recipes).
- **Want to learn the captcha endpoint instead of bypassing it**: see
  `contracts/get-captcha.md` for the 3-step non-bypass curl flow.
- **Want to trace a different chain** (`GET /api/v1/sys-user`, write
  paths, code generation): open a new SpecKit feature with
  `/speckit-specify`.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `connection refused` on port 8000 | Container still building, or build failed | `docker compose -f docker-compose.learning.yml logs` |
| Login returns 401 with no useful detail | Default credentials drifted, or DB has no admin user | Recreate the container (`down` + `up --build`) — image bundles a known-good `go-admin-db.db` |
| Login returns 400 with `"binding failed"` | One of the four required fields missing | All four (`username`, `password`, `code`, `uuid`) must be present and non-empty; for bypass both `code` and `uuid` must be literal `"0"` |
| `sys_login_log` row not yet visible | Async queue write — see `data-model.md` §E3 | `sleep 1` and re-query, or open a `docker compose exec` shell and run `sqlite3 /go-admin-db.db 'SELECT * FROM sys_login_log ORDER BY id DESC LIMIT 1;'` |
| Build fails on `mattn/go-sqlite3` with CGO errors | Missing builder packages | Verify `Dockerfile.learning` builder stage has `gcc g++ libc6-compat` |
