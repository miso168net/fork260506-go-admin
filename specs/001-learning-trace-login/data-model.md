# Phase 1 Data Model — `001-learning-trace-login`

This feature introduces **no new entities**. The learning document references
three existing GORM models and one in-memory JWT claim structure. This file
catalogues their shape so `/speckit-tasks` and `/speckit-implement` have a
single source of truth for what the learning doc should excerpt.

> **Mandatory boundary reminder**: All entities below are read-only with
> respect to this feature (FR-016). The learning doc may quote and explain;
> it MUST NOT modify any field, comment, tag, or struct.

---

## E1 — `SysUser` (existing GORM model)

**Path**: `app/admin/models/sys_user.go` (struct definition starts at line 9).
**TableName**: `sys_user`.
**Used by login chain at**: `common/middleware/handler/login.go:18`
(`tx.Table("sys_user").Where("username = ? and status = '2'", u.Username).First(&user)`).

**Fields read by the login path** (subset of the full struct):

| Field | Go type | DB column (gorm tag) | Login-path role |
|-------|---------|----------------------|-----------------|
| `UserId` | `int` | `primaryKey;autoIncrement;comment:编码` | Becomes `identity` claim in JWT |
| `Username` | `string` | `size:64;comment:用户名` | Where-clause input; becomes `nice` claim |
| `Password` | `string` | `size:128;comment:密码` (`json:"-"` so it never serializes outbound) | Bcrypt-compared against request password |
| `RoleId` | `int` | `size:20;comment:角色ID` | FK used to look up `SysRole` in step 7 of R1 |
| `Status` | `string` | `size:4;comment:状态` | Hard filter `status = '2'` (active) — non-active users fail login |

**Other fields** (`NickName`, `Phone`, `Avatar`, `Sex`, `Email`, `DeptId`,
`PostId`, `Remark`, `Salt`, `DeptIds`, `PostIds`, `RoleIds`, `Dept`) exist on
the struct but are not read or modified by the login chain. The learning doc
SHOULD acknowledge their existence in one sentence so a learner who scrolls
through the struct is not confused; it MUST NOT walk every field.

**Embedded helpers**: `models.ControlBy` and `models.ModelTime` (audit timestamps,
created_by/updated_by) — out of scope for this learning doc beyond a footnote.

---

## E2 — `SysRole` (existing GORM model)

**Path**: `app/admin/models/sys_role.go` (struct starts at line 5).
**TableName**: `sys_role`.
**Used by login chain at**: `common/middleware/handler/login.go:30`
(`tx.Table("sys_role").Where("role_id = ?", user.RoleId).First(&role)`).

**Fields read by the login path**:

| Field | Go type | Login-path role |
|-------|---------|-----------------|
| `RoleId` | `int` | Where-clause input (matches `SysUser.RoleId`); becomes `roleid` claim |
| `RoleKey` | `string` | Becomes `rolekey` claim — used downstream by Casbin enforcement on protected endpoints |
| `RoleName` | `string` | Becomes `rolename` claim |
| `DataScope` | `string` | Becomes `datascope` claim — used by data-scope filtering on later requests |

Other fields (`RoleSort`, `Flag`, `Remark`, `Admin`, `Params`, `MenuIds`,
`DeptIds`, `SysDept`, `SysMenu`) exist but are not part of the JWT payload
or the login decision.

---

## E3 — `SysLoginLog` (existing GORM model — emission target for Constitution Principle V)

**Path**: `app/admin/models/sys_login_log.go` (struct starts at line 15).
**TableName**: `sys_login_log`.
**Write entrypoint**: `common/middleware/handler/auth.go:77` — `LoginLogToDB(c, status, msg, username)`.
**Actual DB INSERT**: indirect via the project's queue: `auth.go:LoginLogToDB`
serializes a `SysLoginLog` instance and pushes through
`sdk.Runtime.GetStreamMessage("", global.LoginLog, l)`; the queue consumer
`models.SaveLoginLog` (registered at `cmd/api/server.go:70`) executes the
final `gorm.DB.Create`.

**Fields populated on every login attempt (success or failure)**:

| Field | Type | Set from |
|-------|------|----------|
| `Username` | `string` | The submitted `Login.Username` (even on auth failure) |
| `Status` | `string` | `'2'` for success, `'1'` for failure (verified: `auth.go:73` `var status = "2"` is the optimistic default; failure branches at lines 83, 91, 103 flip it to `"1"`) |
| `Msg` | `string` | Human-readable result string (e.g. `"登录成功"`, `"密码错误"`) |
| `Ipaddr` | `string` | From `c.ClientIP()` |
| `LoginLocation` | `string` | Geo-IP lookup of `Ipaddr` (best-effort; may be empty in dev) |
| `Browser` | `string` | Parsed from `User-Agent` (via `mssola/user_agent`) |
| `Os` | `string` | Same — User-Agent OS |
| `Platform` | `string` | Same — User-Agent platform |
| `LoginTime` | `time.Time` | `time.Now()` at the moment of `LoginLogToDB` invocation |
| `CreatedAt` / `UpdatedAt` | `time.Time` | GORM's autoUpdateTime / autoCreateTime |
| `Remark` | `string` | Reserved; usually empty |

**Critical timing wrinkle (R2)**: This is **async**. The HTTP login response
returns *before* the queue consumer has written the row. US3-3 verification
must tolerate a short delay; the learning doc SHALL instruct learners to
either `sleep 1` or re-poll the table.

---

## E4 — JWT claims (in-memory only — NOT a database entity)

**Path**: `common/middleware/handler/auth.go:21–35`, function `PayloadFunc`.
**Lifetime**: Built per-login, embedded in the signed JWT, decoded on each
authenticated subsequent request.
**Storage**: None. Lives in the JWT's signed body and in `gin.Context` after
the JWT middleware decodes it.

**Claim keys** (constants from gin-jwt SDK; literal claim names):

| Constant | Literal claim key | Source on `SysUser` / `SysRole` | Downstream consumer |
|----------|-------------------|---------------------------------|---------------------|
| `jwt.IdentityKey` | `identity` | `SysUser.UserId` | gin-jwt's `IdentityHandler`, used as the user identifier in protected handlers |
| `jwt.RoleIdKey` | `roleid` | `SysRole.RoleId` | Role-based middleware checks |
| `jwt.RoleKey` | `rolekey` | `SysRole.RoleKey` | **Casbin enforcement input** (subject) on protected endpoints |
| `jwt.NiceKey` | `nice` | `SysUser.Username` | Display name in audit logs |
| `jwt.DataScopeKey` | `datascope` | `SysRole.DataScope` | Data-scope filter middleware on list endpoints (e.g. for `GET /sys-user`) |
| `jwt.RoleNameKey` | `rolename` | `SysRole.RoleName` | Display only |

**Standard JWT envelope claims** (added by gin-jwt automatically, NOT by
`PayloadFunc`):
- `exp` — expiration (Unix seconds; per `settings.jwt.timeout`, default 3600s)
- `orig_iat` — original issue time (Unix seconds)

**Signing**: HS256 with the secret from `settings.jwt.secret`
(default `"go-admin"` — explicitly noted in `settings.yml` as MUST be changed
for production).

---

## Relationships

```text
SysUser.RoleId ─── 1 ────► SysRole.RoleId        (FK lookup at login.go:30)
SysUser.Username ──────►   sys_login_log.Username (recorded on every login)

PayloadFunc reads ─►   SysUser  + SysRole
            writes ─►  JWT claims (E4)
```

No new relationships introduced. Existing FK semantics (RoleId → RoleId)
are preserved.

---

## Validation rules surfaced by login

These are not new validations — they exist in source — but the learning doc
MUST surface them so learners understand "why does my login fail?":

1. `Login` struct binding (`common/middleware/handler/login.go:9–14`):
   `Username`, `Password`, `Code`, `UUID` are all `binding:"required"`.
   Missing any field returns `400` from gin's binding stage *before* the
   authenticator runs. **Implication**: the dev-mode bypass requires literal
   `"0"` strings, not empty strings.
2. User status filter (`login.go:18`): `status = '2'` (active). Non-active
   users get a "user not found" error, not a "user disabled" error — the
   query simply matches zero rows.
3. Captcha verification (`auth.go:88`): `captcha.Verify(uuid, code, true)`
   with `clear=true` — submitting the same captcha code twice fails the
   second time. The dev-mode `"0/0"` bypass exempts this rule.
4. Bcrypt comparison (`login.go:24`): wrong password produces an error from
   `pkg.CompareHashAndPassword`; the authenticator catches it and the login
   fails with a generic "auth failed" message (does not leak whether the
   user existed).
5. Role lookup (`login.go:30`): if `SysUser.RoleId` does not match any
   `SysRole`, the role lookup errors out and login fails. **In practice**
   the seeded admin user has a valid `RoleId`, so this branch is rarely hit
   in dev.

---

## Out of scope (data-model wise)

- Migrations: the schema is already in place from the existing
  `cmd/migrate/migration/version/1599190683659_tables.go` (cited but not
  modified).
- The `SysLoginLog` mirror struct under `cmd/migrate/migration/models/`:
  exists, but the learning doc focuses on the runtime model under
  `app/admin/models/`.
- `models.ControlBy` and `models.ModelTime` embeddings: footnoted only.
- Casbin policy table (`casbin_rule`): not touched by login (login bypasses
  Casbin); appears in the "Next steps" section as a hint for the future
  `GET /api/v1/sys-user` trace.
