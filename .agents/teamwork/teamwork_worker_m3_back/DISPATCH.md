## 2026-11-08T05:05:08Z
You are a teamwork_worker implementing Wave 3 (Academic Innovation & Quality: CFP, Streaming, CACES) Backend features in Back-CTT.
Your working directory is: /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_worker_m3_back

MANDATORY INPUTS:
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/ORIGINAL_REQUEST.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/PROJECT.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/TEST_INFRA.md
- Read /home/sebas/.gemini/antigravity/brain/e4725dea-975f-4023-949a-7ead631c99a2/plan_ecosistema_empresarial_completo_ctt.md
- Review project rules in /home/sebas/Workspace/Desarrollo/ctt-cursos/AGENTS.md

MANDATORY INTEGRITY WARNING:
DO NOT CHEAT. All implementations must be genuine. DO NOT hardcode test results, create dummy/facade implementations, or circumvent the intended task. A teamwork_auditor will independently verify your work. Integrity violations WILL be detected and your work WILL be rejected.

EXCLUSIVE WRITE OWNERSHIP:
You own and may ONLY edit files inside `Back-CTT/src/`:
- `Back-CTT/src/models/scientific_paper_model.py` (create new)
- `Back-CTT/src/models/caces_survey_model.py` (create new)
- `Back-CTT/src/models/__init__.py`
- `Back-CTT/src/routes/congress_router.py`

YOUR TASK:
1. Call for Papers (CFP):
   - Create `ScientificPaper` SQLModel in `Back-CTT/src/models/scientific_paper_model.py`:
     Fields: `id: Optional[int] = Field(default=None, primary_key=True)`, `id_congress: int = Field(foreign_key="congresos.id_congreso")`, `id_user_platform: int = Field(foreign_key="users_platform.id")`, `title: str = Field(max_length=300)`, `abstract: str`, `track: str = Field(max_length=100)`, `keywords: Optional[str] = None, max_length=255`, `file_url: str = Field(max_length=500)`, `status: str = Field(default="ENVIADO", max_length=50)` (ENVIADO, EN_REVISION, APROBADO, RECHAZADO), `reviewer_comments: Optional[str] = None`, `submitted_at: datetime = Field(default_factory=datetime.utcnow)`, `updated_at: datetime = Field(default_factory=datetime.utcnow)`.
   - Ensure table `scientific_papers` exists in PostgreSQL `ctt_db` (execute DDL if needed).
   - In `Back-CTT/src/routes/congress_router.py`:
     - `POST /api/v1/congresses/{id_congreso}/papers`: Submit paper proposal (validates user and congress).
     - `GET /api/v1/congresses/{id_congreso}/papers`: List papers for congress with author names and details.
     - `PUT /api/v1/congresses/{id_congreso}/papers/{paper_id}/review`: Review endpoint (`status`, `reviewer_comments`). When `status == "APROBADO"`, find or update the author's enrollment in that congress to `tariff_type = "PONENTE"` and `amount = 80.0`.
2. Hybrid Streaming Hub:
   - In `Back-CTT/src/routes/congress_router.py`:
     - `GET /api/v1/congresses/{id_congreso}/streaming`:
       Verifies current user has enrollment with status `PAGADO` or `FACTURADO` for this congress. If not, returns HTTP 403 Forbidden with detail `"Acceso exclusivo para participantes con inscripción confirmada (PAGADO/FACTURADO)"`.
       If authorized, returns streaming URL (e.g. congress zoom/teams link) and downloadable materials list.
3. CACES Academic Quality Survey:
   - Create `CacesSurvey` SQLModel in `Back-CTT/src/models/caces_survey_model.py`:
     Fields: `id: Optional[int] = Field(default=None, primary_key=True)`, `id_congress: int = Field(foreign_key="congresos.id_congreso")`, `id_user_platform: int = Field(foreign_key="users_platform.id")`, `q1_rating: int`, `q2_rating: int`, `q3_rating: int` (all 1 to 5), `comments: Optional[str] = None`, `submitted_at: datetime = Field(default_factory=datetime.utcnow)`.
   - Ensure table `caces_surveys` exists in PostgreSQL `ctt_db`.
   - In `Back-CTT/src/routes/congress_router.py`:
     - `POST /api/v1/congresses/{id_congreso}/survey`: Submits survey ratings (validates 1 <= q <= 5).
     - `GET /api/v1/congresses/{id_congreso}/survey/stats`: Returns average ratings for q1, q2, q3, and total responses.

VERIFICATION:
- Run `docker exec ctt-backend pytest tests/test_enterprise_features.py -v`.
- Ensure all tests continue to pass and no regressions occur.
- Document modified files and verification results in `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_worker_m3_back/handoff.md`.
- Send a message when complete.
