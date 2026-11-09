## 2026-11-08T04:08:24Z
You are a teamwork_worker implementing Wave 1 (Operational Core & Mobile-First) Admin features in CTT-ADMIN.
Your working directory is: /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_worker_m1_admin

MANDATORY INPUTS:
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/ORIGINAL_REQUEST.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/PROJECT.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/TEST_INFRA.md
- Read /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_3/handoff.md
- Read /home/sebas/.gemini/antigravity/brain/e4725dea-975f-4023-949a-7ead631c99a2/plan_ecosistema_empresarial_completo_ctt.md
- Review project rules in /home/sebas/Workspace/Desarrollo/ctt-cursos/AGENTS.md (STRICT PROHIBITION of decorative pill badges/sparkles over headings, clean hierarchy, GSAP clearProps:"all" & once:true, institutional blue #00345e, touch targets >= 44px, mobile bottom sheets).

MANDATORY INTEGRITY WARNING:
DO NOT CHEAT. All implementations must be genuine. DO NOT hardcode test results, create dummy/facade implementations, or circumvent the intended task. A teamwork_auditor will independently verify your work. Integrity violations WILL be detected and your work WILL be rejected.

EXCLUSIVE WRITE OWNERSHIP:
You own and may ONLY edit files inside `CTT-ADMIN/`:
- `CTT-ADMIN/src/components/congress/AttendanceScannerModal.jsx` (create new)
- `CTT-ADMIN/src/pages/congress/CongresosDetalle.jsx`
- `CTT-ADMIN/package.json` / dependency files if installing `html5-qrcode` or QR camera tools

YOUR TASK:
1. QR Session Attendance Scanner Modal (`AttendanceScannerModal.jsx`):
   - Create `CTT-ADMIN/src/components/congress/AttendanceScannerModal.jsx`:
     - Dual Mode:
       a) Smartphone / Webcam Mode: Uses rear camera (`facingMode: "environment"`) with torch/flashlight button (if hardware supports) and auto-focus. You may install `html5-qrcode` via pnpm if needed, or use native WebRTC MediaDevices/BarcodeDetector.
       b) USB Barcode / 2D Scanner Mode: Captures global/window keyboard buffer ending with Enter key (<80ms cadence between keystrokes) so reception staff with physical handheld scanners can scan immediately without focusing an input.
     - Audio & Haptic Feedback: Play a success chime / error tone (via Web Audio API `AudioContext` so no external audio files are required) and `navigator.vibrate([100, 50, 100])` on error or `navigator.vibrate(100)` on success.
     - Post scan to backend: `POST /api/v1/congresses/{id_congreso}/attendance/scan` with `{"qr_payload": code, "id_sesion": selectedSessionId}`.
     - Handles 200 OK (shows green card: attendee name, tariff, total sessions, with timestamp) and 409 Conflict (shows red warning: "Asistencia ya registrada previamente").
     - Mobile responsive: Full-width / Bottom Sheet layout on `< 640px` (`items-end sm:items-center`, `rounded-t-2xl sm:rounded-2xl`).
2. Integration in `CongresosDetalle.jsx`:
   - In `CongresosDetalle.jsx` under the `cronograma` tab (and/or header action bar):
     Add an interactive button "Escanear Asistencia" next to each session in the schedule.
     Clicking opens `AttendanceScannerModal` pre-selected with that session ID and title.
     Display live attendee count badge if available.

VERIFICATION:
- Verify build: `export PATH="/home/sebas/.nvm/versions/node/v26.13.0/bin:$PATH" && cd /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-ADMIN && pnpm run build`.
- Must succeed with 0 errors.
- Document all modified files and verification results in `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_worker_m1_admin/handoff.md`.
- Send a message when complete.
