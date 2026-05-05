---
type: community
cohesion: 0.29
members: 7
---

# LoginLog Model

**Cohesion:** 0.29 - loosely connected
**Members:** 7 nodes

## Members
- [[.Generate()_5]] - code - app/admin/models/sys_login_log.go
- [[.GetId()_5]] - code - app/admin/models/sys_login_log.go
- [[.TableName()_6]] - code - cmd/migrate/migration/models/sys_login_log.go
- [[SaveLoginLog()]] - code - app/admin/models/sys_login_log.go
- [[SysLoginLog_1]] - code - cmd/migrate/migration/models/sys_login_log.go
- [[sys_login_log.go_1]] - code - app/admin/models/sys_login_log.go
- [[sys_login_log.go_5]] - code - cmd/migrate/migration/models/sys_login_log.go

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/LoginLog_Model
SORT file.name ASC
```

## Connections to other communities
- 1 edge to [[_COMMUNITY_Admin REST APIs]]
- 1 edge to [[_COMMUNITY_Admin Models Layer]]

## Top bridge nodes
- [[SaveLoginLog()]] - degree 3, connects to 2 communities