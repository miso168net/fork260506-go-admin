---
source_file: "specs/001-learning-trace-login/data-model.md"
type: "document"
community: "Login-Trace Spec: Data Model & Auth Concepts"
tags:
  - graphify/document
  - graphify/EXTRACTED
  - community/Login-Trace_Spec_Data_Model__Auth_Concepts
---

# login.go login chain

## Connections
- [[Active user status filter (status='2')]] - `implements` [EXTRACTED]
- [[Bcrypt password comparison]] - `calls` [EXTRACTED]
- [[Login request binding struct]] - `references` [EXTRACTED]
- [[LoginLogToDB write entrypoint]] - `calls` [INFERRED]
- [[Read-only boundary mandate (FR-016)]] - `rationale_for` [EXTRACTED]
- [[SysRole (E2) GORM model]] - `references` [EXTRACTED]
- [[SysUser (E1) GORM model]] - `references` [EXTRACTED]
- [[captcha.Verify validation step]] - `calls` [EXTRACTED]

#graphify/document #graphify/EXTRACTED #community/Login-Trace_Spec_Data_Model__Auth_Concepts