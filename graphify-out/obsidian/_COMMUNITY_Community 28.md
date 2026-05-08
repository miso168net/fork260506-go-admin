---
type: community
cohesion: 0.24
members: 17
---

# Community 28

**Cohesion:** 0.24 - loosely connected
**Members:** 17 nodes

## Members
- [[FR-010 - cross-repo SDK code requires separate spec (no deep-trace into core)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Layer 5.11 - Response assembly via SDK default LoginResponse (5-key) vs LogOutUnauthorized (2-key)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Layer 5.2 - gin-jwt LoginHandler SDK boundary (stub, FR-010)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Layer 5.3 - Authenticator (auth.go63-107) - first project callback, defer LoginLogToDB]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Layer 5.4 - Captcha verify outer if !=dev (auth.go88) clear-after-verify flag true]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Layer 5.5 - Login struct + Login.GetUser sys_user query (login.go9-14, 16-33)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Layer 5.6 - pkg.CompareHashAndPassword bcrypt compare (login.go22) - the only CPU-bound step]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Layer 5.7 - GORM role lookup sys_role (login.go27) - 2nd independent SQL round-trip]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Layer 5.8 - PayloadFunc flatten (user, role) - jwt.MapClaims (auth.go21-35)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Layer 5.9 - JWT TokenGenerator HS256 SDK boundary (stub, FR-010)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Recipe (a) FR-012a - GORM SQL capture (sys_user + sys_role two SELECTs + INSERT sys_login_log)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Section 4 - Mermaid sequence diagram (11-actor login chain with two SDK rect boundaries)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[Section 7 - 3 SDK boundaries LoginHandler, TokenGenerator, default LoginResponse]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[commonmiddlewarehandlerauth.go (AuthenticatorPayloadFuncIdentityHandlerAuthorizatorLoginLogToDBUnauthorizedLogOut)]] - rationale - x_fork.tree-20260506.md
- [[commonmiddlewarehandlerlogin.go (Login struct + GetUser bcrypt + sys_role lookup)]] - rationale - x_fork.tree-20260506.md
- [[fork260506-go-admin-core sibling repo (sdkpkgjwtauth houses gin-jwt SDK)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md
- [[settings.jwt.secret default literal 'go-admin' (HS256 signing key)]] - rationale - docs/superpowers/specs/2026-05-07-login-trace.md

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/Community_28
SORT file.name ASC
```

## Connections to other communities
- 7 edges to [[_COMMUNITY_Community 53]]
- 6 edges to [[_COMMUNITY_Community 44]]
- 4 edges to [[_COMMUNITY_Community 49]]
- 1 edge to [[_COMMUNITY_Community 70]]
- 1 edge to [[_COMMUNITY_Community 26]]

## Top bridge nodes
- [[commonmiddlewarehandlerauth.go (AuthenticatorPayloadFuncIdentityHandlerAuthorizatorLoginLogToDBUnauthorizedLogOut)]] - degree 8, connects to 3 communities
- [[Section 4 - Mermaid sequence diagram (11-actor login chain with two SDK rect boundaries)]] - degree 12, connects to 2 communities
- [[commonmiddlewarehandlerlogin.go (Login struct + GetUser bcrypt + sys_role lookup)]] - degree 6, connects to 2 communities
- [[Layer 5.11 - Response assembly via SDK default LoginResponse (5-key) vs LogOutUnauthorized (2-key)]] - degree 5, connects to 2 communities
- [[Recipe (a) FR-012a - GORM SQL capture (sys_user + sys_role two SELECTs + INSERT sys_login_log)]] - degree 4, connects to 2 communities