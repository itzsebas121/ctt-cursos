# Project: CTT-FISEI Enterprise Platform Ecosystem & Mobile-First Implementation

## Architecture
Multi-tier micro-ecosystem comprising:
- **Back-CTT**: FastAPI (Python 3.12) REST API, SQLModel/PostgreSQL (with SQLite in-memory fixtures for unit testing), ReportLab for RIDE PDF generation, bcrypt for platform authentication.
- **CTT-ADMIN**: React 19 + Vite 7 + Tailwind CSS v4 admin dashboard for congresses, sessions, attendance QR scanning, financial KPIs, Moodle exports, and Call for Papers reviews.
- **CTT-WEB**: React 19 + Vite 7 + Tailwind CSS v4 public portal & student dashboard for course/congress discovery, enrollments, student profile, digital QR pass, RIDE downloads, CFP paper submission, and CACES survey.

## Code Layout
- `Back-CTT/`
  - `src/models/`: SQLModel table definitions (`congress_model.py`, `enrollment.py`, `sesion_cronograma_model.py`, `session_attendance_model.py`, `scientific_paper_model.py`, `caces_survey_model.py`, `user_platform.py`).
  - `src/controllers/`: Business logic (`enrollment_controller.py`, `congress_controller.py`).
  - `src/routes/`: FastAPI routers (`congress_router.py`, `enrollments_router.py`, `users_platform_router.py`, `papers_router.py`).
  - `src/utils/`: Services and utilities (`pdf_utils.py`, `email_service.py`, `ride_generator.py`).
  - `tests/`: Pytest suites (`test_enterprise_features.py`, `conftest.py`).
- `CTT-ADMIN/`
  - `src/pages/congress/`: `CongresosDetalle.jsx` (main tabs: general, cronograma, inscritos, finanzas, papers).
  - `src/components/congress/`: `AttendanceScannerModal.jsx`, `PaperReviewModal.jsx`, financial cards.
  - `src/components/common/`: `Table.jsx`, `Modal.jsx`, `BottomSheets`.
- `CTT-WEB/`
  - `src/pages/`: `CongressDetail.jsx`, `CongresosDisponibles.jsx`.
  - `src/pages/application/`: `Perfil.jsx`, `CursosInscritos.jsx`, `Ordenes.jsx`, `Dashboard.jsx`.
  - `src/components/ui/`: `DigitalPassModal.jsx`, `PaymentReceiptModal.jsx`, `AttendanceScannerModal.jsx`, `SurveyModal.jsx`.
  - `src/components/layout/`: `Menu.jsx`, `Nav.jsx`.

## Feature Inventory
| # | Feature | Description | Milestone | Source |
|---|---------|-------------|-----------|--------|
| 1 | Capacity Control (`cupo_maximo`) | Backend validation blocking enrollments at quota; frontend capacity meter & registration lock | M1 | Master Plan §1.1 |
| 2 | QR Session Attendance | `session_attendance` table, backend `POST /attendance/scan` with 409 conflict, and dual mobile/USB scanner in CTT-ADMIN | M1 | Master Plan §1.2 |
| 3 | Comprehensive Student Profile | `/dashboard/perfil` in CTT-WEB with 4 tabs (Personal, SRI Defaults, Password bcrypt, Finance) | M1 | Master Plan §1.3 |
| 4 | Official UTA RIDE PDF | ReportLab generator with RUC `1860000880001`, SRI tax breakdown, and frontend download button | M2 | Master Plan §2.1 |
| 5 | Financial Reporting Dashboard | Revenue breakdown by tariffs ($20, $50, $80), balance, and conversion metrics in CTT-ADMIN | M2 | Master Plan §2.2 |
| 6 | Moodle LMS Exporter & Sync | Official 1-click Moodle CSV export (`username,firstname,lastname,email,course1,role1`) and optional API sync | M2 | Master Plan §2.3 |
| 7 | SMTP Email Engine | Asynchronous transactional notifications across 4 lifecycle moments with local dev fallback | M2 | Master Plan §2.4 |
| 8 | Call for Papers (CFP) | Scientific papers submission in CTT-WEB, review workflow in CTT-ADMIN, automatic $80 tariff assignment | M3 | Master Plan §3.1 |
| 9 | Hybrid Streaming Hub | Protected streaming links & downloadable materials for paid attendees | M3 | Master Plan §3.2 |
| 10 | CACES Academic Quality Survey | 3-question Likert survey (1-5 stars) and comments questionnaire in CTT-WEB | M3 | Master Plan §3.3 |
| 11 | Mobile-First & Media Query Auditing | Stacked cards on `<640px`, bottom sheets for dialogs, touch targets >= 44px, strict AGENTS.md compliance | M4 | Master Plan §2 |
| 12 | Automated Verification & Hardening | Pytest suite `tests/test_enterprise_features.py`, clean builds in CTT-WEB & CTT-ADMIN, adversarial hardening | M5 | Master Plan §4 |

## Milestones
| # | Name | Scope | Dependencies | Status |
|---|------|-------|-------------|--------|
| M1 | Operational Core (Wave 1) | Backend capacity validation, `session_attendance` table & scan endpoint, `AttendanceScannerModal.jsx` in CTT-ADMIN, `Perfil.jsx` in CTT-WEB | none | DONE |
| M2 | Finance, RIDE, LMS & SMTP (Wave 2) | Official UTA RIDE PDF generation, Financial Dashboard KPIs in CTT-ADMIN, Moodle CSV export, SMTP notification engine | M1 | DONE |
| M3 | Academic Innovation & Quality (Wave 3) | Call for Papers (CFP) model & UI, Hybrid Streaming hub, CACES quality survey | M2 | DONE |
| M4 | Mobile-First Deep Audit & UI Polish | Responsive transformation (<640px cards & bottom sheets, touch targets >= 44px), AGENTS.md compliance audit | M1, M2, M3 | DONE |
| M5 | Final E2E Verification & Adversarial Hardening | Run 100% automated test suite `test_enterprise_features.py`, verify production builds, white-box coverage audit | M4 | DONE |

## Interface Contracts
### 1. Capacity Control
- `POST /api/v1/enrollments`: Raises HTTP 400 when active count (`PAGADO`, `FACTURADO`, `INTERESADO`, `ORDEN_GENERADA`) >= `congresos.cupo_maximo`.
- Error payload: `{"detail": "El cupo máximo (N participantes) ha sido alcanzado. Convocatoria cerrada."}`

### 2. QR Attendance Check-In
- `POST /api/v1/congresses/{id_congreso}/attendance/scan`
- Request: `{"qr_payload": "UTA-FISEI-PASS-12-1", "id_sesion": 8}`
- Success (200): `{"status": "OK", "attendee_name": "Juan Perez", "tariff_type": "ESTUDIANTE_UTA", "total_sessions_attended": 3}`
- Duplicate (409): `{"detail": "El participante ya registró asistencia a esta sesión el 2026-11-08T09:30:00Z"}`

### 3. Student Profile & Password
- `PUT /api/v1/users/{id}/profile`: payload `{ "first_name": "...", "last_name": "...", "phone": "...", "address": "..." }` (cannot alter `identification`).
- `PUT /api/v1/users/{id}/password`: payload `{ "current_password": "...", "new_password": "..." }`. Validates with `user.verify_password` and hashes with `UserPlatform.hash_password`.
- `PUT /api/v1/users/{id}/billing`: payload `{ "billing_identification": "...", "billing_name": "...", "billing_address": "...", "billing_phone": "...", "billing_email": "..." }`.

### 4. Official UTA RIDE PDF
- `GET /api/v1/enrollments/{enrollment_id}/ride/download`
- Returns: `application/pdf`, filename `RIDE_UTA_{enrollment_id:09d}.pdf`.
- Contains: Institutional RUC `1860000880001`, SRI tax fields, itemized educational tariffs ($20, $50, $80), 0% IVA.

### 5. Financial Summary
- `GET /api/v1/congresses/{id}/financial-summary`
- Returns: `{"total_revenue": 4500.0, "total_budget": 5000.0, "tariffs_breakdown": {"estudiante_uta": 1200.0, "profesional": 2500.0, "ponente": 800.0}, "conversion_rate": 84.5}`

### 6. Moodle LMS Export
- `GET /api/v1/congresses/{id}/export/moodle-csv`
- Returns: `text/csv`, header `username,firstname,lastname,email,course1,role1`.
