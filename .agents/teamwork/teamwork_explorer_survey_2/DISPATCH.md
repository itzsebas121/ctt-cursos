## 2026-11-08T03:54:59Z
You are a read-only exploration agent (teamwork_explorer).
Your working directory is: /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_2

MANDATORY INPUTS:
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/ORIGINAL_REQUEST.md
- Read /home/sebas/.gemini/antigravity/brain/e4725dea-975f-4023-949a-7ead631c99a2/plan_ecosistema_empresarial_completo_ctt.md
- Review project rules in /home/sebas/Workspace/Desarrollo/ctt-cursos/AGENTS.md (Strict prohibition of decorative pills/capsules over titles, GSAP once:true & clearProps:"all", institutional blue #00345e)

YOUR MISSION:
Survey the CTT-WEB repository (/home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-WEB) to map the frontend architecture, components, routes, and responsive styling:
1. Routing and Navigation: App.jsx, router structure, protected routes, user context/auth.
2. Inscriptions and Orders: CursosInscritos.jsx, Ordenes.jsx, digital pass modals, receipt upload modals.
3. Congress Views: CongressDetail.jsx, CongresosDisponibles.jsx, how cupo / capacity is displayed, registration flow.
4. Profile management: Current /dashboard/perfil or user settings page, forms, SRI taxpayer fields.
5. Mobile & Responsive Audit: Media queries, viewport adaptability (<640px vs >=640px), table structures in CursosInscritos, dialogs/bottom sheets, touch targets.
6. Check for any AGENTS.md violations currently present (pill badges/sparkles over headers, GSAP issues).
7. Build setup: Vite/pnpm config, dependencies (lucide-react, qr scanner, pdf libraries, etc.).

DO NOT modify any code.
Write your detailed findings to:
- /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_2/analysis.md
- /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_2/handoff.md

When complete, send a message to the orchestrator summarizing your findings and referencing the artifact paths.
