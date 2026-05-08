---
type: community
cohesion: 0.15
members: 22
---

# Sys Role Service & Audit ControlBy

**Cohesion:** 0.15 - loosely connected
**Members:** 22 nodes

## Members
- [[.AddError()_1]] - code - common/service/service.go
- [[.Get()_12]] - code - app/admin/service/sys_config.go
- [[.Get()_17]] - code - app/admin/service/sys_menu.go
- [[.Get()_20]] - code - app/admin/service/sys_role.go
- [[.GetById()]] - code - app/admin/service/sys_role.go
- [[.GetRoleMenuId()]] - code - app/admin/service/sys_role.go
- [[.GetWithName()]] - code - app/admin/service/sys_role.go
- [[.Insert()_12]] - code - app/admin/service/sys_menu.go
- [[.Insert()_15]] - code - app/admin/service/sys_role.go
- [[.Remove()_6]] - code - app/admin/service/sys_menu.go
- [[.Remove()_9]] - code - app/admin/service/sys_role.go
- [[.Update()_14]] - code - app/admin/service/sys_menu.go
- [[.Update()_16]] - code - app/admin/service/sys_role.go
- [[.UpdateDataScope()]] - code - app/admin/service/sys_role.go
- [[.initPaths()]] - code - app/admin/service/sys_menu.go
- [[ControlBy]] - code - common/models/by.go
- [[Model]] - code - common/models/by.go
- [[ModelTime]] - code - common/models/by.go
- [[SysRole_2]] - code - app/admin/service/sys_role.go
- [[by.go]] - code - cmd/migrate/migration/models/by.go
- [[by.go_1]] - code - common/models/by.go
- [[sys_role.go_3]] - code - app/admin/service/sys_role.go

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/Sys_Role_Service__Audit_ControlBy
SORT file.name ASC
```

## Connections to other communities
- 18 edges to [[_COMMUNITY_Common API & Service Layer]]
- 14 edges to [[_COMMUNITY_Sys Menu Service & Order DTOs]]
- 13 edges to [[_COMMUNITY_Sys Config  Dict  Post Services]]
- 11 edges to [[_COMMUNITY_Pagination & Search DTOs]]
- 7 edges to [[_COMMUNITY_Sys APIUser Service & DataScope]]
- 5 edges to [[_COMMUNITY_Community 40]]
- 5 edges to [[_COMMUNITY_Community 33]]
- 2 edges to [[_COMMUNITY_Frontend Babel Top-Level Letters]]
- 1 edge to [[_COMMUNITY_Community 61]]
- 1 edge to [[_COMMUNITY_Community 52]]
- 1 edge to [[_COMMUNITY_TinyMCE Form Generator Bundle]]

## Top bridge nodes
- [[Model]] - degree 47, connects to 8 communities
- [[.AddError()_1]] - degree 11, connects to 4 communities
- [[.Update()_14]] - degree 8, connects to 4 communities
- [[.Update()_16]] - degree 6, connects to 4 communities
- [[.UpdateDataScope()]] - degree 6, connects to 3 communities