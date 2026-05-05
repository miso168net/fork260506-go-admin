---
type: community
cohesion: 1.00
members: 2
---

# DB Context Middleware

**Cohesion:** 1.00 - tightly connected
**Members:** 2 nodes

## Members
- [[WithContextDb()]] - code - common/middleware/db.go
- [[db.go]] - code - common/middleware/db.go

## Live Query (requires Dataview plugin)

```dataview
TABLE source_file, type FROM #community/DB_Context_Middleware
SORT file.name ASC
```

## Connections to other communities
- 1 edge to [[_COMMUNITY_Frontend Parser Internals]]

## Top bridge nodes
- [[WithContextDb()]] - degree 2, connects to 1 community