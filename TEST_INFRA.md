# E2E Test Infra: ctt_enterprise_platform

## Test Philosophy
- Opaque-box, requirement-driven. No dependency on implementation internals.
- Methodology: Category-Partition + BVA + Pairwise + Workload Testing.

## Feature Inventory
| # | Feature | Source (requirement) | Tier 1 | Tier 2 | Tier 3 |
|---|---------|---------------------|:------:|:------:|:------:|
| 1 | Capacity Control (`cupo_maximo`) | ORIGINAL_REQUEST §R1 | 5 | 5 | ✓ |
| 2 | QR Session Attendance Check-in | ORIGINAL_REQUEST §R1 | 5 | 5 | ✓ |
| 3 | Student Profile & SRI Defaults | ORIGINAL_REQUEST §R1 | 5 | 5 | ✓ |
| 4 | Official UTA RIDE PDF Generator | ORIGINAL_REQUEST §R2 | 5 | 5 | ✓ |
| 5 | Financial Reporting Dashboard | ORIGINAL_REQUEST §R2 | 5 | 5 | ✓ |
| 6 | Moodle LMS Exporter & API | ORIGINAL_REQUEST §R2 | 5 | 5 | ✓ |
| 7 | Asynchronous SMTP Email Engine | ORIGINAL_REQUEST §R2 | 5 | 5 | ✓ |
| 8 | Call for Papers (CFP) Submission & Review | ORIGINAL_REQUEST §R3 | 5 | 5 | ✓ |
| 9 | Hybrid Streaming Hub & Materials | ORIGINAL_REQUEST §R3 | 5 | 5 | ✓ |
| 10 | CACES Academic Quality Survey | ORIGINAL_REQUEST §R3 | 5 | 5 | ✓ |
| 11 | Mobile-First Responsive Breakpoints | ORIGINAL_REQUEST §R4 | 5 | 5 | ✓ |

## Test Architecture
- Backend Test Suite: `Back-CTT/tests/test_enterprise_features.py`
  - Command: `docker exec ctt-backend pytest tests/test_enterprise_features.py -v`
  - Verifies: Capacity limits, attendance scanning + duplicate 409 rejection, RIDE PDF generation with RUC 1860000880001, Moodle CSV export format, profile updates & bcrypt password change.
- Frontend Production Build Verification:
  - CTT-WEB: `cd /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-WEB && pnpm run build`
  - CTT-ADMIN: `cd /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-ADMIN && pnpm run build`
- Responsive Layout & AGENTS.md Auditing:
  - Static analysis of media queries `<640px` (card layout in `CursosInscritos.jsx`, bottom sheets for modals, `>= 44px` touch targets).
  - Clean typographic hierarchy (zero decorative pill badges or sparkles over headings).

## Real-World Application Scenarios (Tier 4)
| # | Scenario | Features Exercised | Complexity |
|---|----------|--------------------|------------|
| 1 | Full Student Lifecycle | F1, F3, F4, F7, F9, F10 | High |
| 2 | On-site Congress Session Check-in | F2, F7 | Medium |
| 3 | Post-Congress Accounting & LMS Sync | F4, F5, F6 | Medium |
| 4 | Scientific Paper Submission & Speaker Tariff | F8, F1, F4 | High |

## Coverage Thresholds
- Tier 1: ≥5 per feature
- Tier 2: ≥5 per feature (boundary and error rejection)
- Tier 3: Pairwise coverage of major feature interactions
- Tier 4: ≥4 realistic end-to-end application scenarios
