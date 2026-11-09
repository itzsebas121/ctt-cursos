# Handoff Report: Wave 1 (Operational Core) Backend Implementation

**Agent**: `teamwork_worker_m1_back`  
**Date**: 2026-11-08  
**Handoff Type**: Hard (Mission Complete)  
**Target Path**: `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_worker_m1_back/handoff.md`  

---

## 1. Observation

- **Capacity Control (`cupo_maximo`)**:
  - `Back-CTT/src/controllers/enrollment_controller.py`: Added capacity checking inside `create_enrollment` for congresses (`not is_course_enrollment`).
  - Active enrollments are counted using `func.count(Enrollment.id)` filtered by statuses `[EnrollmentStatus.PAGADO, EnrollmentStatus.FACTURADO, EnrollmentStatus.INTERESADO, EnrollmentStatus.ORDEN_GENERADA]`.
  - When `active_count >= resource.cupo_maximo`, raises `ValueError(f"El cupo máximo ({resource.cupo_maximo} participantes) ha sido alcanzado. Convocatoria cerrada.")`.
  - In `create_enrollment`, `tariff_type` and `amount` are now persisted and serialized in the response.

- **Session Attendance Model & DDL**:
  - `Back-CTT/src/models/session_attendance_model.py`: Created `SessionAttendance` SQLModel with:
    - `id: Optional[int] = Field(default=None, primary_key=True)`
    - `id_enrollment: int = Field(foreign_key="enrollments.id", nullable=False)`
    - `id_sesion: int = Field(foreign_key="sesiones_cronograma.id_sesion", nullable=False)`
    - `check_in_time: datetime = Field(default_factory=datetime.utcnow, sa_column=Column(DateTime, nullable=False, default=datetime.utcnow))`
    - `scanned_by_user_id: Optional[int] = Field(default=None, nullable=True)`
    - `notes: Optional[str] = Field(default=None, max_length=255, nullable=True)`
    - `UniqueConstraint("id_enrollment", "id_sesion", name="uq_enrollment_session")`
  - Registered in `Back-CTT/src/models/__init__.py` alongside `Enrollment` and `UserPlatform` to ensure `SQLModel.metadata.create_all` resolves all relational foreign keys in SQLite test environments.
  - Applied DDL in PostgreSQL `ctt_db`:
    ```sql
    CREATE TABLE IF NOT EXISTS session_attendance (
        id SERIAL PRIMARY KEY,
        id_enrollment INTEGER NOT NULL REFERENCES enrollments(id) ON DELETE CASCADE,
        id_sesion INTEGER NOT NULL REFERENCES sesiones_cronograma(id_sesion) ON DELETE CASCADE,
        check_in_time TIMESTAMP WITHOUT TIME ZONE DEFAULT (NOW() AT TIME ZONE 'utc'),
        scanned_by_user_id INTEGER,
        notes VARCHAR(255),
        CONSTRAINT uq_enrollment_session UNIQUE (id_enrollment, id_sesion)
    );
    CREATE INDEX IF NOT EXISTS idx_attendance_session ON session_attendance(id_sesion);
    ```
  - Added Alembic migration `Back-CTT/alembic/versions/2026_11_08_0415-c3f81e29a001_create_session_attendance_table.py` and executed `alembic upgrade head`.

- **Attendance QR Scanning & Session Listing Endpoints**:
  - `Back-CTT/src/routes/congress_router.py`:
    - `POST /api/v1/congresses/{id_congreso}/attendance/scan` (and alias `/api/v1/congresses-new/{id_congreso}/attendance/scan`):
      - Robust QR parser (`_parse_qr_enrollment_id`) parses `UTA-FISEI-PASS-{enrollment_id}-{congress_id}`, raw numeric strings, URL-encoded tokens, and JSON payloads with `credential` / `id_enrollment`.
      - Verifies congress existence (404), session existence and congress match (404/400), enrollment existence and congress match (404/400).
      - Verifies enrollment status is `PAGADO` or `FACTURADO` (400 if not).
      - Rejects duplicate check-in with `HTTP 409 Conflict` and detail `f"El participante ya registró asistencia a esta sesión el {existing.check_in_time.isoformat()}"`.
      - Inserts `SessionAttendance` record.
      - Calculates `total_sessions_attended` for the participant in that congress.
      - Returns `{"status": "OK", "attendee_name": f"{user.first_name} {user.last_name}", "tariff_type": enrollment.tariff_type, "total_sessions_attended": total_sessions}`.
    - `GET /api/v1/congresses/{id_congreso}/attendance/session/{id_sesion}` (and alias `/api/v1/congresses-new/{id_congreso}/attendance/session/{id_sesion}`):
      - Returns `{ "id_congreso": id_congreso, "id_sesion": id_sesion, "total_attendees": len(attendees), "data": [...] }`.

- **User Profile & Security Endpoints**:
  - `Back-CTT/src/routes/users_platform_router.py`:
    - `PUT /api/v1/users/{user_id}/profile`: Updates `first_name`, `last_name`, `phone`/`cellphone`, `address`. Explicitly preserves `user.identification` (cédula cannot be modified).
    - `PUT /api/v1/users/{user_id}/password`: Accepts `current_password` and `new_password`. Verifies current password using `user.verify_password` inside a safe try/except block. Rejects with `HTTP 400 Bad Request` `"Contraseña actual incorrecta"` if invalid. Hashes and updates with `UserPlatform.hash_password(new_password)`.
    - `GET /api/v1/users/{user_id}` and `GET /api/v1/users/{user_id}/profile`: Returns user profile data and authorized SRI billing fields.

---

## 2. Logic Chain

1. **Capacity Enforcement**: Enforcing capacity control directly in `EnrollmentController.create_enrollment` ensures that regardless of whether an enrollment request arrives via frontend, API, or batch process, the count of active enrollments (`PAGADO`, `FACTURADO`, `INTERESADO`, `ORDEN_GENERADA`) cannot exceed `resource.cupo_maximo`. Anulled enrollments are excluded, allowing spots to be recycled if a reservation is canceled.
2. **Attendance Model Integrity**: Adding `UniqueConstraint("id_enrollment", "id_sesion", name="uq_enrollment_session")` guarantees database-level uniqueness, while endpoint-level validation provides a human-readable `409 Conflict` error with exact previous check-in timestamp.
3. **Dual URL Prefix Support**: By creating route aliases inside `congress_router.routes` for both `/api/v1/congresses-new` and `/api/v1/congresses`, legacy routes, frontend components (`CTT-ADMIN`), and external integrations function transparently without breakage.
4. **Resilient Security Verification**: By wrapping `user.verify_password` in a defensive try/except block against `ValueError: Invalid salt`, the backend handles legacy literal mock passwords or corrupt hashes gracefully with `HTTP 400 Bad Request` instead of unhandled 500 crashes.
5. **Cédula Inmutability**: By strictly omitting `identification` from update mutations in `update_user_profile`, Ecuadorian legal identification (cédula/RUC) remains tamper-proof after registration.

---

## 3. Caveats

- **Container File Sync**: The running Docker container `ctt-backend` copies files during image build and only binds `/logs` and `/static/`. Changes made on the host were copied into the container via `docker cp` and tested directly in the container runtime. Any future host edits must similarly sync or rebuild.
- **Wave 2 Feature Tests**: In `Back-CTT/tests/test_enterprise_features.py`, the two future-proof Wave 2 stubs (`test_ride_pdf_download_generates_valid_pdf_with_institutional_ruc` and `test_moodle_csv_export_format_and_headers`) are marked `XFAIL` as planned until Wave 2 implementation commences.

---

## 4. Conclusion

All Wave 1 Backend requirements have been fully implemented with zero regressions:
- Capacity control strictly limits congress registration to `cupo_maximo`.
- `SessionAttendance` model and PostgreSQL table exist with unique constraint on `(id_enrollment, id_sesion)`.
- QR scan endpoint (`POST /attendance/scan`) correctly validates enrollment and payment status, enforces duplicate rejection with HTTP 409 Conflict, and returns attendee stats.
- Session attendee listing endpoint (`GET /attendance/session/{id_sesion}`) returns real attendance records.
- User profile update prevents cédula mutation while allowing name/phone/address updates.
- Password change validates current password and applies bcrypt rehashing.
- 100% of Wave 1 enterprise tests pass in Docker (15 passed, 2 xfailed for Wave 2).
- Pre-existing enrollment and congress test suites pass with zero regressions (25 passed in enrollments, 34 passed in congress).

---

## 5. Verification Method

To independently verify the implementation:

1. **Run Enterprise Features Test Suite**:
   ```bash
   docker exec ctt-backend pytest tests/test_enterprise_features.py -v
   ```
   *Expected*: `15 passed, 2 xfailed, 65 warnings in ~40s`.

2. **Run Enrollments Router Test Suite**:
   ```bash
   docker exec ctt-backend pytest tests/test_routes/test_enrollments_router.py -v
   ```
   *Expected*: `25 passed in ~8s`.

3. **Run Congress Router Test Suite**:
   ```bash
   docker exec ctt-backend pytest tests/test_routes/test_congress_router.py -v
   ```
   *Expected*: `34 passed in ~48s`.

4. **Verify PostgreSQL Schema**:
   ```bash
   docker exec ctt-postgres psql -U ctt_app -d ctt_db -c "\d session_attendance"
   ```
   *Expected*: Displays table `session_attendance` with `idx_attendance_session` index and `uq_enrollment_session` unique constraint.
