## 2026-11-08T05:05:23Z
You are a teamwork_worker implementing Wave 3 (Academic Innovation & Quality: CFP, Streaming, CACES) Frontend features in CTT-ADMIN and CTT-WEB.
Your working directory is: /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_worker_m3_frontend

MANDATORY INPUTS:
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/ORIGINAL_REQUEST.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/PROJECT.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/TEST_INFRA.md
- Read /home/sebas/.gemini/antigravity/brain/e4725dea-975f-4023-949a-7ead631c99a2/plan_ecosistema_empresarial_completo_ctt.md
- Review project rules in /home/sebas/Workspace/Desarrollo/ctt-cursos/AGENTS.md (STRICT PROHIBITION of decorative pill badges/sparkles over headings, clean hierarchy, GSAP clearProps:"all" & once:true, institutional blue #00345e, touch targets >= 44px).

MANDATORY INTEGRITY WARNING:
DO NOT CHEAT. All implementations must be genuine. DO NOT hardcode test results, create dummy/facade implementations, or circumvent the intended task. A teamwork_auditor will independently verify your work. Integrity violations WILL be detected and your work WILL be rejected.

EXCLUSIVE WRITE OWNERSHIP:
You own and may ONLY edit:
- `CTT-WEB/src/pages/CongressDetail.jsx`
- `CTT-WEB/src/pages/application/CursosInscritos.jsx`
- `CTT-WEB/src/components/ui/SurveyModal.jsx` (create new)
- `CTT-ADMIN/src/pages/congress/CongresosDetalle.jsx`
- `CTT-ADMIN/src/components/congress/PaperReviewModal.jsx` (create new)

YOUR TASK:
1. Call for Papers (CFP):
   - In `CTT-WEB/src/pages/CongressDetail.jsx`:
     - Add tab or modal for "Call for Papers / Enviar Artículo Científico": form with Title, Abstract, Track, Keywords, PDF URL/File.
     - Submits to `POST /api/v1/congresses/{id}/papers`.
     - Mobile friendly, single column on `<sm`.
   - In `CTT-ADMIN/src/pages/congress/CongresosDetalle.jsx`:
     - Add tab "papers" ("Artículos Científicos / CFP"):
       Table/cards listing submitted papers with author, title, track, status badge (`ENVIADO`, `EN_REVISION`, `APROBADO`, `RECHAZADO`).
       Action button "Dictaminar Paper" opening `PaperReviewModal.jsx`.
   - Create `CTT-ADMIN/src/components/congress/PaperReviewModal.jsx`:
     - Allows admin/reviewer to set status (`APROBADO` or `RECHAZADO`), write reviewer comments, and displays notice: "Al aprobarse, el autor recibirá automáticamente la tarifa preferencial de Ponente ($80)".
     - Calls `PUT /api/v1/congresses/{id}/papers/{paper_id}/review`.
     - Bottom Sheet layout on `< 640px`.
2. Hybrid Streaming Hub:
   - In `CTT-WEB/src/pages/CongressDetail.jsx` and `CursosInscritos.jsx`:
     - For paid/facturado enrollments, render the "Hub de Transmisión Híbrida y Materiales":
       Zoom/Teams direct access link, auditorium live status, and academic material downloads.
     - Add a floating mobile-accessible action button "Transmisión en Vivo" (visible on mobile when user is enrolled & paid).
3. CACES Academic Quality Survey:
   - Create `CTT-WEB/src/components/ui/SurveyModal.jsx`:
     - 3-question Likert questionnaire (1 to 5 stars) evaluating:
       Q1: Calidad de los ponentes y contenidos académicos.
       Q2: Organización, puntualidad y logística.
       Q3: Aplicabilidad profesional de los conocimientos adquiridos.
     - Star rating buttons must have tactile target >= 44x44px.
     - Comments textarea and submit button.
     - Submits to `POST /api/v1/congresses/{id}/survey`.
     - Can be opened from `CursosInscritos.jsx` or `CongressDetail.jsx` with a button "Evaluar Congreso (CACES)".
     - Bottom Sheet layout on mobile `< 640px`.


## 2026-11-08T05:20:28Z
**Context**: Wave 3 Frontend Implementation in CTT-ADMIN and CTT-WEB
**Content**: Heartbeat check: The backend worker has delivered all Wave 3 endpoints (CFP, Streaming, CACES) and tests are passing.
**Action**: Please report your progress on the CFP tabs, streaming hub buttons, SurveyModal, and production builds in CTT-ADMIN and CTT-WEB.
