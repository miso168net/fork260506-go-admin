<!-- SPECKIT START -->
Active SpecKit feature: `001-learning-trace-login`
Current plan (Technical Context, Constitution gates, project structure):
[specs/001-learning-trace-login/plan.md](specs/001-learning-trace-login/plan.md)

Companion artifacts in the same feature directory:
- `spec.md` — user stories, FRs, SCs, clarifications
- `research.md` — login-chain layer catalogue (file:line), LoginLog async-queue note, Dockerfile.learning research
- `data-model.md` — sys_user / sys_role / sys_login_log fields read by login + JWT claim shape
- `contracts/post-login.md`, `contracts/get-captcha.md` — REST contracts (documented existing behaviour)
- `quickstart.md` — 5-minute "clone → JWT" path
- `tasks.md` — Phase 2 task list (T001–T022, present and executed)
<!-- SPECKIT END -->

## graphify

This project has a graphify knowledge graph at graphify-out/.

Rules:
- Before answering architecture or codebase questions, read graphify-out/GRAPH_REPORT.md for god nodes and community structure
- If graphify-out/wiki/index.md exists, navigate it instead of reading raw files
- After modifying code files in this session, run `graphify update .` to keep the graph current (AST-only, no API cost)
