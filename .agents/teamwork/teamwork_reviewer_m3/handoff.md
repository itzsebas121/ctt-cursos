# Handoff Report — Wave 3 & Mobile-First (R4) Comprehensive Review & Adversarial Challenge

**Agent**: `teamwork_reviewer_m3`  
**Timestamp**: 2026-11-08T05:31:00Z  
**Parent Agent ID**: `5173c970-207a-4e4b-805b-9e7e4e5b56af`  
**Verdict**: `REQUEST_CHANGES`

---

## 1. Observation

### 1.1 Automated Build & Test Executions
1. **Backend Integration Test Suite**:
   - Command: `docker exec ctt-backend pytest tests/test_enterprise_features.py -v`
   - Result: 17 passed, 67 warnings in 44.73s (Exit code 0).
   - Scope covered in file: Wave 1 (Capacity, Session Attendance QR, Profile/Password bcrypt) and Wave 2 (RIDE PDF, Moodle CSV). Wave 3 endpoints (CFP, Streaming, CACES) are not yet integrated as test classes in `test_enterprise_features.py`.

2. **Frontend Production Builds**:
   - `CTT-ADMIN`: `pnpm run build` completed with code `0` in 21.08s (`dist/assets/index-CwpypDwR.js` 1,212.32 kB, `dist/assets/index-CBuDtfEr.css` 165.06 kB).
   - `CTT-WEB`: `pnpm run build` completed with code `0` in 25.16s (`dist/assets/index-D7kYnuZK.js` 1,203.05 kB, `dist/assets/index-BbVZQvY8.css` 119.29 kB).

### 1.2 Direct Code Observations & Critical Failures
1. **Critical Contract Mismatch in CACES Survey Submission**:
   - In `CTT-WEB/src/components/ui/SurveyModal.jsx` (lines 55–61):
     ```javascript
     const payload = {
       q1_score: Number(q1),
       q2_score: Number(q2),
       q3_score: Number(q3),
       comments: comments.trim(),
       id_user_platform: user?.id || null,
     };
     ```
   - In `Back-CTT/src/models/caces_survey_model.py` (lines 26–32):
     ```python
     class CacesSurveyCreate(BaseModel):
         q1_rating: int
         q2_rating: int
         q3_rating: int
         comments: Optional[str] = None
         id_user_platform: Optional[int] = None
     ```
   - Verbatim error reproduced via `TestClient`:
     `HTTP 422 Unprocessable Entity`:
     `{'detail': [{'type': 'missing', 'loc': ['body', 'q1_rating'], 'msg': 'Field required'}, {'type': 'missing', 'loc': ['body', 'q2_rating'], 'msg': 'Field required'}, {'type': 'missing', 'loc': ['body', 'q3_rating'], 'msg': 'Field required'}]}`.

2. **Critical Security Vulnerability — Missing Authentication on Paper Review**:
   - In `Back-CTT/src/routes/congress_router.py` (lines 1154–1168):
     ```python
     @congress_router.put(
         "/{id_congreso}/papers/{paper_id}/review",
         summary="Dictaminar y calificar paper científico",
         ...
     )
     def review_scientific_paper(
         id_congreso: int,
         paper_id: int,
         payload: ScientificPaperReview,
         db: SessionDep,
     ):
     ```
   - Verifiable test: executed unauthenticated PUT request from `TestClient`:
     `client.put('/api/v1/congresses/15/papers/1/review', json={'status': 'APROBADO', 'reviewer_comments': 'Hacked without auth'})`
     Result: HTTP 200 OK. The endpoint updated the paper status to `APROBADO`, updated reviewer comments, and automatically altered the user enrollment to `tariff_type="PONENTE"` and `amount=80.0` without validating any credentials or authorization tokens.

3. **Privacy Leakage — Unauthenticated Paper Author Directory**:
   - In `Back-CTT/src/routes/congress_router.py` (lines 1098–1151):
     `def list_congress_papers(id_congreso: int, db: SessionDep)`
     Has no authentication check. Returns `author_name`, `author_email`, and `author_identification` (cédula) for all submitted papers to any unauthenticated caller.

4. **Frontend Architecture Shortcut — Unconsumed Streaming Hub Endpoint**:
   - In `Back-CTT/src/routes/congress_router.py` (lines 1242–1320), `GET /api/v1/congresses/{id_congreso}/streaming` enforces 403 Forbidden for unpaid users and returns streaming URL, access code, and academic materials.
   - In `CTT-WEB/src/pages/CongressDetail.jsx` (line 973) and `CTT-WEB/src/pages/application/CursosInscritos.jsx` (line 568, 1068):
     The client hardcodes `href="https://cedia.zoom.us/j/uta-fisei-congreso-2026"` and `congress.streaming_url` without calling the `GET /streaming` endpoint.

5. **Mobile-First & UI Standards Compliance (R4)**:
   - `< 640px` Cards: `CursosInscritos.jsx` implements `sm:hidden` structured stacked cards without horizontal table scrolling; desktop renders clean tabular view (`hidden sm:block`).
   - Bottom Sheets: Modals (`PaperReviewModal.jsx`, `SurveyModal.jsx`, `DigitalPassModal.jsx`, `PaymentReceiptModal.jsx`, `AttendanceScannerModal.jsx`) correctly anchor as bottom sheets on smartphones with `items-end sm:items-center`, `rounded-t-2xl sm:rounded-2xl`, and visual pull-handles.
   - Touch targets: All action buttons and star ratings meet or exceed `min-h-[44px]` (or `h-11`).
   - AGENTS.md compliance: Zero decorative pill badges, sparkle icons, or overlines above headings (`h1`, `h2`, `h3`). Institutional color `#00345e` consistently applied.

---

## 2. Logic Chain

1. **Impact of CACES Field Mismatch**:
   - Observation: `SurveyModal.jsx` passes `q1_score`, `q2_score`, `q3_score`, while `CacesSurveyCreate` strictly expects `q1_rating`, `q2_rating`, `q3_rating`.
   - Inference: FastAPI schema validation rejects every legitimate submission from the web interface with HTTP 422. Because `SurveyModal` is the sole entry point for student evaluation, the CACES quality survey feature is currently non-functional end-to-end for students.

2. **Impact of Missing Authentication on CFP Review**:
   - Observation: `submit_scientific_paper`, `get_congress_streaming_hub`, and `submit_caces_survey` all use `Depends(get_current_user_any_type)`. However, `review_scientific_paper` omitted this dependency.
   - Inference: Reviewing a paper has direct financial and academic consequences (assigns `tariff_type="PONENTE"` and modifies payment order amount to $80). Leaving this endpoint unauthenticated allows any malicious user or crawler to auto-approve their own papers, acquire speaker discounts without review, or maliciously reject other participants' work.

3. **Impact of Unauthenticated Author Listing**:
   - Observation: `list_congress_papers` exposes author identification (cédula), email, and full name without requiring authentication.
   - Inference: Violates Ecuadorian personal data protection regulations (LOPDP). Anyone can scrape researcher and student contact and identification data.

4. **Impact of Frontend Streaming Bypass**:
   - Observation: The frontend uses static fallback links rather than querying `GET /api/v1/congresses/{id}/streaming`.
   - Inference: Bypasses dynamic meeting room credentials, material links, and server-side authorization validation, decoupling frontend behavior from backend business logic.

---

## 3. Caveats

- Pre-existing Wave 1 and Wave 2 integration tests (17 tests) continue to pass 100%. The regressions and security gaps identified are contained within the newly introduced Wave 3 endpoints and modals.
- Responsive styling (`<640px`), bottom sheets, touch targets, and AGENTS.md anti-pill compliance are well-implemented across both frontend applications.
- Production builds (`vite build`) succeed without compilation errors in both CTT-WEB and CTT-ADMIN.

---

## 4. Conclusion & Findings

### Verdict: **REQUEST_CHANGES**

### Critical Findings (Must Fix)

#### 1. [Critical] Contract Mismatch in CACES Survey (`SurveyModal.jsx` ↔ `caces_survey_model.py`)
- **What**: Payload field names differ between frontend (`q1_score`, `q2_score`, `q3_score`) and backend schema (`q1_rating`, `q2_rating`, `q3_rating`), triggering HTTP 422 Unprocessable Entity.
- **Where**: `CTT-WEB/src/components/ui/SurveyModal.jsx`: lines 55–61; `Back-CTT/src/models/caces_survey_model.py`: lines 26–32.
- **Why**: Completely breaks survey submission for students in production.
- **Suggestion**:
  - In `SurveyModal.jsx`, send `q1_rating`, `q2_rating`, `q3_rating`.
  - In `CacesSurveyCreate`, support aliases or optional `q1_score` fallback fields so both formats are accepted.

#### 2. [Critical - Adversarial Security] Unauthenticated Paper Review & Financial Tariff Mutation
- **What**: `PUT /api/v1/congresses/{id_congreso}/papers/{paper_id}/review` lacks authentication and authorization checks.
- **Where**: `Back-CTT/src/routes/congress_router.py`: line 1154.
- **Why**: Allows any unauthenticated attacker to approve papers, inject reviewer comments, and automatically assign the $80 Ponente tariff to any user.
- **Suggestion**: Add `user_data: tuple[Union[User, UserPlatform], str] = Depends(get_current_user_any_type)` and enforce `if user_type != "admin": raise HTTPException(403, "Acceso exclusivo para administradores y comité editorial")`.

### Major Findings (Should Fix)

#### 3. [Major - Privacy] Unauthenticated Paper Directory Exposing Cédulas and Emails
- **What**: `GET /api/v1/congresses/{id_congreso}/papers` is publicly accessible without authentication and returns authors' national identification (cédula) and personal email.
- **Where**: `Back-CTT/src/routes/congress_router.py`: line 1098.
- **Why**: Exposes protected personal data under Ecuador's LOPDP.
- **Suggestion**: Require authentication (`Depends(get_current_user_any_type)`) and restrict identification numbers to administrators or the author themselves.

#### 4. [Major - Architecture] Frontend Streaming Hub Bypasses Backend Protected Endpoint
- **What**: Neither `CongressDetail.jsx` nor `CursosInscritos.jsx` invokes `GET /api/v1/congresses/{id}/streaming`.
- **Where**: `CTT-WEB/src/pages/CongressDetail.jsx`: line 973; `CTT-WEB/src/pages/application/CursosInscritos.jsx`: lines 568, 1068.
- **Why**: Misses dynamic Zoom/Teams credentials and server-side enrollment status verification.
- **Suggestion**: Connect the frontend components to fetch `/api/v1/congresses/{id}/streaming` upon tab activation or modal load.

#### 5. [Minor] Automated Pytest Suite Lacks Wave 3 Test Classes
- **What**: `Back-CTT/tests/test_enterprise_features.py` only tests Waves 1 & 2. Wave 3 tests were only run in a manual script.
- **Where**: `Back-CTT/tests/test_enterprise_features.py`.
- **Suggestion**: Add classes `TestCallForPapers`, `TestStreamingHub`, and `TestCacesSurvey` to the test suite.

---

## 5. Verification Method

To reproduce and verify these findings independently:

1. **Verify CACES Survey 422 Rejection**:
   ```bash
   docker exec ctt-backend python3 -c "
   from fastapi.testclient import TestClient
   from fastapi import FastAPI
   from src.routes.congress_router import congress_router
   from src.routes.enrollments_router import get_current_user_any_type
   from src.models.user_platform import UserPlatform

   app = FastAPI()
   app.include_router(congress_router)
   u = UserPlatform(id=1, identification='1800000000', email='test@uta.edu.ec')
   app.dependency_overrides[get_current_user_any_type] = lambda: (u, 'platform')
   client = TestClient(app)

   # Payload sent by SurveyModal.jsx
   res = client.post('/api/v1/congresses/1/survey', json={
       'q1_score': 5, 'q2_score': 5, 'q3_score': 5, 'comments': 'Test', 'id_user_platform': 1
   })
   print('HTTP Status:', res.status_code)
   print('Detail:', res.json())
   assert res.status_code == 422
   print('VERIFIED: SurveyModal payload is rejected with 422')
   "
   ```

2. **Verify Unauthenticated Paper Review Vulnerability**:
   ```bash
   docker exec ctt-backend python3 -c "
   from fastapi.testclient import TestClient
   from fastapi import FastAPI
   from src.routes.congress_router import congress_router

   app = FastAPI()
   app.include_router(congress_router)
   client = TestClient(app)

   res = client.put('/api/v1/congresses/15/papers/1/review', json={
       'status': 'APROBADO', 'reviewer_comments': 'Exploit without token'
   })
   print('HTTP Status:', res.status_code)
   assert res.status_code == 200
   print('VERIFIED VULNERABILITY: Endpoint allows unauthenticated paper approval and tariff modification')
   "
   ```

3. **Verify Builds & Baseline Pytest**:
   ```bash
   docker exec ctt-backend pytest tests/test_enterprise_features.py -v
   export PATH="/home/sebas/.nvm/versions/node/v26.13.0/bin:$PATH"
   cd CTT-ADMIN && pnpm run build
   cd ../CTT-WEB && pnpm run build
   ```
