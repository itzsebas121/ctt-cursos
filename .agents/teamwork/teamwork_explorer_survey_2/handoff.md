# Handoff Report — CTT-WEB Architecture & Mobile-First Survey

**Author:** `teamwork_explorer_survey_2`  
**Working Directory:** `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_2`  
**Date:** 2026-11-08  
**Type:** Hard Handoff (Investigation & Architecture Survey Complete)

---

## 1. Observation

1. **Routing and Structure (`App.jsx`):**
   - File: `CTT-WEB/src/App.jsx:55-72`.
   - The `<Dashboard />` route defines nested subroutes: `cursos/disponibles`, `cursos/inscrito`, `congresos/disponibles`, `ordenes/pendientes`, `ordenes/completadas`, `curso/:id`, `congreso/:id`.
   - Exact observation: Line 71 has `<Route path="*" element={<Navigate to="/dashboard/cursos/disponibles" replace />} />`. There is NO route for `perfil` (`<Route path="perfil" element={<Perfil />} />` does NOT exist).
   - In `CTT-WEB/src/pages/application/`, the directory contains: `Cursos.jsx`, `CursosInscritos.jsx`, `Dashboard.jsx`, `CongresosDisponibles.jsx`, `Ordenes.jsx`. `Perfil.jsx` is absent from the filesystem.

2. **Dashboard Navigation (`Menu.jsx`):**
   - File: `CTT-WEB/src/components/layout/Menu.jsx:45-92`.
   - `navGroups` contains items for Home, Área Académica, and Gestión y Pagos.
   - User account dock (`Menu.jsx:219-270`) renders user initials, user email, and logout button, but contains no link or navigation handler to `/dashboard/perfil`.

3. **Inscriptions and Action Cell Structure (`CursosInscritos.jsx`):**
   - File: `CTT-WEB/src/pages/application/CursosInscritos.jsx:286-540`.
   - Table is rendered inside `<div className="overflow-x-auto"><table className="w-full text-left border-collapse table-fixed">...</table></div>` for all viewport widths without mobile breakdown.
   - Action buttons in lines 449, 467, 480 have class `h-8 px-2.5` (height 32px) and hide text on mobile (`<span className="hidden sm:inline">...</span>`), displaying only 14x14px icons.
   - There is NO button or link to download official UTA RIDE payment receipt PDF.

4. **Congress Views & Capacity (`CongressDetail.jsx` & `CongresosDisponibles.jsx`):**
   - File: `CTT-WEB/src/pages/CongressDetail.jsx:704-711`:
     ```jsx
     {congress.cupo_maximo && (
       <div className="flex justify-between items-center">
         <span className="text-zinc-500 dark:text-zinc-400">Aforo Máximo:</span>
         <span className="font-semibold text-zinc-800 dark:text-zinc-200">
           {congress.cupo_maximo} participantes
         </span>
       </div>
     )}
     ```
   - No capacity percentage bar, no check comparing active enrollments against `cupo_maximo`, and button *"Inscribirme al Congreso"* (`CongressDetail.jsx:783-798`) has no disabled condition based on capacity.
   - `CongresosDisponibles.jsx:355-462` also lacks capacity percentage and quota lock.

5. **Modal Positioning (`DigitalPassModal.jsx` & `PaymentReceiptModal.jsx`):**
   - File: `CTT-WEB/src/components/ui/DigitalPassModal.jsx:101-118` and `PaymentReceiptModal.jsx:185-202`.
   - Both modals use `<div className="flex min-h-full items-center justify-center p-4 text-center">` and `<Dialog.Panel className="w-full max-w-lg ... rounded-2xl">`. They are center-aligned on all screen widths rather than anchored to the bottom as Bottom Sheets on `< 640px`.

6. **AGENTS.md Compliance Verification:**
   - Command: `grep -rn "SparklesIcon" CTT-WEB/src/` returned exit code 1 (zero occurrences).
   - Headings in `Section.jsx:39`, `CourseSection.jsx:96`, `CoursesList.jsx:199`, `CongressDetail.jsx:318`, `CursosInscritos.jsx:161`, `Ordenes.jsx:252` are clean typography without decorative pill badges or eyebrow capsules.
   - GSAP animations in `Section.jsx:13-25`, `Stats.jsx:19-31`, `CoursesList.jsx:165-189` all include `clearProps: "all"` and `scrollTrigger: { ..., once: true }`. No `toggleActions: "play reverse play reverse"` exists.
   - `ScrollToTop.jsx:13-17` sets `window.scrollTo({ top: 0, left: 0, behavior: "instant" })` and triggers `ScrollTrigger.refresh()`.

7. **Build and Dependencies (`package.json`):**
   - Command: `export PATH="/home/sebas/.nvm/versions/node/v26.13.0/bin:$PATH" && cd CTT-WEB && pnpm run build`.
   - Result: Exit code 0, `✓ built in 23.91s` (`dist/assets/index-CxOHn7_K.js` 1,126 kB).
   - `lucide-react` is not present in `package.json`; `@heroicons/react` (`^2.2.0`) is the standard icon library.

---

## 2. Logic Chain

1. **Premise:** The unified enterprise roadmap requires sequential delivery of Wave 1 (Operational Core: capacity enforcement + Student Profile), Wave 2 (Finance/RIDE PDF download), and Wave 3 (CFP, Hybrid Streaming, CACES Survey), under strict Mobile-First and AGENTS.md constraints.
2. **From Observation 1 & 2:** Since `Perfil.jsx` is completely absent from `CTT-WEB/src/pages/application/` and no route exists in `App.jsx`, Wave 1.3 requires implementing `Perfil.jsx` with 4 tabs (Personal Data, SRI Taxpayer Defaults, bcrypt Password Change, Financial History), registering `<Route path="perfil" element={<Perfil />} />` in `App.jsx`, and adding navigation entries in `Menu.jsx` and `Nav.jsx`.
3. **From Observation 4:** Since `CongressDetail.jsx` only prints raw text `{congress.cupo_maximo} participantes` without verifying enrolled count or locking registration, Wave 1.1 requires computing capacity percentage, displaying an aforo progress bar, showing a red badge *"Convocatoria Completa / Cupos Agotados"*, and disabling the registration button when `inscritos >= cupo_maximo`.
4. **From Observation 3 & 5:** Since `CursosInscritos.jsx` forces an HTML table with horizontal overflow on `< 640px`, and modal dialogs are centered instead of bottom-sheet-anchored, Requirement R4 is violated. Mobile viewports (< 640px) require rendering structured cards with 44px (`h-11`) touch targets, and modal panels require `items-end sm:items-center` positioning with `rounded-t-2xl sm:rounded-2xl`.
5. **From Observation 3 & Back-CTT Survey:** Since the backend endpoint `GET /api/v1/enrollments/{id}/ride/download` will serve the official UTA RIDE PDF, the frontend must add tactile *"Descargar RIDE / Comprobante PDF"* action buttons in `CursosInscritos.jsx`, `Ordenes.jsx`, and `Perfil.jsx`.
6. **From Observation 6 & 7:** Existing views strictly adhere to AGENTS.md (zero pill badges, GSAP `once: true` + `clearProps: 'all'`, institutional blue `#00345e`), and the production build compiles cleanly without errors. All newly added components must strictly follow these patterns and use `@heroicons/react`.

---

## 3. Caveats

- **Backend Sync:** The exact endpoint URL for password update in Back-CTT (`PUT /api/v1/platform-auth/change-password` vs `PUT /api/v1/users/{id}/password`) will be provided by the backend agent; frontend profile form is ready to bind to standard payload `{ current_password, new_password }`.
- **Receipt Upload in Production:** `uploadPaymentReceiptTemp` currently defaults to `https://httpbin.org/post` in `.env`. When the backend receipt endpoint is finalized, changing the target URL in `src/api/inscripciones.js` will complete real file storage.
- **QR Server:** `DigitalPassModal` currently renders QR images via `api.qrserver.com`. If offline campus rendering is desired later, a lightweight offline SVG component or canvas generator can be introduced, but current implementation renders immediately when connected.

---

## 4. Conclusion

CTT-WEB possesses a solid foundation: modern Vite + Tailwind v4 + React 19 architecture, clean GSAP animation lifecycle management, zero decorative pill badge violations, and institutional `#00345e` styling. However, it requires targeted enterprise additions:
1. **Creation of `Perfil.jsx`** with 4 tabs and route registration in `App.jsx` + `Menu.jsx`.
2. **Capacity meter & registration lockout** in `CongressDetail.jsx` and `CongresosDisponibles.jsx`.
3. **Mobile-First transformation of `CursosInscritos.jsx`** (stacked cards on `< 640px` and `h-11` touch targets).
4. **Bottom Sheet ergonomics** on `< 640px` for `PaymentReceiptModal.jsx` and `DigitalPassModal.jsx`.
5. **RIDE PDF download buttons** for paid/facturado enrollments.

---

## 5. Verification Method

To independently verify these findings:

1. **Verify Current Build Status:**
   ```bash
   export PATH="/home/sebas/.nvm/versions/node/v26.13.0/bin:$PATH"
   cd CTT-WEB && pnpm run build
   ```
   *Expected result:* Vite build succeeds in ~24s with zero compilation errors.

2. **Verify Route & Perfil Inexistence:**
   ```bash
   grep -rn "perfil" CTT-WEB/src/App.jsx CTT-WEB/src/components/layout/Menu.jsx
   ls -la CTT-WEB/src/pages/application/Perfil.jsx
   ```
   *Expected result:* No route or file exists.

3. **Verify Capacity Handling:**
   Inspect `CTT-WEB/src/pages/CongressDetail.jsx` lines 704-711 and 783-798.
   *Expected result:* No percentage bar, no enrollment count check, button not disabled by capacity.

4. **Verify Mobile Table Layout in `CursosInscritos`:**
   Inspect `CTT-WEB/src/pages/application/CursosInscritos.jsx` lines 293-335.
   *Expected result:* Standard HTML `<table>` wrapped in `overflow-x-auto` without `<sm` card alternative.

5. **Verify AGENTS.md Clean Headers:**
   ```bash
   grep -rn "SparklesIcon" CTT-WEB/src/
   ```
   *Expected result:* Empty (no AI decorative pills present).
