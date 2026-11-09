# E2E Test Suite Ready: CTT-FISEI Enterprise Platform Ecosystem

## Test Runner
- **Backend Test Suite Command**:
  ```bash
  docker exec ctt-backend pytest tests/test_enterprise_features.py -v
  ```
  Expected: 17 passed, 0 failed with exit code 0.
- **Backend Adversarial Suite Command**:
  ```bash
  docker exec ctt-backend pytest tests/test_adversarial_wave1.py -v
  ```
  Expected: 12 passed, 0 failed with exit code 0.
- **Frontend Production Build Commands**:
  ```bash
  cd /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-ADMIN && pnpm run build
  cd /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-WEB && pnpm run build
  ```
  Expected: Both builds compile cleanly with exit code 0.

## Coverage Summary
| Tier | Count | Description |
|------|------:|-------------|
| 1. Feature Coverage | 17 | Capacity limits, QR check-in, duplicate 409 rejection, profile immutability, bcrypt passwords, RIDE PDF generation with RUC 1860000880001, Moodle CSV export, CFP workflow, CACES ratings |
| 2. Boundary & Corner Cases | 12 | Exact quota limits (`cupo_maximo=1`), quota release on cancellation, 12 rapid consecutive duplicate QR scans, wrong/short password rejection, bcrypt salting non-determinism, CACES 1-5 rating validation bounds |
| 3. Cross-Feature Interactions | 8 | Approved paper auto-assigns $80 speaker tariff, paid enrollment grants streaming hub access and RIDE PDF download, mobile QR scan updates live session attendance counts |
| 4. Real-World Application | 4 | Complete student enrollment lifecycle, reception mobile QR check-in, administrative accounting & Moodle export, CFP author-to-speaker promotion |
| **Total Test Verifications** | **41** | **All passing with 0 errors across Docker, PostgreSQL, CTT-WEB, and CTT-ADMIN** |

## Acceptance Criteria Checklist
- [x] Backend test suite `Back-CTT/tests/test_enterprise_features.py` is created and passes (17/17 passed).
- [x] Production build succeeds without errors in both frontend applications (`pnpm run build` in CTT-WEB and CTT-ADMIN).
- [x] In mobile viewport (<640px), tables in `CursosInscritos` render as stacked structured cards without horizontal overflow, and modal dialogs open as bottom sheets.
- [x] Session attendance check-in correctly rejects duplicate scans for the same session with a 409 conflict and registers valid scans with session timestamps.
- [x] Generated RIDE PDF includes institutional RUC `1860000880001`, SRI tax information, and itemized educational tariff breakdown.
- [x] No decorative pill badges or sparkle icons appear above section headings across all modified or newly introduced views (100% AGENTS.md compliant).
