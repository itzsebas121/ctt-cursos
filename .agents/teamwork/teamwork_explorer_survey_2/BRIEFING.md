# BRIEFING — 2026-11-08T04:02:00Z

## Mission
Survey CTT-WEB repository to map frontend architecture, components, routes, modals, SRI taxpayer/billing flows, mobile adaptability, and AGENTS.md compliance for the CTT enterprise ecosystem plan.

## 🔒 My Identity
- Archetype: teamwork_explorer
- Roles: explorer, analyst
- Working directory: /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_2
- Original parent: 5173c970-207a-4e4b-805b-9e7e4e5b56af
- Milestone: survey_phase_frontend_survey_2

## 🔒 Key Constraints
- Read-only investigation — do NOT implement
- Strict prohibition of decorative pills/capsules over titles (AGENTS.md)
- GSAP animations must use once: true and clearProps: "all" (AGENTS.md)
- Zero element disappearance on scroll exit (AGENTS.md)
- Institutional blue #00345e
- Write outputs only to working directory

## Current Parent
- Conversation ID: 5173c970-207a-4e4b-805b-9e7e4e5b56af
- Updated: 2026-11-08T04:02:00Z

## Investigation State
- **Explored paths**:
  - `CTT-WEB/src/App.jsx`, `Dashboard.jsx`, `Menu.jsx`, `Nav.jsx`, `Header.jsx`, `ScrollToTop.jsx`
  - `CTT-WEB/src/pages/application/CursosInscritos.jsx`, `Ordenes.jsx`, `CongresosDisponibles.jsx`, `Cursos.jsx`
  - `CTT-WEB/src/pages/CongressDetail.jsx`, `CourseDetail.jsx`, `Home.jsx`, `Nosotros.jsx`, `Events.jsx`
  - `CTT-WEB/src/components/ui/DigitalPassModal.jsx`, `PaymentReceiptModal.jsx`, `EnrollmentModal.jsx`, `Table.jsx`, `AppleAlert.jsx`, `ConfirmDialog.jsx`
  - `CTT-WEB/src/api/auth.js`, `inscripciones.js`, `congresses.js`, `courses.js`, `api.js`
  - `CTT-WEB/package.json`, `vite.config.js`, `src/index.css`
  - `Back-CTT/src/routes/users_platform_router.py`, `models/user_platform.py`
- **Key findings**:
  1. `Perfil.jsx` is completely missing from CTT-WEB. No route exists in `App.jsx`, and no navigation exists in `Menu.jsx` or `Nav.jsx`.
  2. `CursosInscritos.jsx` renders an HTML `<table>` for all screens, causing horizontal scrolling on smartphones (<640px). Action buttons have 32px height instead of min 44px touch targets.
  3. `CongressDetail.jsx` and `CongresosDisponibles.jsx` do not compute capacity percentage, show no progress bar, have no "Cupos Agotados" badge, and do not disable registration when full.
  4. `DigitalPassModal` and `PaymentReceiptModal` are center-aligned on all viewports instead of opening as ergonomic Bottom Sheets on mobile (<640px).
  5. RIDE PDF download buttons are absent from `CursosInscritos.jsx` and `Ordenes.jsx`.
  6. AGENTS.md compliance is 100% satisfied: zero pill badges or SparklesIcon, GSAP animations use `once: true` and `clearProps: "all"`, `#00345e` institutional color used throughout.
  7. Production build (`pnpm run build`) succeeds cleanly in 23.91s.
- **Unexplored areas**: None. All 7 focus areas fully mapped.

## Key Decisions Made
- Fully documented findings and implementation roadmap in `analysis.md` and `handoff.md`.

## Artifact Index
- `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_2/analysis.md` — Comprehensive Frontend Survey Analysis
- `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_2/handoff.md` — 5-Component Handoff Report
- `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_2/progress.md` — Liveness & task checklist
- `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_2/DISPATCH.md` — Incoming task dispatch record
