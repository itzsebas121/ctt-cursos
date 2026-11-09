# Handoff Report: Back-CTT Architectural Survey

**Agent**: `teamwork_explorer_survey_1`  
**Date**: 2026-11-08  
**Handoff Type**: Hard (Mission Complete)  
**Target Path**: `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_1/handoff.md`

---

## 1. Observation

- **Capacity & Enrollment Models**:
  - `Back-CTT/src/models/congress_model.py:31`: `cupo_maximo: Optional[int] = Field(default=250)` exists on table `congresos`.
  - `Back-CTT/src/models/enrollment.py:18-58`: Table `enrollments` has statuses `INTERESADO`, `ORDEN_GENERADA`, `PAGADO`, `FACTURADO`, `ANULADO`, and fields `tariff_type` and `amount`.
  - `Back-CTT/src/controllers/enrollment_controller.py:40-105`: `create_enrollment` validates resource existence and duplicate enrollment, but does **not** count active enrollments against `resource.cupo_maximo`.
- **Attendance & Sessions**:
  - `Back-CTT/src/models/sesion_cronograma_model.py:13-25`: Table `sesiones_cronograma` exists with primary key `id_sesion` and foreign keys `id_congreso` and `id_speaker`.
  - In PostgreSQL `ctt_db`, the table is named `sesiones_cronograma`.
  - `Back-CTT/src/routes/congress_router.py`: Does not contain any attendance scan routes.
  - `CTT-WEB/src/components/ui/DigitalPassModal.jsx:67`: QR payload encodes `passCode = "UTA-FISEI-PASS-${enrollment.id}-${enrollment.id_congress || 1}"` inside an object or raw string.
  - Table `session_attendance` does not exist yet in PostgreSQL.
- **User Profile & SRI Tax Fields**:
  - `Back-CTT/src/models/user_platform.py:31-36`: Already includes `billing_identification`, `billing_name`, `billing_address`, `billing_phone`, `billing_email`.
  - Direct PostgreSQL check (`\d users_platform`) verified that all 5 billing columns exist in the active database.
  - `Back-CTT/src/models/user_platform.py:43-55`: Implements `verify_password` and `hash_password` using `bcrypt`.
  - `Back-CTT/src/routes/users_platform_router.py:21-58`: Already exposes `PUT /api/v1/users/{user_id}/billing`.
  - Missing profile endpoints: `PUT /api/v1/users/{user_id}/profile` (with cédula protection) and `PUT /api/v1/users/{user_id}/password` (validating current password before bcrypt rehashing).
- **PDF & RIDE Generation**:
  - Container inspection of `ctt-backend`: `reportlab 5.1.0`, `qrcode 8.2`, and `pillow 13.0.0` are already installed in Python 3.12.
  - `Back-CTT/src/utils/pdf_utils.py` and `Back-CTT/src/routes/pdf_router.py`: Only handle static article PDF uploads and SHA256 deduplication.
  - No RIDE PDF generation routine exists.
- **Moodle Integration**:
  - No references to `moodle` exist anywhere in `Back-CTT`, `CTT-ADMIN`, or `CTT-WEB`.
- **Email & SMTP Service**:
  - No `email_service.py` or SMTP configuration exists in `Back-CTT/.env`.
- **CFP, Streaming & CACES**:
  - No tables or endpoints exist for `scientific_papers`, streaming links, or `caces_surveys`.
- **Test Infrastructure**:
  - `Back-CTT/tests/conftest.py` configures in-memory SQLite (`sqlite:///:memory:`) with `StaticPool` and `PRAGMA foreign_keys=ON`.
  - `FastAPI` `TestClient` is configured with `dependency_overrides`.
  - Full suite run in container yielded: 531 passed, 1 failed (date logic in `test_get_upcoming_congresses_with_details_eager_loads_sponsor`), 1 skipped out of 533 tests.
  - No `Back-CTT/tests/test_enterprise_features.py` exists yet.

---

## 2. Logic Chain

1. **Capacity Control Validation**: Because `Congress.cupo_maximo` is already an attribute of the congress model, adding capacity enforcement in `EnrollmentController.create_enrollment` requires counting active enrollments in status `PAGADO`, `FACTURADO`, `INTERESADO`, `ORDEN_GENERADA` and raising a `ValueError` if `count >= resource.cupo_maximo`. The router catches `ValueError` and maps it to `HTTP 400 Bad Request`.
2. **Attendance Model Alignment**: Because the existing session schedule table is named `sesiones_cronograma` (with PK `id_sesion`), creating `SessionAttendance` with foreign key `ForeignKey("sesiones_cronograma.id_sesion")` and `UniqueConstraint("id_enrollment", "id_sesion")` guarantees relational integrity and duplicate prevention at the database level.
3. **QR Check-in Flow**: Because the frontend QR payload in `DigitalPassModal.jsx` generates `UTA-FISEI-PASS-{enrollment_id}-{id_congreso}`, the backend check-in endpoint can parse either raw pass string or JSON payload to identify `id_enrollment` and `id_congreso`, verify valid payment status, and return `HTTP 409 Conflict` on duplicate check-in.
4. **Student Profile**: Because `users_platform` already stores `billing_*` columns and has `PUT /api/v1/users/{user_id}/billing`, completing the student profile only requires adding `PUT /api/v1/users/{user_id}/profile` (preventing updates to `identification`) and `PUT /api/v1/users/{user_id}/password` (invoking `user.verify_password` and `UserPlatform.hash_password`).
5. **RIDE PDF Generation**: Because `reportlab 5.1.0` and `qrcode 8.2` are already present in the backend runtime, generating the official UTA RIDE PDF in memory (`io.BytesIO`) with RUC `1860000880001` and returning `StreamingResponse` can be implemented without adding new external package dependencies.
6. **Moodle CSV Export**: Because enrollment and congress models have all student and course metadata, generating the official Moodle batch CSV (`username,firstname,lastname,email,course1,role1`) is a direct mapping from paid/facturado enrollments.
7. **Email Service**: Creating `src/utils/email_service.py` with asynchronous/threaded sending and an automatic development fallback (logging when `SMTP_HOST` is unset) ensures that enrollment and checkout workflows succeed even in offline or staging environments.
8. **Automated Testing Suite**: Because `tests/conftest.py` provides SQLite in-memory fixtures and `TestClient`, `tests/test_enterprise_features.py` can be authored and executed inside `ctt-backend` to verify all enterprise features in isolation.

---

## 3. Caveats

- **SQLite vs PostgreSQL in Unit Tests**: Unit tests use SQLite in-memory. Date functions, partial unique indexes, and schema types must remain standard SQL to pass under both SQLite and PostgreSQL.
- **Docker Container vs Host Python**: The host machine lacks Python 3.12 / pip in its default PATH (`/usr/bin/python3` is 3.14 without pip), but `ctt-backend` is actively running with Python 3.12 and all required libraries (`reportlab`, `qrcode`, `bcrypt`, `pytest`). All backend tests must be run via `docker exec ctt-backend pytest`.
- **Existing Pre-survey Test Failure**: Exactly 1 test in `test_congress_controller.py` fails due to hardcoded future date expectations. This is pre-existing and does not affect enterprise features.

---

## 4. Conclusion

The Back-CTT architecture is ready for Wave 1, Wave 2, and Wave 3 implementations. All prerequisite libraries (`reportlab`, `qrcode`, `pillow`, `bcrypt`) and database columns (SRI billing fields) are already in place. The missing logic consists of:
- Enforcement of `cupo_maximo` in `EnrollmentController`.
- `SessionAttendance` model and scan endpoint with 409 duplicate rejection.
- Personal profile and password update endpoints in `users_platform_router.py`.
- In-memory RIDE PDF generation with ReportLab and institutional RUC `1860000880001`.
- Moodle CSV export endpoint.
- Asynchronous SMTP engine with development fallback.
- `scientific_papers`, streaming hub, and `caces_surveys` models and routes.
- Verification test suite in `tests/test_enterprise_features.py`.

---

## 5. Verification Method

To independently verify the observations and architecture:
1. **Check PostgreSQL schema**:
   ```bash
   docker exec ctt-postgres psql -U ctt_app -d ctt_db -c "\d users_platform"
   docker exec ctt-postgres psql -U ctt_app -d ctt_db -c "\d sesiones_cronograma"
   ```
2. **Check container libraries**:
   ```bash
   docker exec ctt-backend pip list | grep -E "reportlab|qrcode|bcrypt|pytest"
   ```
3. **Execute existing test suite**:
   ```bash
   docker exec ctt-backend pytest tests/test_routes/test_enrollments_router.py -v
   ```
4. **Inspect artifacts**:
   - `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_1/analysis.md`
   - `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_1/handoff.md`
