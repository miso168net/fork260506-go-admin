---
type: community
cohesion: 0.25
members: 23
---

# Pagination & Search DTOs

**Cohesion:** 0.25 - loosely connected
**Members:** 23 nodes

## Members
- [[.GetAll()_2]] - code - app/admin/service/sys_dict_data.go
- [[.GetAll()_3]] - code - app/admin/service/sys_dict_type.go
- [[.GetNeedSearch()_13]] - code - app/jobs/service/dto/sys_job.go
- [[.GetPage()_11]] - code - app/admin/service/sys_api.go
- [[.GetPage()_12]] - code - app/admin/service/sys_config.go
- [[.GetPage()_13]] - code - app/admin/service/sys_dict_data.go
- [[.GetPage()_14]] - code - app/admin/service/sys_dict_type.go
- [[.GetPage()_15]] - code - app/admin/service/sys_login_log.go
- [[.GetPage()_17]] - code - app/admin/service/sys_opera_log.go
- [[.GetPage()_18]] - code - app/admin/service/sys_post.go
- [[.GetPage()_19]] - code - app/admin/service/sys_role.go
- [[.GetPage()_20]] - code - app/admin/service/sys_user.go
- [[.GetPageIndex()]] - code - common/dto/pagination.go
- [[.GetPageSize()]] - code - common/dto/pagination.go
- [[.GetWithKeyList()]] - code - app/admin/service/sys_config.go
- [[GeneralGetDto]] - code - common/dto/search.go
- [[IndexAction()]] - code - common/actions/index.go
- [[MakeCondition()]] - code - common/dto/search.go
- [[Paginate()]] - code - common/dto/search.go
- [[Pagination]] - code - common/dto/pagination.go
- [[index.go]] - code - common/actions/index.go
- [[pagination.go]] - code - common/dto/pagination.go
- [[search.go]] - code - common/dto/search.go

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/Pagination__Search_DTOs
SORT file.name ASC
```

## Connections to other communities
- 29 edges to [[_COMMUNITY_Common API & Service Layer]]
- 11 edges to [[_COMMUNITY_Sys Role Service & Audit ControlBy]]
- 8 edges to [[_COMMUNITY_Sys APIUser Service & DataScope]]
- 8 edges to [[_COMMUNITY_Sys Config  Dict  Post Services]]
- 5 edges to [[_COMMUNITY_Sys Menu Service & Order DTOs]]
- 2 edges to [[_COMMUNITY_Community 40]]
- 1 edge to [[_COMMUNITY_Community 33]]
- 1 edge to [[_COMMUNITY_Community 42]]
- 1 edge to [[_COMMUNITY_Community 46]]
- 1 edge to [[_COMMUNITY_Community 48]]
- 1 edge to [[_COMMUNITY_Frontend Babel Async-Arrow Edge Cases]]
- 1 edge to [[_COMMUNITY_Community 55]]

## Top bridge nodes
- [[IndexAction()]] - degree 20, connects to 8 communities
- [[.GetNeedSearch()_13]] - degree 17, connects to 3 communities
- [[.GetAll()_3]] - degree 6, connects to 3 communities
- [[MakeCondition()]] - degree 17, connects to 2 communities
- [[.GetPage()_11]] - degree 9, connects to 2 communities