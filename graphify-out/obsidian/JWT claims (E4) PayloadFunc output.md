---
source_file: "specs/001-learning-trace-login/data-model.md"
type: "document"
community: "Login-Trace Spec: Data Model & Auth Concepts"
tags:
  - graphify/document
  - graphify/EXTRACTED
  - community/Login-Trace_Spec_Data_Model__Auth_Concepts
---

# JWT claims (E4) PayloadFunc output

## Connections
- [[Casbin enforcement (rolekey subject)]] - `shares_data_with` [EXTRACTED]
- [[Data-scope filter middleware]] - `shares_data_with` [EXTRACTED]
- [[HS256 signing with settings.jwt.secret]] - `implements` [EXTRACTED]
- [[Login 5-key success response shape]] - `shares_data_with` [EXTRACTED]
- [[PayloadFunc (auth.go)]] - `implements` [EXTRACTED]
- [[Standard JWT envelope claims (exp, orig_iat)]] - `conceptually_related_to` [EXTRACTED]
- [[Step 3 JWT base64 decode of claims]] - `references` [EXTRACTED]

#graphify/document #graphify/EXTRACTED #community/Login-Trace_Spec_Data_Model__Auth_Concepts