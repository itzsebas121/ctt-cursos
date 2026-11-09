## 2026-11-08T04:08:14Z
You are a teamwork_worker implementing Wave 1 (Operational Core & Mobile-First) Frontend features in CTT-WEB.
Your working directory is: /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_worker_m1_web

MANDATORY INPUTS:
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/ORIGINAL_REQUEST.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/PROJECT.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/TEST_INFRA.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_2/handoff.md
- Read /home/sebas/.gemini/antigravity/brain/e4725dea-975f-4023-949a-7ead631c99a2/plan_ecosistema_empresarial_completo_ctt.md
- Review project rules in /home/sebas/Workspace/Desarrollo/ctt-cursos/AGENTS.md (STRICT PROHIBITION of decorative pill badges/sparkles over headings, clean hierarchy, GSAP clearProps:"all" & once:true, institutional blue #00345e, touch targets >= 44px, mobile bottom sheets).

MANDATORY INTEGRITY WARNING:
DO NOT CHEAT. All implementations must be genuine. DO NOT hardcode test results, create dummy/facade implementations, or circumvent the intended task. A teamwork_auditor will independently verify your work. Integrity violations WILL be detected and your work WILL be rejected.

EXCLUSIVE WRITE OWNERSHIP:
You own and may ONLY edit files inside `CTT-WEB/src/`:
- `CTT-WEB/src/pages/application/Perfil.jsx` (create new)
- `CTT-WEB/src/App.jsx`
- `CTT-WEB/src/components/layout/Menu.jsx`
- `CTT-WEB/src/pages/CongressDetail.jsx`
- `CTT-WEB/src/pages/CongresosDisponibles.jsx`
- `CTT-WEB/src/pages/application/CursosInscritos.jsx`
- `CTT-WEB/src/components/ui/DigitalPassModal.jsx`
- `CTT-WEB/src/components/ui/PaymentReceiptModal.jsx`

YOUR TASK:
1. Student Profile (`/dashboard/perfil`):
   - Create `CTT-WEB/src/pages/application/Perfil.jsx`:
     - Clean layout without decorative eyebrow pills or SparklesIcon over headings.
     - 4 Tabs:
       1. "Datos Personales": First name, Last name, Identification (read-only / locked against impersonation), Email, Phone, Address. Saves via `PUT /api/v1/users/{user_id}/profile`.
       2. "Facturación SRI": Identification/RUC, Business Name (Razón Social), Fiscal Address, Phone, Billing Email. Saves via `PUT /api/v1/users/{user_id}/billing`.
       3. "Seguridad": Change password form (Current password, New password, Confirm new password). Validates match and submits to `PUT /api/v1/users/{user_id}/password`.
       4. "Historial Financiero": Summary list/cards of user's payment orders, amounts, statuses, and links/buttons.
     - Mobile responsive: Tabs scroll horizontally with smooth styling; forms render 1 column on `<sm` (`space-y-4`) and 2 columns on `>=sm` (`sm:grid-cols-2`).
     - Touch targets >= 44px (`h-11`).
   - Register `<Route path="perfil" element={<Perfil />} />` in `CTT-WEB/src/App.jsx`.
   - Add "Mi Perfil" navigation link in `CTT-WEB/src/components/layout/Menu.jsx` (under account dock and navigation groups).
2. Capacity Meter & Quota Lock:
   - In `CongressDetail.jsx` and `CongresosDisponibles.jsx`:
     - Display capacity progress bar with percentage (`inscritos / cupo_maximo * 100`).
     - If `inscritos >= cupo_maximo`: Render semantic badge "Convocatoria Completa / Cupos Agotados" and DISABLE the registration button ("Inscribirme al Congreso") with tooltip/notice.
3. Mobile-First Transformations (<640px):
   - In `CursosInscritos.jsx`: Transform the desktop HTML table into stacked structured cards on `<sm` (`< 640px`), with program title, semantic badges, and full-width action buttons (>=44px touch target) without horizontal scroll overflow. On `>=sm` keep compact table.
   - In `DigitalPassModal.jsx` and `PaymentReceiptModal.jsx`: Update modal dialogs so that on `< 640px` they anchor as Bottom Sheets (`items-end sm:items-center`, `rounded-t-2xl sm:rounded-2xl`).

VERIFICATION:
- Verify build: `export PATH="/home/sebas/.nvm/versions/node/v26.13.0/bin:$PATH" && cd /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-WEB && pnpm run build`.
- Must succeed with 0 errors.
- Document all modified files and verification results in `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_worker_m1_web/handoff.md`.
- Send a message when complete.
