---
type: community
cohesion: 0.08
members: 37
---

# Login-Trace Spec: Data Model & Auth Concepts

**Cohesion:** 0.08 - loosely connected
**Members:** 37 nodes

## Members
- [[Active user status filter (status='2')]] - rationale - specs/001-learning-trace-login/data-model.md
- [[Async login-log write timing wrinkle (R2)]] - rationale - specs/001-learning-trace-login/data-model.md
- [[Bcrypt password comparison]] - document - specs/001-learning-trace-login/data-model.md
- [[Casbin enforcement (rolekey subject)]] - document - specs/001-learning-trace-login/data-model.md
- [[Constitution Principle V (audit logging)]] - rationale - specs/001-learning-trace-login/contracts/post-login.md
- [[Data-scope filter middleware]] - document - specs/001-learning-trace-login/data-model.md
- [[Dev-mode captcha bypass (code='0',uuid='0')]] - rationale - specs/001-learning-trace-login/data-model.md
- [[Dev-mode captcha bypass (sentinel both '0')]] - rationale - specs/001-learning-trace-login/contracts/post-login.md
- [[FR-016 read-only contract preservation]] - rationale - specs/001-learning-trace-login/contracts/post-login.md
- [[HS256 signing with settings.jwt.secret]] - document - specs/001-learning-trace-login/data-model.md
- [[JWT claims (E4) PayloadFunc output]] - document - specs/001-learning-trace-login/data-model.md
- [[Login 5-key success response shape]] - document - specs/001-learning-trace-login/contracts/post-login.md
- [[Login failure-mode response matrix]] - document - specs/001-learning-trace-login/contracts/post-login.md
- [[Login request binding struct]] - document - specs/001-learning-trace-login/data-model.md
- [[LoginLogToDB write entrypoint]] - document - specs/001-learning-trace-login/data-model.md
- [[POST apiv1login endpoint contract]] - document - specs/001-learning-trace-login/contracts/post-login.md
- [[POST apiv1login request body schema]] - document - specs/001-learning-trace-login/contracts/post-login.md
- [[PayloadFunc (auth.go)]] - document - specs/001-learning-trace-login/data-model.md
- [[Quickstart troubleshooting matrix]] - document - specs/001-learning-trace-login/quickstart.md
- [[Read-only boundary mandate (FR-016)]] - rationale - specs/001-learning-trace-login/data-model.md
- [[Related get-captcha]] - document - specs/001-learning-trace-login/contracts/post-login.md
- [[Standard JWT envelope claims (exp, orig_iat)]] - document - specs/001-learning-trace-login/data-model.md
- [[Step 2 login curl with dev-mode bypass]] - document - specs/001-learning-trace-login/quickstart.md
- [[Step 3 JWT base64 decode of claims]] - document - specs/001-learning-trace-login/quickstart.md
- [[Step 4 GET apiv1getinfo protected call]] - document - specs/001-learning-trace-login/quickstart.md
- [[SysLoginLog (E3) GORM model]] - document - specs/001-learning-trace-login/data-model.md
- [[SysRole (E2) GORM model]] - document - specs/001-learning-trace-login/data-model.md
- [[SysUser (E1) GORM model]] - document - specs/001-learning-trace-login/data-model.md
- [[antd-pro front-end compatibility (currentAuthority alias)]] - rationale - specs/001-learning-trace-login/quickstart.md
- [[captcha.Verify validation step]] - document - specs/001-learning-trace-login/data-model.md
- [[docslearninglogin-trace.md (full learning doc)]] - document - specs/001-learning-trace-login/quickstart.md
- [[get-captcha.md non-bypass flow contract]] - document - specs/001-learning-trace-login/quickstart.md
- [[gin-jwt Authenticator stage]] - document - specs/001-learning-trace-login/contracts/post-login.md
- [[gin-jwt default LoginResponse callback]] - rationale - specs/001-learning-trace-login/contracts/post-login.md
- [[login.go login chain]] - document - specs/001-learning-trace-login/data-model.md
- [[models.SaveLoginLog queue consumer]] - document - specs/001-learning-trace-login/data-model.md
- [[sys_login_log async side effect]] - document - specs/001-learning-trace-login/contracts/post-login.md

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/Login-Trace_Spec_Data_Model__Auth_Concepts
SORT file.name ASC
```
