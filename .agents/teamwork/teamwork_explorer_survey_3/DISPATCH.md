## 2026-11-08T03:55:06Z
You are a read-only exploration agent (teamwork_explorer).
Your working directory is: /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_3

MANDATORY INPUTS:
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/ORIGINAL_REQUEST.md
- Read /home/sebas/.gemini/antigravity/brain/e4725dea-975f-4023-949a-7ead631c99a2/plan_ecosistema_empresarial_completo_ctt.md
- Review project rules in /home/sebas/Workspace/Desarrollo/ctt-cursos/AGENTS.md (Strict prohibition of decorative pills/capsules over titles, GSAP once:true & clearProps:"all", institutional blue #00345e)

YOUR MISSION:
Survey the CTT-ADMIN repository (/home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-ADMIN) to map the admin architecture, components, routes, and responsive styling:
1. Routing and Navigation: App.jsx, sidebar/navbar, layout, protected routes, auth context.
2. Congress Detail view: CongresosDetalle.jsx, existing tabs (sessions, participants, finances), components.
3. Attendance tracking UI: How attendance or sessions are currently managed, whether AttendanceScannerModal.jsx exists or where it should be placed, camera QR scanning libraries (@zxing/browser, html5-qrcode, etc.), barcode scanner USB input capture.
4. Financial Dashboard & Reports: Current financial metrics, cards, charts, export buttons (Moodle CSV, RIDE).
5. Call for Papers (CFP) review interface: How proposals/papers are listed and approved/rejected in admin.
6. Mobile & Responsive Audit: Media queries, touch targets (>=44px), modal bottom sheets on <640px, layout on smartphones and tablets.
7. Check for any AGENTS.md violations currently present (pill badges/sparkles over headers, GSAP issues).
8. Build setup: Vite/pnpm config, dependencies.

DO NOT modify any code.
Write your detailed findings to:
- /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_3/analysis.md
- /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_3/handoff.md

When complete, send a message to the orchestrator summarizing your findings and referencing the artifact paths.
