---
type: community
cohesion: 0.18
members: 20
---

# Sys API/User Service & DataScope

**Cohesion:** 0.18 - loosely connected
**Members:** 20 nodes

## Members
- [[.CheckStorageSysApi()]] - code - app/admin/service/sys_api.go
- [[.Get()_11]] - code - app/admin/service/sys_api.go
- [[.Get()_21]] - code - app/admin/service/sys_user.go
- [[.Remove()]] - code - app/admin/service/sys_api.go
- [[.Remove()_10]] - code - app/admin/service/sys_user.go
- [[.ResetPwd()_1]] - code - app/admin/service/sys_user.go
- [[.TableName()_23]] - code - common/models/migrate.go
- [[.Update()_9]] - code - app/admin/service/sys_api.go
- [[.Update()_17]] - code - app/admin/service/sys_user.go
- [[.UpdateAvatar()]] - code - app/admin/service/sys_user.go
- [[.UpdatePwd()_1]] - code - app/admin/service/sys_user.go
- [[.UpdateStatus()_2]] - code - app/admin/service/sys_user.go
- [[GetDataScope (method, dead code)]] - code - app/admin/models/datascope.go
- [[Migration_1]] - code - common/models/migrate.go
- [[Permission()]] - code - common/actions/permission.go
- [[SysApi_2]] - code - app/admin/service/sys_api.go
- [[SysUser_2]] - code - app/admin/service/sys_user.go
- [[migrate.go]] - code - common/models/migrate.go
- [[sys_api.go_3]] - code - app/admin/service/sys_api.go
- [[sys_user.go_3]] - code - app/admin/service/sys_user.go

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/Sys_API/User_Service__DataScope
SORT file.name ASC
```

## Connections to other communities
- 25 edges to [[_COMMUNITY_Common API & Service Layer]]
- 8 edges to [[_COMMUNITY_Pagination & Search DTOs]]
- 7 edges to [[_COMMUNITY_Sys Role Service & Audit ControlBy]]
- 5 edges to [[_COMMUNITY_Sys Config  Dict  Post Services]]
- 2 edges to [[_COMMUNITY_Community 33]]
- 2 edges to [[_COMMUNITY_Community 42]]
- 1 edge to [[_COMMUNITY_Job Scheduling Module]]

## Top bridge nodes
- [[.TableName()_23]] - degree 19, connects to 4 communities
- [[Permission()]] - degree 19, connects to 3 communities
- [[SysUser_2]] - degree 11, connects to 3 communities
- [[.Remove()]] - degree 6, connects to 3 communities
- [[.Remove()_10]] - degree 6, connects to 3 communities