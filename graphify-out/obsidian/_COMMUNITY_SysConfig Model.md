---
type: community
cohesion: 0.33
members: 6
---

# SysConfig Model

**Cohesion:** 0.33 - loosely connected
**Members:** 6 nodes

## Members
- [[.Generate()_1]] - code - app/admin/models/sys_config.go
- [[.GetId()_1]] - code - app/admin/models/sys_config.go
- [[.TableName()_2]] - code - cmd/migrate/migration/models/sys_config.go
- [[SysConfig_1]] - code - cmd/migrate/migration/models/sys_config.go
- [[sys_config.go_1]] - code - app/admin/models/sys_config.go
- [[sys_config.go_5]] - code - cmd/migrate/migration/models/sys_config.go

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/SysConfig_Model
SORT file.name ASC
```
