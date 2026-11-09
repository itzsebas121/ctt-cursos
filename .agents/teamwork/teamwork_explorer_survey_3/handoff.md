# Handoff Report: CTT-ADMIN Architectural & Component Survey

**Agent**: `teamwork_explorer_survey_3`  
**Milestone**: Survey CTT-ADMIN architecture, components, routes, responsive styling, and project standards.  
**Destination Folder**: `/home/sebas/Workspace/Desarrollo/ctt-cursos/.agents/teamwork/teamwork_explorer_survey_3`  
**Timestamp**: 2026-11-08T03:59:30Z

---

## 1. Observation

1. **Rutas y Enrutador (`CTT-ADMIN/src/App.jsx:41-71`)**:
   - `App.jsx` define dos rutas principales: `/login` (pública) y `/dashboard/*` (protegida con `PrivateRoute`).
   - Las subrutas dentro de `/dashboard` abarcan cursos (`cursos`, `cursos/:id`, `cursos/:id/editar`, `cursos/:course_id/inscritos`), congresos (`congresos`, `categoriasCongresos`, `congresos/crear`, `congresos/:id`, `congresos/:id/editar`), estudiantes (`estudiantes`) y configuración (`configuracion/home`).
   - No hay rutas registradas para `/dashboard/finanzas`, `/dashboard/asistencia` ni `/dashboard/papers`.

2. **Autenticación e Interceptores (`CTT-ADMIN/src/utils/auth.js:8-27` y `CTT-ADMIN/src/api/api.js:12-21`)**:
   - El token JWT y `tokenTimestamp` se guardan en `localStorage`. La expiración está fijada en 8 horas (`EIGHT_HOURS = 8 * 60 * 60 * 1000`).
   - Axios inyecta automáticamente `Authorization: Bearer ${token}` mediante `api.interceptors.request.use`.
   - No existe un React Context global de sesión (`AuthContext`); las vistas leen directamente de `localStorage` con `isAuthenticated()` y `getUserData()`.

3. **Vista de Detalle de Congreso (`CTT-ADMIN/src/pages/congress/CongresosDetalle.jsx:39, 273-314`)**:
   - Pestañas actuales controladas por `activeTab`: `"general"`, `"cronograma"`, `"patrocinadores"`, `"inscritos"`.
   - En la pestaña `"cronograma"` (líneas 336-362), las sesiones se muestran como elementos de texto y badges informativos sin botones interactivos para iniciar escaneo ni revisar asistencia.
   - En la pestaña `"inscritos"` (líneas 398-486), se renderiza una tabla con participantes y un botón `"Revisar Pago"` que abre `ValidatePaymentModal`.
   - No existen pestañas ni secciones para Finanzas, LMS Moodle ni Call for Papers.

4. **Escáner de Asistencia y Dependencias QR**:
   - `find /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-ADMIN -name "*Attendance*" -o -name "*Scanner*"`: sin resultados. `AttendanceScannerModal.jsx` no existe.
   - En `CTT-ADMIN/package.json:12-27`: no están instaladas librerías de lectura QR de cámara (`@zxing/browser`, `html5-qrcode`).

5. **Métricas Financieras y LMS Moodle**:
   - Búsqueda en código `grep -rn -i "financial\|revenue\|moodle\|ride" src/`: 0 coincidencias.
   - No existen componentes ni tarjetas de desglose de aranceles ($20, $50, $80), balance contable, ni botón de descarga CSV para Moodle en CTT-ADMIN.

6. **Call for Papers (CFP)**:
   - Búsqueda en código `grep -rn -i "paper\|scientific\|cfp" src/`: solo coincidencias descriptivas de tarifa ponente en `CreateSpeakerModal.jsx:8` y `StepGeneralData.jsx:319`.
   - No existe interfaz para auditar ni aprobar/rechazar papers.

7. **Auditoría de Adaptabilidad y Media Queries**:
   - En `Table.jsx:295-372` y `CongresosDetalle.jsx:401-480`: las tablas están envueltas en `<div className="overflow-x-auto">`. En pantallas `< 640px`, no se transforman en tarjetas apiladas sino que fuerzan scroll horizontal.
   - En `ValidatePaymentModal.jsx:52-58` y `ConfirmDialog.jsx:56-57`: los modales usan `items-center justify-center p-4` fijos en lugar de desplegarse como Bottom Sheets en móvil (`items-end sm:items-center`, `rounded-t-2xl sm:rounded-xl`).
   - Múltiples botones de acción (`Table.jsx:403-442`, `ValidatePaymentModal.jsx:211-249`) tienen altura inferior a 44px (`h-8`, `py-1.5`).

8. **Cumplimiento de `AGENTS.md`**:
   - Prohibición de cápsulas/pill badges sobre títulos `<h2>`/`<h3>`: 0 infracciones en todo `CTT-ADMIN/src`. Encabezados limpios y directos.
   - En `StepDescriptionPoster.jsx:8`: `SparklesIcon` está importado pero no se utiliza ni renderiza.
   - `gsap`: Configurado con `ScrollTrigger` en `src/utils/gsapConfig.js`. Sin animaciones de salida a `opacity: 0`.
   - Azul institucional `#00345e`: Presente de forma coherente en todo el proyecto.

9. **Herramientas de Build y Compilación**:
   - Node `v26.13.0` y pnpm `v12.18.0` en `/home/sebas/.nvm/versions/node/v26.13.0/bin/`.
   - `export PATH="/home/sebas/.nvm/versions/node/v26.13.0/bin:$PATH" && cd CTT-ADMIN && pnpm run build`:
     `✓ built in 19.27s` sin errores.

---

## 2. Logic Chain

1. A partir de la observación (1) y (3), la arquitectura actual de `CTT-ADMIN` organiza la gestión de congresos en una vista de detalle unificada (`CongresosDetalle.jsx`) orientada a pestañas. Por ende, la integración de la Wave 1 (Control de Asistencia QR), Wave 2 (Dashboard Financiero y Moodle) y Wave 3 (Call for Papers) debe articularse directamente como extensiones tabulares dentro de `CongresosDetalle.jsx` sin quebrar el esquema existente de navegación.
2. A partir de la observación (4), dado que no existe `AttendanceScannerModal.jsx` ni librerías de escaneo, la implementación requerirá agregar `html5-qrcode` a las dependencias y situar el modal en `src/components/congress/AttendanceScannerModal.jsx`, permitiendo activación tanto desde cada fila del cronograma como desde un acceso directo en la cabecera del congreso.
3. A partir de las observaciones (5) y (6), la ausencia de finanzas y CFP implica que ambos módulos pueden desarrollarse con total libertad arquitectónica, siguiendo el diseño Dokploy/Tailwind v4 y las directivas Mobile-First del plan maestro (`plan_ecosistema_empresarial_completo_ctt.md`).
4. A partir de la observación (7), para cumplir el criterio de aceptación R4 de la orden inicial, se deben refactorizar las tablas de participantes y los modales (`ValidatePaymentModal`, `AttendanceScannerModal`, `ConfirmDialog`) para que en `< 640px` se presenten como tarjetas fluidas y Bottom Sheets anclados abajo con dianas táctiles `>= 44px`.
5. A partir de las observaciones (8) y (9), el código base de `CTT-ADMIN` está 100% sano en compilación y cumple estrictamente con las prohibiciones de `AGENTS.md`.

---

## 3. Caveats

- **Sin backend en vivo durante el survey**: Las respuestas de los endpoints fueron analizadas contra el código fuente y contratos OpenAPI/FastAPI de `Back-CTT`, sin tráfico HTTP de prueba.
- **Hardware de escáner USB**: La interacción con lectores USB de código de barras se basa en eventos de teclado del navegador estándar (`window.addEventListener('keydown')`); la calibración de tiempos de cadencia (<80ms) debe ser probada con dispositivos físicos de recepción.

---

## 4. Conclusion

El repositorio `CTT-ADMIN` cuenta con una base sólida basada en React 19, Tailwind v4 y Vite 7, con una estructura limpia de rutas y cumplimiento de directivas de estilo (`AGENTS.md`). Las funcionalidades empresariales planificadas (Asistencia QR con soporte dual móvil/USB, Dashboard Financiero con KPIs de aranceles, Exportador CSV Moodle, y Dictamen CFP) están completamente ausentes y listas para ser implementadas sin conflictos estructurales. La principal deuda técnica radica en la adaptación Mobile-First (transformación de tablas a tarjetas y modales a Bottom Sheets con dianas táctiles >= 44px).

---

## 5. Verification Method

Para verificar independientemente estos hallazgos:
1. **Comprobar la compilación limpia actual**:
   ```bash
   export PATH="/home/sebas/.nvm/versions/node/v26.13.0/bin:$PATH"
   cd /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-ADMIN
   pnpm run build
   ```
   *Condición de éxito*: Salida `✓ built in ~19s` con 0 errores de compilación.

2. **Verificar ausencia de AttendanceScannerModal**:
   ```bash
   find /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-ADMIN -name "*AttendanceScannerModal*"
   ```
   *Condición de éxito*: Salida vacía (0 resultados).

3. **Verificar pestañas en `CongresosDetalle.jsx`**:
   Inspeccionar línea 39 y líneas 273-314 de `CTT-ADMIN/src/pages/congress/CongresosDetalle.jsx` para confirmar las 4 pestañas actuales (`general`, `cronograma`, `patrocinadores`, `inscritos`).

4. **Verificar cumplimiento de `AGENTS.md` (cápsulas decorativas sobre títulos)**:
   ```bash
   grep -rn -B 2 -A 1 "<h[1-3]" /home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-ADMIN/src
   ```
   *Condición de éxito*: Ningún badge/pill colocado como overline sobre los títulos.
