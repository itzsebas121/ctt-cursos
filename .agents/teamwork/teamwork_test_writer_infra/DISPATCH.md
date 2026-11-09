## 2026-11-08T04:08:36Z

You are a teamwork_test_writer authoring the automated backend test suite for enterprise features in Back-CTT.
Your working directory is: /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_test_writer_infra

MANDATORY INPUTS:
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/ORIGINAL_REQUEST.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/PROJECT.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/TEST_INFRA.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_1/handoff.md
- Read /home/sebas/.gemini/antigravity/brain/e4725dea-975f-4023-949a-7ead631c99a2/plan_ecosistema_empresarial_completo_ctt.md
- Review project rules in /home/sebas/Workspace/Desarrollo/ctt-cursos/AGENTS.md

MANDATORY INTEGRITY WARNING:
DO NOT CHEAT. All implementations must be genuine. DO NOT hardcode test results, create dummy/facade implementations, or circumvent the intended task. A teamwork_auditor will independently verify your work. Integrity violations WILL be detected and your work WILL be rejected.

EXCLUSIVE WRITE OWNERSHIP:
You own and may ONLY edit files inside `Back-CTT/tests/`:
- `Back-CTT/tests/test_enterprise_features.py` (create new)

YOUR TASK:
Author a comprehensive, robust test suite in `Back-CTT/tests/test_enterprise_features.py` that utilizes the existing pytest test setup (`client`, `db_session` from `conftest.py` with in-memory SQLite):
1. Test Capacity Control (`cupo_maximo`):
   - Create a congress with `cupo_maximo = 2`.
   - Enroll 2 participants successfully.
   - Attempting to enroll a 3rd participant must be rejected with HTTP 400 (or ValueError handled by router).
2. Test QR Session Attendance Check-in:
   - Create a congress, session, and paid enrollment.
   - Scan valid QR payload `UTA-FISEI-PASS-{enrollment_id}-{congress_id}` for session: asserts HTTP 200, status "OK", attendee name returned.
   - Scan duplicate for the same session: asserts HTTP 409 Conflict with detail mentioning previous check-in.
   - Scan for unpaid enrollment (status `INTERESADO`): asserts HTTP 400.
3. Test User Profile & Password Update:
   - Create platform user with password "Secret123!".
   - Update profile (first_name, last_name, phone, address): asserts updated in DB and cannot alter identification.
   - Update password with correct current password: asserts success, new password verifies with bcrypt, old password no longer works.
   - Update password with incorrect current password: asserts HTTP 400.
4. Test RIDE PDF Generation (stub/future proof for Wave 2):
   - Test `GET /api/v1/enrollments/{id}/ride/download`: asserts response content-type is `application/pdf`, contains header `%PDF`, and contains institutional RUC `1860000880001` or valid byte stream.
5. Test Moodle CSV Export (stub/future proof for Wave 2):
   - Test `GET /api/v1/congresses/{id}/export/moodle-csv`: asserts response is CSV with header `username,firstname,lastname,email,course1,role1` and enrolled students.

VERIFICATION:
- Run tests via `docker exec ctt-backend pytest tests/test_enterprise_features.py -v`.
- Note: If some Wave 2 endpoints (RIDE/Moodle) are not yet implemented in Back-CTT while the worker is finishing, mark those specific tests with `@pytest.mark.xfail(reason="Wave 2 implementation in progress")` or structure them so the Wave 1 tests pass 100% immediately.
- Document test suite execution and results in `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_test_writer_infra/handoff.md`.
- Send a message when complete.
