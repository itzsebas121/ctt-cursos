## 2026-11-09T01:16:41Z
MANDATORY: Read `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/ORIGINAL_REQUEST.md`, `/home/sebas/Workspace/Desarrollo/ctt-cursos/AGENTS.md`, and `/home/sebas/Workspace/Desarrollo/ctt-cursos/PROJECT.md` before starting work.

Identity:
- Archetype: teamwork_reviewer
- Role: Documentation Cross-Validation Reviewer & Auditor
- Working directory: /home/sebas/Workspace/Desarrollo/ctt-cursos
- State folder: /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/reviewer_docs/

Mission (Requirement R6 & Acceptance Criteria):
Perform an independent, exhaustive technical cross-validation and audit of the entire technical documentation generated in `docs/`:
- `docs/README.md`
- `docs/arquitectura/`
- `docs/base-de-datos/`
- `docs/backend/`
- `docs/frontend-web/`
- `docs/frontend-admin/`
- `docs/despliegue/`

Audit Checklists & Verification Tasks:
1. Directory Structure & File Completeness:
   - Verify that all 6 required folders exist under `docs/`: `arquitectura/`, `base-de-datos/`, `backend/`, `frontend-web/`, `frontend-admin/`, `despliegue/`.
   - Verify that all documents are present and non-empty.
2. Language & Institutional Rules (AGENTS.md):
   - Verify that 100% of documentation is in Spanish.
   - Verify that NO decorative capsules, pill badges, or overlines exist above headings in any markdown file.
   - Verify institutional color `#00345e` and clear typographic hierarchy.
3. Code Alignment & Technical Accuracy:
   - Compare `docs/base-de-datos/` models with actual models in `Back-CTT/app/models/` and `Back-CTT/src/models/` (fields, constraints, types).
   - Compare `docs/backend/` endpoints and controllers with actual routers in `Back-CTT/`.
   - Compare `docs/frontend-web/` and `docs/frontend-admin/` with actual components, routes, GSAP animations, and styles in `CTT-WEB/` and `CTT-ADMIN/`.
   - Compare `docs/despliegue/` with actual `docker-compose.yml`, Dockerfiles, and environment configurations.
4. Mermaid Diagram Validation:
   - Parse and validate every Mermaid diagram (`erDiagram`, `sequenceDiagram`, `flowchart`, `graph`, `C4Context`) across all `.md` files in `docs/`.
   - Ensure there are no syntax errors, no invalid tokens, and that they render cleanly.
5. Relative Link Integrity:
   - Programmatically or systematically verify all relative links `[text](target.md)` across all markdown files in `docs/`.
   - Confirm zero broken links (all referenced files and paths exist).
6. Quality Gate Verdict:
   - Issue an explicit verdict: APPROVE or REQUEST_CHANGES.
   - Document all verification methods, evidence, test scripts or checks executed in `.agents/teamwork/reviewer_docs/handoff.md`.
   - Send completion message to orchestrator via `send_message`.
