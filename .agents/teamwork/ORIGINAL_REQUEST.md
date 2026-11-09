# Original User Request

## Initial Request — 2026-11-08T03:53:36Z

Execute the production enterprise implementation plan for the CTT-FISEI platform across Back-CTT, CTT-ADMIN, and CTT-WEB, delivering Wave 1, Wave 2, and Wave 3 sequentially with Mobile-First priority and media query auditing.

Integrity mode: development

Reference master plan: `/home/sebas/.gemini/antigravity/brain/e4725dea-975f-4023-949a-7ead631c99a2/plan_ecosistema_empresarial_completo_ctt.md`

## Requirements

### R1. Operational Core (Wave 1)
- Capacity control (`cupo_maximo`): Enforce backend validation in `EnrollmentController.create_enrollment` blocking enrollments when `cupo_maximo` is reached, and display capacity badge/lock in CTT-WEB.
- QR Session Attendance: Implement `session_attendance` table in PostgreSQL with a unique constraint on `(id_enrollment, id_sesion)`, backend check-in endpoint `POST /api/v1/congresses/{id_congreso}/attendance/scan` with duplicate detection, and `AttendanceScannerModal.jsx` in CTT-ADMIN with dual support for mobile rear camera and USB barcode scanners.
- Comprehensive Student Profile: Implement `/dashboard/perfil` in CTT-WEB with tabs for Personal Info, SRI Taxpayer Billing defaults, Security/Password change using bcrypt, and Financial Order History.

### R2. Accounting, Finance, LMS & SMTP (Wave 2)
- Official UTA RIDE PDF: Generate downloadable payment receipts with institutional RUC `1860000880001`, tariff breakdown, and SRI billing data.
- Financial Reporting Dashboard: Provide revenue breakdown by tariffs ($20, $50, $80), balance, and conversion metrics in CTT-ADMIN.
- Moodle LMS Exporter & Sync: Provide official 1-click Moodle CSV batch download and optional REST API sync via `core_enrol_manual_enrol_users`.
- SMTP Email Engine: Implement asynchronous transactional notifications (Order created, Payment approved with QR pass, Receipt rejected with notes, 24h event reminder) with local development fallback when credentials are not configured.

### R3. Academic Innovation & Quality (Wave 3)
- Call for Papers (CFP): Scientific papers table and submission/review workflow in CTT-WEB and CTT-ADMIN, automatically assigning the $80 speaker tariff upon paper approval.
- Hybrid Streaming Hub: Provide protected live streaming links and downloadable materials for paid attendees.
- CACES Academic Quality Survey: Implement a 3-question Likert rating (1-5 stars) and comments questionnaire.

### R4. Mobile-First & UI Standards
- Conduct media query auditing across viewports:
  - `< 640px` (Smartphones): Convert tables to structured cards, display dialogs as bottom sheets, ensure touch targets are at least 44x44px, and provide a full-width camera QR scanner.
  - `640px - 1024px` (Tablets): Use 2-column grids and compact tables.
  - `> 1024px` (Desktop): Maintain full desktop layouts.
- Strict UI compliance: No decorative pill badges or sparkle capsules over section headings; clean typographic hierarchy; institutional blue `#00345e`; high-contrast readability in both light and dark modes.

## Acceptance Criteria

### Automated Verification
- [ ] Backend test suite `Back-CTT/tests/test_enterprise_features.py` is created and passes, verifying capacity limit checks, attendance QR scanning and duplicate handling, RIDE PDF generation, Moodle CSV export, and profile password updates.
- [ ] Production build succeeds without errors in both frontend applications: `pnpm run build` in `CTT-WEB` and `pnpm run build` in `CTT-ADMIN`.

### Functional and Responsive Verification
- [ ] In mobile viewport (<640px), tables in `CursosInscritos` render as stacked structured cards without horizontal overflow, and modal dialogs open as bottom sheets.
- [ ] Session attendance check-in correctly rejects duplicate scans for the same session with a 409 conflict and registers valid scans with session timestamps.
- [ ] Generated RIDE PDF includes institutional RUC `1860000880001`, SRI tax information, and itemized educational tariff breakdown.
- [ ] No decorative pill badges or sparkle icons appear above section headings across all modified or newly introduced views.
