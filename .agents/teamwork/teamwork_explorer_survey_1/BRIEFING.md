# BRIEFING — 2026-11-08T04:06:00Z

## Mission
Survey the Back-CTT repository to map existing backend architecture, models, schemas, and routes across 8 core domain areas (enrollment, attendance, user/SRI, RIDE PDF, Moodle LMS, email, CFP/streaming/CACES, test infra).

## 🔒 My Identity
- Archetype: teamwork_explorer
- Roles: explorer, analyst, synthesizer
- Working directory: /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_1
- Original parent: 5173c970-207a-4e4b-805b-9e7e4e5b56af
- Milestone: backend_survey

## 🔒 Key Constraints
- Read-only investigation — do NOT implement
- Write ONLY to /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_1/
- Produce analysis.md and handoff.md
- Verify all findings with exact file paths and line numbers
- Respect AGENTS.md rules

## Current Parent
- Conversation ID: 5173c970-207a-4e4b-805b-9e7e4e5b56af
- Updated: 2026-11-08T04:06:00Z

## Investigation State
- **Explored paths**:
  - `Back-CTT/src/models/congress_model.py` (lines 1-185)
  - `Back-CTT/src/models/enrollment.py` (lines 1-146)
  - `Back-CTT/src/controllers/enrollment_controller.py` (lines 1-140)
  - `Back-CTT/src/routes/enrollments_router.py` (lines 1-371)
  - `Back-CTT/src/models/sesion_cronograma_model.py` (lines 1-100)
  - `Back-CTT/src/models/user_platform.py` (lines 1-55)
  - `Back-CTT/src/routes/users_platform_router.py` (lines 1-80)
  - `Back-CTT/src/routes/platform_auth_router.py` (lines 1-93)
  - `Back-CTT/src/utils/pdf_utils.py` (lines 1-100)
  - `Back-CTT/tests/conftest.py` (lines 1-400)
  - `docker-compose.yml`, `Dockerfile`, database schemas in PostgreSQL container.
- **Key findings**:
  - `cupo_maximo` exists in `Congress` model but is not checked in `EnrollmentController.create_enrollment`.
  - `sesiones_cronograma` exists in PostgreSQL; `session_attendance` table needs to be created.
  - QR payload format in `DigitalPassModal.jsx` is `UTA-FISEI-PASS-{enrollment.id}-{congress_id}`.
  - `users_platform` already has all 5 SRI billing columns in DB; `PUT /api/v1/users/{id}/billing` exists. Needs personal profile and password endpoints.
  - `reportlab 5.1.0`, `qrcode 8.2`, `pillow 13.0.0` already installed in container.
  - Moodle LMS, SMTP engine, CFP, Streaming Hub, and CACES survey models/routes do not exist yet.
  - Pytest runs inside `ctt-backend` against SQLite in-memory (531 tests passing).
- **Unexplored areas**: None. All 8 required dimensions thoroughly mapped.

## Key Decisions Made
- Confirmed tests run via `docker exec ctt-backend pytest`.
- Verified database table names (`congresos`, `sesiones_cronograma`, `users_platform`).
- Verified ReportLab and QR generation libraries are already present in runtime.

## Artifact Index
- `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_1/DISPATCH.md` — Incoming instructions
- `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_1/BRIEFING.md` — Persistent memory
- `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_1/progress.md` — Liveness & status
- `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_1/analysis.md` — In-depth architectural analysis
- `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_1/handoff.md` — 5-component handoff report
