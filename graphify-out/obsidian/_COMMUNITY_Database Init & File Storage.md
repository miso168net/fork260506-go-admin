---
type: community
cohesion: 0.13
members: 19
---

# Database Init & File Storage

**Cohesion:** 0.13 - loosely connected
**Members:** 19 nodes

## Members
- [[.SetDb()]] - code - cmd/migrate/migration/init.go
- [[.Setup()]] - code - common/file_store/initialize.go
- [[NoCache()]] - code - common/middleware/header.go
- [[OXS]] - code - common/file_store/initialize.go
- [[Options()]] - code - common/middleware/header.go
- [[Secure()]] - code - common/middleware/header.go
- [[Setup()_1]] - code - common/storage/initialize.go
- [[TestOBSUpload()]] - code - common/file_store/obs_test.go
- [[TestOSSUpload()]] - code - common/file_store/oss_test.go
- [[header.go]] - code - common/middleware/header.go
- [[initialize.go]] - code - common/database/initialize.go
- [[initialize.go_1]] - code - common/file_store/initialize.go
- [[initialize.go_2]] - code - common/storage/initialize.go
- [[obs_test.go]] - code - common/file_store/obs_test.go
- [[oss_test.go]] - code - common/file_store/oss_test.go
- [[setupCache()]] - code - common/storage/initialize.go
- [[setupCaptcha()]] - code - common/storage/initialize.go
- [[setupQueue()]] - code - common/storage/initialize.go
- [[setupSimpleDatabase()]] - code - common/database/initialize.go

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/Database_Init__File_Storage
SORT file.name ASC
```

## Connections to other communities
- 5 edges to [[_COMMUNITY_Community 25]]
- 4 edges to [[_COMMUNITY_Common API & Service Layer]]
- 2 edges to [[_COMMUNITY_Community 63]]
- 2 edges to [[_COMMUNITY_Community 31]]
- 2 edges to [[_COMMUNITY_Frontend Babel Async-Arrow Edge Cases]]
- 1 edge to [[_COMMUNITY_Sys Menu Service & Order DTOs]]
- 1 edge to [[_COMMUNITY_Job Scheduling Module]]
- 1 edge to [[_COMMUNITY_Community 52]]
- 1 edge to [[_COMMUNITY_TinyMCE Form Generator Bundle]]

## Top bridge nodes
- [[Setup()_1]] - degree 15, connects to 3 communities
- [[setupSimpleDatabase()]] - degree 6, connects to 2 communities
- [[setupQueue()]] - degree 4, connects to 2 communities
- [[TestOBSUpload()]] - degree 4, connects to 2 communities
- [[TestOSSUpload()]] - degree 4, connects to 2 communities