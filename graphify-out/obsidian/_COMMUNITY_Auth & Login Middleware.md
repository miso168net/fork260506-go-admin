---
type: community
cohesion: 0.15
members: 17
---

# Auth & Login Middleware

**Cohesion:** 0.15 - loosely connected
**Members:** 17 nodes

## Members
- [[.GetPasswordHash()]] - code - common/models/user.go
- [[.GetUser()]] - code - common/middleware/handler/login.go
- [[.SetPassword()]] - code - common/models/user.go
- [[.Verify()]] - code - common/models/user.go
- [[.generateSalt()]] - code - common/models/user.go
- [[Authenticator()]] - code - common/middleware/handler/auth.go
- [[Authorizator()]] - code - common/middleware/handler/auth.go
- [[BaseUser]] - code - common/models/user.go
- [[IdentityHandler()]] - code - common/middleware/handler/auth.go
- [[LogOut()]] - code - common/middleware/handler/auth.go
- [[Login]] - code - common/middleware/handler/login.go
- [[LoginLogToDB()]] - code - common/middleware/handler/auth.go
- [[PayloadFunc()]] - code - common/middleware/handler/auth.go
- [[Unauthorized()]] - code - common/middleware/handler/auth.go
- [[auth.go_1]] - code - common/middleware/handler/auth.go
- [[login.go]] - code - common/middleware/handler/login.go
- [[user.go_1]] - code - common/models/user.go

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/Auth_&_Login_Middleware
SORT file.name ASC
```

## Connections to other communities
- 4 edges to [[_COMMUNITY_Admin REST APIs]]
- 1 edge to [[_COMMUNITY_Admin Router & Permissions]]

## Top bridge nodes
- [[LoginLogToDB()]] - degree 5, connects to 2 communities
- [[Authenticator()]] - degree 6, connects to 1 community
- [[.GetUser()]] - degree 3, connects to 1 community