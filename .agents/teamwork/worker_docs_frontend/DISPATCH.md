## 2026-11-09T01:06:53Z
MANDATORY: Read `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/ORIGINAL_REQUEST.md`, `/home/sebas/Workspace/Desarrollo/ctt-cursos/AGENTS.md`, and `/home/sebas/Workspace/Desarrollo/ctt-cursos/PROJECT.md` before starting work.

MANDATORY INTEGRITY WARNING:
DO NOT CHEAT. All implementations must be genuine. DO NOT hardcode test results, create dummy/facade implementations, or circumvent the intended task. A teamwork_auditor will independently verify your work. Integrity violations WILL be detected and your work WILL be rejected.

Identity:
- Archetype: teamwork_worker
- Role: Frontend Architecture Specialist
- Working directory: /home/sebas/Workspace/Desarrollo/ctt-cursos
- State folder: /home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/worker_docs_frontend/

Scope & Write Ownership:
You exclusively own and will write the documentation files inside `docs/frontend-web/` and `docs/frontend-admin/`. DO NOT modify files in `docs/base-de-datos/`, `docs/backend/`, `docs/arquitectura/`, or `docs/despliegue/`.

Task Objective (Requirements R3, R5 & Acceptance Criteria):
Investigate the frontend applications in `CTT-WEB/` (React 19 + Vite + Tailwind CSS + GSAP, port 3000) and `CTT-ADMIN/` (React 19 + Vite + Tailwind CSS, port 3001).
Inspect `package.json`, `vite.config.js`, Tailwind configurations, routing, pages, components, services/api integrations, GSAP hooks/animations, and responsive layouts.
Generate comprehensive, highly detailed technical documentation in Spanish.

Required Files to Create:
1. `docs/frontend-web/README.md`:
   - Visión general del portal público y plataforma del estudiante CTT-WEB (React 19, Vite, Tailwind CSS, GSAP).
   - Estructura de carpetas `src/`, dependencias clave y guía de navegación de la documentación.
2. `docs/frontend-web/componentes-rutas.md`:
   - Arquitectura de componentes, layout institucional (Menu, Nav, Footer, ScrollToTop).
   - Árbol de rutas (React Router): rutas públicas (`/`, `/congresos`, `/congresos/:id`) y rutas privadas (`/dashboard`, `/dashboard/perfil`, `/dashboard/cursos-inscritos`, `/dashboard/ordenes`).
   - Flujo de autenticación, almacenamiento de tokens JWT y guardias de ruta.
3. `docs/frontend-web/catalogo-pagos.md`:
   - Catálogo unificado de congresos y cursos, vista de detalle e inscripción con control de cupo (`cupo_maximo`).
   - Flujo de órdenes de pago, estados de inscripción (`INTERESADO`, `ORDEN_GENERADA`, `PAGADO`, `FACTURADO`), carga de comprobantes.
   - Perfil completo del estudiante (`/dashboard/perfil`) con 4 pestañas: Datos personales, facturación SRI, cambio de contraseña bcrypt, e historial financiero.
   - Pase digital QR (`DigitalPassModal.jsx`) y descarga de recibo oficial RIDE UTA.
   - Módulos complementarios: Envío de artículos científicos (CFP) y encuesta de calidad académica CACES.
4. `docs/frontend-web/animaciones-gsap.md`:
   - Arquitectura de animaciones con GSAP y ScrollTrigger.
   - Reglas institucionales estrictas (AGENTS.md):
     * Cero desaparición de elementos en scroll inverso (prohibición estricta de `toggleActions: "play reverse play reverse"`).
     * Uso obligatorio de `clearProps: "all"` y `once: true` para persistencia total de contenido visible.
     * Reset de scroll al tope (`top: 0, left: 0`) en cambios de ruta con `ScrollToTop`.
   - Ejemplos y patrones de uso con el hook `useGSAP`.
5. `docs/frontend-web/diseno-responsive.md`:
   - Estrategia Mobile-First y directrices de media queries:
     * Dispositivos móviles (< 640px): transformación de tablas a tarjetas estructuradas apiladas (sin scroll horizontal desbordado), modales que abren como bottom sheets inferiores, touch targets mínimos de 44x44px.
     * Tablets (640px - 1024px): grids compactos de 2 columnas.
     * Escritorio (> 1024px): layouts expansivos de alta densidad de información.
   - Paleta cromática institucional (azul principal `#00345e`), tipografía y contraste en modo claro y oscuro.
6. `docs/frontend-admin/README.md`:
   - Visión general del panel administrativo CTT-ADMIN (React 19, Vite, Tailwind CSS, puerto 3001).
   - Estructura de carpetas `src/`, dependencias y guía de navegación de la sección.
7. `docs/frontend-admin/componentes-rutas.md`:
   - Arquitectura de componentes, layout del panel de administración (Sidebar, Header, Breadcrumbs).
   - Árbol de rutas protegidas para administradores y coordinadores de congreso.
   - Componentes comunes reutilizables: Tablas dinámicas, modales, alertas y botones de acción.
8. `docs/frontend-admin/gestion-academica-financiera.md`:
   - Módulo de administración de congresos y cronograma de sesiones.
   - Escáner de Asistencia QR a sesiones (`AttendanceScannerModal.jsx`): soporte dual para cámara trasera de smartphones y lectores de código de barras USB; lógica de validación y control de duplicados (409 Conflict).
   - Dashboard financiero: métricas clave de recaudación, ingresos desglosados por tarifas ($20, $50, $80), balance y tasa de conversión.
   - Exportador Moodle LMS: descarga en 1 clic de archivo CSV (`username,firstname,lastname,email,course1,role1`) para matriculación masiva.
9. `docs/frontend-admin/call-for-papers.md`:
   - Módulo de gestión y arbitraje de artículos científicos (Call for Papers - CFP).
   - Modal de revisión (`PaperReviewModal.jsx`) con estados de aprobación y notas para autores.
   - Asignación automática de tarifa Ponente ($80) al aprobar una ponencia.
