# Contract: `POST /api/v1/login`

> **This contract documents existing behaviour for learning purposes. It is
> NOT a new contract — the endpoint already exists and is unchanged by this
> feature (FR-016).** Source of truth: `app/admin/router/sys_router.go:71` +
> `common/middleware/handler/auth.go` + `common/middleware/handler/login.go`.

## Endpoint

- **Method**: `POST`
- **Path**: `/api/v1/login`
- **Auth required**: No (this IS the auth issuance endpoint)
- **Casbin enforcement**: No (registered on `v1` group BEFORE the
  `MiddlewareFunc()` wrap on `v1auth`)

## Request

### Headers

| Header | Required | Value |
|--------|----------|-------|
| `Content-Type` | yes | `application/json` |

### Body schema

JSON object. All four fields are `binding:"required"` — gin returns `400`
before the authenticator runs if any field is missing or empty.

```json
{
  "username": "string (required, non-empty)",
  "password": "string (required, non-empty)",
  "code":     "string (required, non-empty; '0' bypasses captcha in dev mode)",
  "uuid":     "string (required, non-empty; '0' bypasses captcha in dev mode)"
}
```

| Field | Source struct field | Notes |
|-------|---------------------|-------|
| `username` | `Login.Username` (`form:"UserName"`) | Looked up against `sys_user` with `status = '2'` |
| `password` | `Login.Password` (`form:"Password"`) | Bcrypt-compared against `sys_user.password` |
| `code` | `Login.Code` (`form:"Code"`) | The digit string from the captcha image (or `"0"` in dev) |
| `uuid` | `Login.UUID` (`form:"UUID"`) | The captcha id returned by `GET /api/v1/captcha` (or `"0"` in dev) |

### Dev-mode captcha bypass

When `settings.application.mode: dev`, both `code: "0"` and `uuid: "0"`
together cause `captcha.Verify` to return success without any real captcha
state. This is documented in the swagger comment block at
`common/middleware/handler/auth.go:51–66`. The bypass only activates when
**both** sentinel values are present; mixing one real value with one `"0"`
fails normally.

## Response — success (HTTP 200)

```json
{
  "code": 200,
  "expire": "2026-05-07T16:00:00Z",
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

(Exact wrapper field names — `code` / `expire` / `token` — are produced by
`common/middleware/handler/auth.go:155` `c.JSON(http.StatusOK, gin.H{...})`.
Implementer to confirm exact field names by reading the function body when
producing the learning doc.)

### JWT claims inside `token`

Decode the second segment of the JWT (base64url) to see:

```json
{
  "identity":  <SysUser.UserId>,
  "roleid":    <SysRole.RoleId>,
  "rolekey":   "<SysRole.RoleKey>",
  "nice":      "<SysUser.Username>",
  "datascope": "<SysRole.DataScope>",
  "rolename":  "<SysRole.RoleName>",
  "exp":       <unix_seconds>,
  "orig_iat":  <unix_seconds>
}
```

Signed HS256 with `settings.jwt.secret` (default `"go-admin"`).

## Response — failure modes

| HTTP | Trigger | Response shape (informally) |
|------|---------|------------------------------|
| 400 | Any of the four required fields missing or empty | gin binding error JSON |
| 401 | Captcha invalid (and not bypassed) | `{"code": 401, "msg": "验证码错误"}` (exact text from auth.go) |
| 401 | Username not found OR `status != '2'` | Generic auth failure — does not leak whether user exists |
| 401 | Password mismatch (bcrypt fails) | Generic auth failure |
| 401 | Role lookup fails (rare — implies seed inconsistency) | Auth failure |
| 500 | DB unreachable / unexpected error | Generic 500 |

The learning doc's negative-path recipe (FR-013) MUST exercise at least one
of: wrong password, missing captcha field (forces 400 from binding), or
expired-token-on-protected-endpoint.

## Side effects (per Constitution Principle V)

Every call — success or failure — produces a `sys_login_log` row via the
**asynchronous** queue:

1. Authenticator calls `LoginLogToDB(c, status, msg, username)`
   (`auth.go:77` for failure path; equivalent call on success path).
2. `LoginLogToDB` (`auth.go:109–...`) builds a `SysLoginLog` value and pushes
   it via `sdk.Runtime.GetStreamMessage("", global.LoginLog, l)`.
3. The queue consumer `models.SaveLoginLog` (registered at
   `cmd/api/server.go:70`) executes the actual `gorm.DB.Create`.

**Timing**: The HTTP response can return before the queue consumer has run.
Verifiers must allow ~50–500ms of skew, or re-query.

## Related contracts

- [`get-captcha.md`](./get-captcha.md) — the captcha-fetch endpoint, useful
  if the learner wants to exercise the non-bypass path.
