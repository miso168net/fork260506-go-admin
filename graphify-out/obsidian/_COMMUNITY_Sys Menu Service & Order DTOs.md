---
type: community
cohesion: 0.13
members: 26
---

# Sys Menu Service & Order DTOs

**Cohesion:** 0.13 - loosely connected
**Members:** 26 nodes

## Members
- [[.Bind()_3]] - code - common/dto/generate.go
- [[.GetId()_56]] - code - common/dto/generate.go
- [[.GetIds()]] - code - common/dto/search.go
- [[.GetList()]] - code - app/admin/service/sys_menu.go
- [[.GetPage()_16]] - code - app/admin/service/sys_menu.go
- [[.GetSysMenuByRoleName()]] - code - app/admin/service/sys_menu.go
- [[.Len()]] - code - app/admin/models/sys_menu.go
- [[.Less()]] - code - app/admin/models/sys_menu.go
- [[.SetLabel()]] - code - app/admin/service/sys_menu.go
- [[.SetMenuRole()]] - code - app/admin/service/sys_menu.go
- [[.Swap()]] - code - app/admin/models/sys_menu.go
- [[.getByRoleName()]] - code - app/admin/service/sys_menu.go
- [[CustomError()]] - code - common/middleware/customerror.go
- [[GeneralDelDto]] - code - common/dto/search.go
- [[ObjectById]] - code - common/dto/generate.go
- [[OrderDest()]] - code - common/dto/order.go
- [[SysMenu_2]] - code - app/admin/service/sys_menu.go
- [[SysMenuSlice]] - code - app/admin/models/sys_menu.go
- [[customerror.go]] - code - common/middleware/customerror.go
- [[menuCall()]] - code - app/admin/service/sys_menu.go
- [[menuDistinct()]] - code - app/admin/service/sys_menu.go
- [[menuLabelCall()]] - code - app/admin/service/sys_menu.go
- [[order.go]] - code - common/dto/order.go
- [[recursiveSetMenu()]] - code - app/admin/service/sys_menu.go
- [[sys_menu.go_1]] - code - app/admin/models/sys_menu.go
- [[sys_menu.go_3]] - code - app/admin/service/sys_menu.go

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/Sys_Menu_Service__Order_DTOs
SORT file.name ASC
```

## Connections to other communities
- 14 edges to [[_COMMUNITY_Sys Role Service & Audit ControlBy]]
- 12 edges to [[_COMMUNITY_Common API & Service Layer]]
- 6 edges to [[_COMMUNITY_Community 40]]
- 5 edges to [[_COMMUNITY_Pagination & Search DTOs]]
- 2 edges to [[_COMMUNITY_Community 56]]
- 1 edge to [[_COMMUNITY_Community 62]]
- 1 edge to [[_COMMUNITY_Community 72]]
- 1 edge to [[_COMMUNITY_Community 58]]
- 1 edge to [[_COMMUNITY_Sys User DTOs]]
- 1 edge to [[_COMMUNITY_Job Scheduling Module]]
- 1 edge to [[_COMMUNITY_Community 61]]
- 1 edge to [[_COMMUNITY_Frontend Babel Top-Level Letters]]
- 1 edge to [[_COMMUNITY_Community 25]]
- 1 edge to [[_COMMUNITY_Database Init & File Storage]]
- 1 edge to [[_COMMUNITY_Community 30]]
- 1 edge to [[_COMMUNITY_Frontend Babel Async-Arrow Edge Cases]]

## Top bridge nodes
- [[.Len()]] - degree 40, connects to 13 communities
- [[.GetPage()_16]] - degree 9, connects to 3 communities
- [[.GetList()]] - degree 5, connects to 2 communities
- [[CustomError()]] - degree 4, connects to 2 communities
- [[SysMenu_2]] - degree 12, connects to 1 community