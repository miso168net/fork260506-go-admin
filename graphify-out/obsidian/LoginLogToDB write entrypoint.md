---
source_file: "specs/001-learning-trace-login/data-model.md"
type: "document"
community: "Login-Trace Spec: Data Model & Auth Concepts"
tags:
  - graphify/document
  - graphify/EXTRACTED
  - community/Login-Trace_Spec_Data_Model__Auth_Concepts
---

# LoginLogToDB write entrypoint

## Connections
- [[SysLoginLog (E3) GORM model]] - `shares_data_with` [EXTRACTED]
- [[gin-jwt Authenticator stage]] - `calls` [EXTRACTED]
- [[login.go login chain]] - `calls` [INFERRED]
- [[models.SaveLoginLog queue consumer]] - `calls` [EXTRACTED]
- [[sys_login_log async side effect]] - `references` [EXTRACTED]

#graphify/document #graphify/EXTRACTED #community/Login-Trace_Spec_Data_Model__Auth_Concepts