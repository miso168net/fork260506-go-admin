---
source_file: "x_fork.tree-20260506.md"
type: "rationale"
community: "Community 54"
tags:
  - graphify/rationale
  - graphify/EXTRACTED
  - community/Community_54
---

# app/admin/ (13 resources x 4-layer architecture: apis/models/router/service)

## Connections
- [[app (business modules admin  jobs  other)]] - `references` [EXTRACTED]
- [[appadminapis (gin handler layer MakeContext-Bind-service-Custom)]] - `references` [EXTRACTED]
- [[appadminmodels (GORM models for sys_ tables)]] - `references` [EXTRACTED]
- [[appadminrouter (route registration, mounted by cmdapiserver.go)]] - `references` [EXTRACTED]
- [[appadminservice (business logic GORM query + aggregation)]] - `references` [EXTRACTED]

#graphify/rationale #graphify/EXTRACTED #community/Community_54