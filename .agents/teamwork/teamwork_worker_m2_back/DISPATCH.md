## 2026-11-08T04:42:26Z
You are a teamwork_worker implementing Wave 2 (Finance, RIDE, LMS & SMTP) Backend features in Back-CTT.
Your working directory is: /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_worker_m2_back

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
- `Back-CTT/src/utils/ride_generator.py` (create new)
- `Back-CTT/src/utils/email_service.py` (create new)
- `Back-CTT/src/routes/enrollments_router.py`
- `Back-CTT/src/routes/congress_router.py`

YOUR TASK:
1. Official UTA RIDE PDF Generator:
   - Create `Back-CTT/src/utils/ride_generator.py` using `reportlab` and `qrcode` (already installed in container):
     - Razón Social: "Universidad Técnica de Ambato - FISEI (CTT)"
     - RUC Institucional: "1860000880001"
     - Dirección Matriz: "Campus Huachi, Av. Los Chasquis y Río Payamino, Ambato"
     - Número de Comprobante: `001-002-{enrollment.id:09d}`
     - Desglose contable: Concepto del programa/congreso, Categoría arancelaria ($20 Estudiante UTA, $50 Profesional, $80 Ponente), Subtotal, IVA 0% (Educación exenta), Total.
     - Datos del usuario/SRI: `billing_identification` or `identification`, `billing_name` or student name, `billing_address`, etc.
     - Institutional header, styling (using #00345e blue), and embedded QR code containing receipt verification payload.
     - Generate as in-memory `io.BytesIO`.
   - In `Back-CTT/src/routes/enrollments_router.py`:
     - Implement `GET /api/v1/enrollments/{enrollment_id}/ride/download`:
       Validates enrollment exists and belongs to user (or admin).
       Verifies status is `PAGADO` or `FACTURADO` (or returns 400 if not paid).
       Calls `generate_ride_pdf(enrollment, user, congress, db)` and returns `StreamingResponse(pdf_buffer, media_type="application/pdf", headers={"Content-Disposition": f"attachment; filename=RIDE_UTA_{enrollment_id:09d}.pdf"})`.
2. Financial Reporting Endpoint:
   - In `Back-CTT/src/routes/congress_router.py`:
     - Implement `GET /api/v1/congresses/{id_congreso}/financial-summary`:
       Queries enrollments for `{id_congreso}`.
       Computes:
       - `total_revenue`: sum of `amount` for status in `[PAGADO, FACTURADO]`.
       - `total_budget`: congress budget or target revenue (default estimated from cupo_maximo * 50.0).
       - `tariffs_breakdown`: revenue and count grouped by tariff (`estudiante_uta` ($20), `profesional` ($50), `ponente` ($80)).
       - `conversion_rate`: percentage of `[PAGADO, FACTURADO]` out of total active enrollments (`[INTERESADO, ORDEN_GENERADA, PAGADO, FACTURADO]`).
       - Returns JSON: `{"total_revenue": ..., "total_budget": ..., "tariffs_breakdown": {"estudiante_uta": ..., "profesional": ..., "ponente": ...}, "conversion_rate": ...}`.
3. Moodle LMS Exporter:
   - In `Back-CTT/src/routes/congress_router.py`:
     - Implement `GET /api/v1/congresses/{id_congreso}/export/moodle-csv`:
       Queries all paid/facturado enrollments with student data.
       Generates CSV in memory with header: `username,firstname,lastname,email,course1,role1`.
       Mapping:
       - `username`: student `identification` (cédula).
       - `firstname`: student `first_name`.
       - `lastname`: student `first_last_name` + optional `second_last_name`.
       - `email`: student `email`.
       - `course1`: sanitized congress code or `f"CONG-{id_congreso}"`.
       - `role1`: "student" (or "editingteacher" if ponente).
       Returns `Response(content=csv_content, media_type="text/csv", headers={"Content-Disposition": f"attachment; filename=moodle_enrollments_congreso_{id_congreso}.csv"})`.
     - Optional helper `sync_moodle_users(...)` that checks `os.getenv("MOODLE_TOKEN")` and `os.getenv("MOODLE_URL")`, and calls `core_enrol_manual_enrol_users` via requests if configured, or logs notice if not configured.
4. Transactional SMTP Notification Engine:
   - Create `Back-CTT/src/utils/email_service.py`:
     - Asynchronous or threaded sending of HTML notification emails with institutional header and responsive design:
       1. `send_order_created_notification(enrollment, user, congress)`: Payment order details and UTA bank account instructions.
       2. `send_payment_approved_notification(enrollment, user, congress)`: Payment confirmed with digital pass QR code.
       3. `send_receipt_rejected_notification(enrollment, user, congress, reason)`: Rejection notice with reviewer notes.
       4. `send_reminder_24h_notification(enrollment, user, congress)`: 24-hour reminder with schedule and auditorium instructions.
     - Safe development fallback: If `SMTP_HOST` is not set or fails, log the notification to stdout/logger cleanly without raising exceptions or breaking API requests.

VERIFICATION:
- Copy tests if needed: `docker cp /home/sebas/Workspace/Desarrollo/ctt-cursos/Back-CTT/tests/test_enterprise_features.py ctt-backend:/app/tests/test_enterprise_features.py`
- Run `docker exec ctt-backend pytest tests/test_enterprise_features.py -v`.
- The RIDE PDF and Moodle CSV export tests must now PASS! Target: 17 passed, 0 failed!
- Document files modified and verification results in `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_worker_m2_back/handoff.md`.
- Send a message when complete.
