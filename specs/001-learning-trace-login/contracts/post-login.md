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

> **T012/T017 finding**: this contract was originally drafted at planning time
> with a 3-key shape. The actual response has **5 top-level keys**. The shape
> is produced by gin-jwt SDK's default `LoginResponse` callback (the fork
> does not override it), NOT by `auth.go:155` (which is the unrelated `LogOut`
> handler). See `docs/learning/login-trace.md` §3.2 + §5.11 for the full
> trace.

```json
{
  "code":             200,
  "currentAuthority": "eyJhbGciOiJIUzI1NiIs...",
  "expire":           "2126-04-13T12:08:37+08:00",
  "success":          true,
  "token":            "eyJhbGciOiJIUzI1NiIs..."
}
```

`currentAuthority` and `token` carry the same JWT — `currentAuthority` exists
for antd-pro front-end compatibility. Use `token` for `Authorization: Bearer
...` headers.

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

> **T017 finding**: go-admin / gin-jwt wrap auth failures in **HTTP 200**
> with a body `{code, msg}` that carries the actual numeric error code in the
> body's `code` field (NOT the HTTP status). The failure-mode `msg` strings
> in the HTTP response come from gin-jwt SDK's English defaults (e.g.
> `"incorrect Username or Password"`); the in-process Simplified Chinese
> strings from `auth.go` (`"登录失败"`, `"数据解析失败"`, `"验证码错误"`)
> appear ONLY in `sys_login_log` rows, not in HTTP responses.

| HTTP | Body `code` | Trigger | Body `msg` (HTTP) | sys_login_log `msg` |
|------|-------------|---------|--------------------|---------------------|
| 400 | — | Any of the four required fields missing/empty | gin binding error | (no row written: failure pre-Authenticator) |
| 200 | 400 | Captcha invalid (non-bypass path) | (SDK string) | `"验证码错误"` |
| 200 | 400 | Username not found, password mismatch, or role lookup fails | `"incorrect Username or Password"` | `"登录失败"` |
| 200 | 400 | JSON binding fails inside `Authenticator` | (SDK string) | `"数据解析失败"` |
| 500 | — | DB unreachable / unexpected error | Generic 500 | (no row written) |

For the failure paths that do produce a `sys_login_log` row, the row also
carries `status='1'` (vs `'2'` for success) and `username=''` (empty —
`Authenticator`'s default-then-conditional-assign pattern leaves it empty
on most failure branches; see `docs/learning/login-trace.md` §6.4 for the
trace).

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
