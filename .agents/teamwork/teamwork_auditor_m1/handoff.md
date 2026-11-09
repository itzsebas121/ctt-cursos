# Forensic Audit Report: Wave 1 Operational Core Implementation

**Work Product**: Wave 1 Implementations across Back-CTT, CTT-WEB, CTT-ADMIN, and Backend Test Suite  
**Profile**: General Project  
**Integrity Mode**: Development Mode (with Demo/Benchmark Mode static rigor)  
**Verdict**: **CLEAN**  

---

## Forensic Audit Summary

| Check | Scope | Result | Details |
|---|---|:---:|---|
| **Capacity Control Query** | `enrollment_controller.py` | **PASS** | Genuine SQLModel `select(func.count(Enrollment.id))` with active statuses (`PAGADO`, `FACTURADO`, `INTERESADO`, `ORDEN_GENERADA`). No hardcoded bypasses or stubs. |
| **Session Attendance Model** | `session_attendance_model.py` | **PASS** | `UniqueConstraint("id_enrollment", "id_sesion", name="uq_enrollment_session")` defined at SQLModel level. |
| **Physical DB Schema** | PostgreSQL `session_attendance` | **PASS** | Table exists in `ctt_db` with `uq_enrollment_session` btree unique constraint and foreign key cascades. |
| **QR Scan & Duplicate Check** | `congress_router.py` | **PASS** | Genuine QR parsing (`UTA-FISEI-PASS-*`, JSON, numeric), verifies payment status (`PAGADO`, `FACTURADO`), raises HTTP 409 Conflict with check-in timestamp. |
| **Profile & Security** | `users_platform_router.py` | **PASS** | Cédula (`identification`) is strictly immutable; password update verifies current password and rehashes with `bcrypt`. |
| **Web Student Profile** | `CTT-WEB/src/pages/application/Perfil.jsx` | **PASS** | Authentic 4-tab interface, real API endpoints (`/users/{id}/profile`, `/billing`, `/password`), zero decorative pill badges/sparkles. |
| **Admin Attendance Scanner** | `CTT-ADMIN/src/components/congress/AttendanceScannerModal.jsx` | **PASS** | Authentic dual mode (camera via `Html5Qrcode` + USB barcode cadence listener <80ms), Web Audio API chimes, haptic vibration, bottom sheet layout. |
| **Test Suite Authenticity** | `tests/test_enterprise_features.py` | **PASS** | 17 comprehensive test cases, 0 trivial assertions (`assert True`), genuine HTTP status and payload checks. |
| **Runtime Test Execution** | `ctt-backend` container | **PASS** | `15 passed, 2 xfailed, 65 warnings in 42.35s` (Exit code 0). |
| **Frontend Production Builds** | `CTT-WEB` & `CTT-ADMIN` | **PASS** | Both apps compile cleanly with Vite in 25.78s and 23.67s respectively (Exit code 0). |

---

## 1. Observation

### 1.1 Backend Capacity Control (`Back-CTT/src/controllers/enrollment_controller.py:98-116`)
```python
if not is_course_enrollment:
    if getattr(resource, "cupo_maximo", None) and resource.cupo_maximo > 0:
        active_statuses = [
            EnrollmentStatus.PAGADO,
            EnrollmentStatus.FACTURADO,
            EnrollmentStatus.INTERESADO,
            EnrollmentStatus.ORDEN_GENERADA,
        ]
        active_count = db.exec(
            select(func.count(Enrollment.id)).where(
                Enrollment.id_congress == enrollment_data.id_congress,
                Enrollment.status.in_(active_statuses),
            )
        ).one()
        if active_count >= resource.cupo_maximo:
            raise ValueError(
                f"El cupo máximo ({resource.cupo_maximo} participantes) ha sido alcanzado. Convocatoria cerrada."
            )
```
- Real database aggregation query against PostgreSQL/SQLite session.
- No hardcoded IDs or conditional skips.
- Validates active statuses while allowing canceled/anulled enrollments to release quota.

### 1.2 Database Table & Constraint Forensics
Execution of `docker exec ctt-postgres psql -U ctt_app -d ctt_db -c "\d session_attendance"` produced:
```text
                                            Table "public.session_attendance"
       Column       |            Type             | Collation | Nullable |                    Default                     
--------------------+-----------------------------+-----------+----------+------------------------------------------------
 id                 | integer                     |           | not null | nextval('session_attendance_id_seq'::regclass)
 id_enrollment      | integer                     |           | not null | 
 id_sesion          | integer                     |           | not null | 
 check_in_time      | timestamp without time zone |           |          | (now() AT TIME ZONE 'utc'::text)
 scanned_by_user_id | integer                     |           |          | 
 notes              | character varying(255)      |           |          | 
Indexes:
    "session_attendance_pkey" PRIMARY KEY, btree (id)
    "idx_attendance_session" btree (id_sesion)
    "uq_enrollment_session" UNIQUE CONSTRAINT, btree (id_enrollment, id_sesion)
Foreign-key constraints:
    "session_attendance_id_enrollment_fkey" FOREIGN KEY (id_enrollment) REFERENCES enrollments(id) ON DELETE CASCADE
    "session_attendance_id_sesion_fkey" FOREIGN KEY (id_sesion) REFERENCES sesiones_cronograma(id_sesion) ON DELETE CASCADE
```
- Physical table exists in PostgreSQL.
- Unique constraint `uq_enrollment_session` is enforced at the database storage engine level.

### 1.3 QR Scan Endpoint & Duplicate Protection (`Back-CTT/src/routes/congress_router.py:587-704`)
- Genuine QR parser `_parse_qr_enrollment_id` decodes string patterns (`UTA-FISEI-PASS-{enrollment_id}-{congress_id}`), JSON payloads, and numeric IDs.
- Validates enrollment status is `EnrollmentStatus.PAGADO` or `EnrollmentStatus.FACTURADO`.
- Performs duplicate check:
  ```python
  existing = db.exec(
      select(SessionAttendance).where(
          SessionAttendance.id_enrollment == enrollment.id,
          SessionAttendance.id_sesion == payload.id_sesion
      )
  ).first()
  if existing:
      raise HTTPException(
          status_code=status.HTTP_409_CONFLICT,
          detail=f"El participante ya registró asistencia a esta sesión el {existing.check_in_time.isoformat()}"
      )
  ```
- Persists genuine check-in timestamp and returns aggregated attendance count for that student.

### 1.4 Profile & Security Endpoints (`Back-CTT/src/routes/users_platform_router.py`)
- `update_user_profile` updates only `first_name`, `second_name`, `last_name`, `phone`, `address`. `user.identification` is explicitly untouched, preventing cédula tampering.
- `update_user_password` verifies existing credentials with `user.verify_password(payload.current_password)`, rejects invalid passwords with HTTP 400 (`"Contraseña actual incorrecta"`), and hashes new passwords using `UserPlatform.hash_password(payload.new_password)` (bcrypt).

### 1.5 Frontend Authenticity & Style Directives
- `CTT-WEB/src/pages/application/Perfil.jsx`: Real 4-tab implementation (`personal`, `billing`, `security`, `finance`). Form submission invokes `api.put` with JWT bearer tokens. Heading structure is clean; 0 occurrences of `SparklesIcon` or decorative capsules over headings across `CTT-WEB/src/`.
- `CTT-ADMIN/src/components/congress/AttendanceScannerModal.jsx`: Dual scanner with Web Audio API (`AudioContext`, gain nodes, oscillators) for chimes and `navigator.vibrate` for haptic feedback. USB barcode reader captures keystroke bursts under 80ms cadence. Bottom sheet layout on `<640px` with touch targets `>= 44px`.

### 1.6 Runtime Test Execution
Execution of `docker exec ctt-backend pytest tests/test_enterprise_features.py -v`:
```text
============================= test session starts ==============================
platform linux -- Python 3.12.16, pytest-9.2.0, pluggy-1.7.0 -- /usr/local/bin/python3.12
cachedir: .pytest_cache
rootdir: /app
configfile: pytest.ini
plugins: cov-7.1.0, anyio-4.16.0
collecting ... collected 17 items

tests/test_enterprise_features.py::TestCapacityControl::test_congress_enrollment_allowed_under_capacity PASSED [  5%]
tests/test_enterprise_features.py::TestCapacityControl::test_congress_enrollment_blocked_when_capacity_reached PASSED [ 11%]
tests/test_enterprise_features.py::TestCapacityControl::test_direct_controller_call_raises_value_error_on_capacity PASSED [ 17%]
tests/test_enterprise_features.py::TestCapacityControl::test_canceled_enrollments_do_not_count_towards_capacity PASSED [ 23%]
tests/test_enterprise_features.py::TestSessionAttendanceQR::test_scan_attendance_success_and_total_count PASSED [ 29%]
tests/test_enterprise_features.py::TestSessionAttendanceQR::test_scan_attendance_duplicate_rejected_with_409 PASSED [ 35%]
tests/test_enterprise_features.py::TestSessionAttendanceQR::test_scan_attendance_rejected_for_unpaid_enrollment PASSED [ 41%]
tests/test_enterprise_features.py::TestSessionAttendanceQR::test_scan_attendance_supports_json_payload PASSED [ 47%]
tests/test_enterprise_features.py::TestSessionAttendanceQR::test_list_session_attendance PASSED [ 52%]
tests/test_enterprise_features.py::TestUserProfileAndSecurity::test_update_profile_success PASSED [ 58%]
tests/test_enterprise_features.py::TestUserProfileAndSecurity::test_update_profile_cannot_change_identification PASSED [ 64%]
tests/test_enterprise_features.py::TestUserProfileAndSecurity::test_update_password_wrong_current_password_rejected PASSED [ 70%]
tests/test_enterprise_features.py::TestUserProfileAndSecurity::test_update_password_success_verifies_with_bcrypt PASSED [ 76%]
tests/test_enterprise_features.py::TestUserProfileAndSecurity::test_get_user_profile PASSED [ 82%]
tests/test_enterprise_features.py::TestUserProfileAndSecurity::test_update_password_forbidden_for_other_user PASSED [ 88%]
tests/test_enterprise_features.py::TestRidePdfGeneration::test_ride_pdf_download_generates_valid_pdf_with_institutional_ruc XFAIL [ 94%]
tests/test_enterprise_features.py::TestMoodleCsvExport::test_moodle_csv_export_format_and_headers XFAIL [100%]

================= 15 passed, 2 xfailed, 65 warnings in 42.35s ==================
```
- 15 Wave 1 tests pass cleanly.
- 2 Wave 2 tests are gracefully marked `XFAIL` as planned until Wave 2 implementation starts.

### 1.7 Production Builds
- `CTT-WEB`: `pnpm run build` completed in 25.78s with exit code 0 (`1789 modules transformed`).
- `CTT-ADMIN`: `pnpm run build` completed in 23.67s with exit code 0 (`1387 modules transformed`).

---

## 2. Logic Chain

1. **Anti-Cheat Verification**:
   - Analyzed AST and raw source code of `enrollment_controller.py`, `congress_router.py`, and `users_platform_router.py`. No hardcoded student IDs, no static returns, no facade stubs, and no bypassed validations exist.
2. **Double-Layered Integrity**:
   - Attendance duplicate protection is enforced at both the endpoint logic level (raising HTTP 409 with exact timestamp) and the PostgreSQL storage level (`uq_enrollment_session`), guaranteeing resilience against concurrent race conditions.
3. **Security Invariance**:
   - By locking `identification` in both backend routes (`update_user_profile`) and frontend forms (`Perfil.jsx`), student identities cannot be impersonated or modified post-registration.
4. **Authentic Frontend Implementations**:
   - Both `Perfil.jsx` and `AttendanceScannerModal.jsx` interact with real backend endpoints, manage loading states, and handle conflict responses accurately.
5. **Empirical Reproduction**:
   - The auditor executed tests and builds directly in the target runtime environments (`ctt-postgres`, `ctt-backend`, and local Node v26 toolchain), corroborating that all claims match raw execution outputs.

---

## 3. Caveats

- **Container Test Syncing**: The `ctt-backend` container mounts `/src`, but `/tests` is built into the container root. Future additions to `Back-CTT/tests/` should be synced via `docker cp` or a compose volume mount.
- **Wave 2 Stubs**: The two test cases in `TestRidePdfGeneration` and `TestMoodleCsvExport` are set to `XFAIL` by design until Wave 2 (Finance & LMS) is delivered.

---

## 4. Conclusion

The Wave 1 implementation across Back-CTT, CTT-WEB, CTT-ADMIN, and the automated test suite contains zero integrity violations, zero facades, zero hardcoded cheat paths, and zero decorative rule breaches. The implementation is genuine, resilient, and verified.

**Official Verdict**: **CLEAN**

---

## 5. Verification Method

To reproduce all forensic checks:

1. **Verify Database Physical Table & Constraint**:
   ```bash
   docker exec ctt-postgres psql -U ctt_app -d ctt_db -c "\d session_attendance"
   ```
2. **Execute Automated Enterprise Backend Test Suite**:
   ```bash
   docker exec ctt-backend pytest tests/test_enterprise_features.py -v
   ```
3. **Verify CTT-WEB Production Build**:
   ```bash
   export PATH="/home/sebas/.nvm/versions/node/v26.13.0/bin:$PATH"
   cd /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-WEB && pnpm run build
   ```
4. **Verify CTT-ADMIN Production Build**:
   ```bash
   export PATH="/home/sebas/.nvm/versions/node/v26.13.0/bin:$PATH"
   cd /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-ADMIN && pnpm run build
   ```
5. **Verify Zero Decorative Sparkles/Pills over Headings**:
   ```bash
   grep -rn "SparklesIcon" /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-WEB/src/
   ```
