## 2026-11-08T03:53:36Z

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


## 2026-11-09T01:04:35Z

Trabajar con varios subagentes: uno para base de datos, otro para backend, otro para los frontends y otro que vaya revisando todo el trabajo. Generar en `docs/` toda la documentación técnica de la aplicación tanto backend como frontend en español, organizada en carpetas estructuradas y basada en el código existente en `Back-CTT/`, `CTT-WEB/` y `CTT-ADMIN/`.

Integrity mode: demo
The team must NOT: ejecutar scripts externos o delegar la ejecución a herramientas externas

## Requirements

### R1. Documentación de Base de Datos y Modelos
Documentar los modelos relacionales, diagramas entidad-relación en Mermaid, diccionario de datos completo, esquemas de migración con Alembic y semillas/datos iniciales (incluyendo modelos de eventos oficiales FISEI UTA como CSEI y CISED).

### R2. Documentación del Backend (FastAPI)
Documentar la arquitectura modular de FastAPI en `Back-CTT/` (Python 3.12, SQLModel/SQLAlchemy, PostgreSQL 16), routers, controladores, sistema de autenticación JWT dual (Admin y Plataforma), generador de reportes PDF con ReportLab (RIDE SRI y Órdenes de Pago UTA), catálogo de endpoints y contratos de datos.

### R3. Documentación de los Frontends (Estudiante y Administrativo)
Documentar la arquitectura de `CTT-WEB/` (React 19 + Vite + Tailwind CSS + GSAP, puerto 3000) y `CTT-ADMIN/` (React 19 + Vite + Tailwind CSS, puerto 3001), detallando el catálogo unificado, gestión de pagos y órdenes, sistema de rutas, animaciones GSAP y diseño responsive.

### R4. Guías de Despliegue y Arquitectura Global
Documentar la visión general del sistema, arquitectura integrada entre backend y frontends, y guías de despliegue para desarrollo local y configuración estándar con Docker/producción según los archivos existentes en el repositorio.

### R5. Cumplimiento de Reglas Institucionales y Organización
Organizar toda la documentación en `docs/` en carpetas estructuradas (`arquitectura/`, `base-de-datos/`, `backend/`, `frontend-web/`, `frontend-admin/`, `despliegue/`), redactada íntegramente en español, cumpliendo estrictamente con las reglas de `AGENTS.md` (sin cápsulas, pill badges o overlines decorativos en encabezados, identidad visual con azul institucional `#00345e`).

### R6. Validación Cruzada y Auditoría
El subagente revisor / auditor debe validar de forma cruzada toda la documentación generada en `docs/`, verificando coherencia técnica con el código fuente, validez del formato Markdown, renderizado de diagramas Mermaid e integridad de enlaces.

## Acceptance Criteria

### Integridad y Cobertura Técnica
- [ ] La estructura de directorios en `docs/` contiene las carpetas `arquitectura/`, `base-de-datos/`, `backend/`, `frontend-web/`, `frontend-admin/` y `despliegue/`.
- [ ] Toda la documentación técnica está redactada en español.
- [ ] Los modelos relacionales, migraciones Alembic y diagramas ER en Mermaid coinciden con las definiciones en `Back-CTT/app/models/`.
- [ ] Todos los routers y endpoints del backend (`Back-CTT/`) están documentados con sus métodos HTTP, rutas, parámetros y contratos de respuesta.
- [ ] Los dos frontends (`CTT-WEB/` y `CTT-ADMIN/`) cuentan con documentación de su arquitectura de componentes, enrutamiento, integración de APIs y estilos.
- [ ] La guía de despliegue detalla los requisitos, variables de entorno y pasos de ejecución local y con Docker.
- [ ] Los encabezados markdown son limpios y directos, sin cápsulas ni overlines decorativos, en estricto cumplimiento de `AGENTS.md`.
- [ ] Todos los bloques Mermaid de diagramas de arquitectura y ER compilan y renderizan sin errores de sintaxis.
- [ ] Todos los enlaces relativos entre archivos markdown dentro de `docs/` resuelven a rutas existentes sin enlaces rotos.
