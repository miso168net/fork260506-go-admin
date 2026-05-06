---
type: community
cohesion: 0.09
members: 28
---

# Job Scheduling Module

**Cohesion:** 0.09 - loosely connected
**Members:** 28 nodes

## Members
- [[.Generate()_36]] - code - app/jobs/models/sys_job.go
- [[.GetId()_54]] - code - app/jobs/models/sys_job.go
- [[.GetList()_1]] - code - app/jobs/models/sys_job.go
- [[.RemoveAllEntryID()]] - code - app/jobs/models/sys_job.go
- [[.Run()]] - code - app/jobs/jobbase.go
- [[.Run()_1]] - code - app/jobs/jobbase.go
- [[.SetCreateBy()]] - code - app/jobs/models/sys_job.go
- [[.SetUpdateBy()]] - code - app/jobs/models/sys_job.go
- [[.StartJob()]] - code - app/jobs/service/sys_job.go
- [[.TableName()_12]] - code - cmd/migrate/migration/models/sys_job.go
- [[.Update()_18]] - code - app/jobs/models/sys_job.go
- [[.addJob()_1]] - code - app/jobs/jobbase.go
- [[.addJob()]] - code - app/jobs/jobbase.go
- [[AddJob()]] - code - app/jobs/jobbase.go
- [[CallExec()]] - code - app/jobs/type.go
- [[ExecJob]] - code - app/jobs/jobbase.go
- [[HttpJob]] - code - app/jobs/jobbase.go
- [[Job]] - code - app/jobs/type.go
- [[JobCore]] - code - app/jobs/jobbase.go
- [[JobExec]] - code - app/jobs/type.go
- [[Setup()]] - code - app/jobs/jobbase.go
- [[SysJob_1]] - code - cmd/migrate/migration/models/sys_job.go
- [[SysJob_2]] - code - app/jobs/service/sys_job.go
- [[jobbase.go]] - code - app/jobs/jobbase.go
- [[sys_job.go_1]] - code - app/jobs/models/sys_job.go
- [[sys_job.go_3]] - code - app/jobs/service/sys_job.go
- [[sys_job.go_5]] - code - cmd/migrate/migration/models/sys_job.go
- [[type.go]] - code - app/jobs/type.go

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/Job_Scheduling_Module
SORT file.name ASC
```

## Connections to other communities
- 4 edges to [[_COMMUNITY_Common API & Service Layer]]
- 1 edge to [[_COMMUNITY_Sys Dept Service & General Del DTO]]
- 1 edge to [[_COMMUNITY_Frontend Babel Comments & Decorators]]
- 1 edge to [[_COMMUNITY_Community 30]]
- 1 edge to [[_COMMUNITY_Community 34]]
- 1 edge to [[_COMMUNITY_Community 58]]
- 1 edge to [[_COMMUNITY_Community 25]]

## Top bridge nodes
- [[Setup()]] - degree 7, connects to 4 communities
- [[.StartJob()]] - degree 4, connects to 2 communities
- [[jobbase.go]] - degree 6, connects to 1 community
- [[SysJob_2]] - degree 3, connects to 1 community
- [[CallExec()]] - degree 3, connects to 1 community