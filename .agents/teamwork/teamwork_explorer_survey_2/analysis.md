# Reporte de Exploración Técnica CTT-WEB — Ecosistema Empresarial & Auditoría Mobile-First

**Fecha:** 2026-11-08  
**Autor:** `teamwork_explorer_survey_2`  
**Repositorio inspeccionado:** `/home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-WEB`  
**Objetivo:** Mapear la arquitectura frontend, componentes, rutas, estados, adaptabilidad responsive y cumplimiento estricto de las directivas institucionales (`AGENTS.md`) para las Oleadas 1, 2 y 3 del plan de producción empresarial CTT-FISEI.

---

## 1. Enrutamiento y Navegación (`Routing and Navigation`)

### 1.1 Estructura en `App.jsx`
- **Ubicación:** `CTT-WEB/src/App.jsx:1-83`
- **Enrutador:** React Router 7 (`react-router-dom@^7.9.1`).
- **Control de Acceso:**
  - Componente de ruta protegida `PrivateRoute` (`App.jsx:32-34`):
    ```jsx
    const PrivateRoute = ({ children }) => {
      return isAuthenticated() ? children : <Navigate to="/login" replace />;
    };
    ```
  - `isAuthenticated()` (`src/utils/auth.js:5-25`) verifica existencia de `token` y tiempo de sesión menor a 8 horas (`8 * 60 * 60 * 1000`).
  - `useSessionExpiration()` (`src/hooks/useSessionExpiration.js`) monitorea el tiempo restante y cierra la sesión automáticamente con modal/timeout.
- **Rutas Públicas (`App.jsx:44-52`):**
  - `/login` → `<Login />`
  - `/register` → `<Register />`
  - `/` → `<Home />`
  - `/courses` → `<Courses />`
  - `/events` → `<Events />`
  - `/course/:id` → `<CourseDetail />`
  - `/congreso/:id` → `<CongressDetail />`
  - `/nosotros` → `<Nosotros />`
  - `/contact` → `<Contact />`
- **Rutas Privadas bajo `/dashboard` (`App.jsx:55-72`):**
  - Contenedor con Layout: `<Dashboard />` (`src/pages/application/Dashboard.jsx`)
  - Redirección por defecto: `/dashboard` → `/dashboard/cursos/disponibles`
  - `/dashboard/cursos/disponibles` → `<Cursos />`
  - `/dashboard/cursos/inscrito` → `<CursosInscritos />`
  - `/dashboard/congresos/disponibles` → `<CongresosDisponibles />`
  - `/dashboard/ordenes/pendientes` → `<Ordenes type="pendientes" />`
  - `/dashboard/ordenes/completadas` → `<Ordenes type="completadas" />`
  - `/dashboard/curso/:id` → `<CourseDetail />`
  - `/dashboard/congreso/:id` → `<CongressDetail />`
- **Hallazgo Crítico:**
  - **Falta la ruta de Perfil del Estudiante:** Actualmente no existe `<Route path="perfil" element={<Perfil />} />` ni el componente `Perfil.jsx`.
  - La redirección de comodín dentro de `/dashboard` envía cualquier ruta desconocida a `cursos/disponibles`.

### 1.2 Barras de Navegación y Menús
- **Layout del Dashboard (`Dashboard.jsx:91-195`):**
  - Sidebar flotante: `<Menu />` (`src/components/layout/Menu.jsx`)
  - Barra superior con breadcrumbs automáticos según `location.pathname` (`Dashboard.jsx:31-86`) y botón de alternancia de tema claro/oscuro con `useDashboardTheme`.
- **Menú Lateral (`Menu.jsx:45-92`):**
  - Agrupaciones:
    - *Home:* Página Principal (`/`)
    - *Área Académica:* Cursos Disponibles, Cursos Inscritos, Congresos Disponibles.
    - *Gestión y Pagos:* Órdenes Pendientes, Órdenes Completadas.
  - Dock Inferior de Usuario (`Menu.jsx:219-270`): Muestra iniciales del estudiante, correo y botón de cerrar sesión (`setShowLogoutConfirm`).
  - **Falta Enlace a Perfil:** No existe botón ni enlace a `Mi Perfil` / `/dashboard/perfil`.
- **Navegación Pública (`Nav.jsx:127-180`):**
  - En desktop, cuando el usuario está autenticado, muestra menú desplegable con botón *"Ir al Panel"* (`/dashboard/cursos/disponibles`) y *"Cerrar Sesión"*.
  - En móvil (`Nav.jsx:281-317`), muestra botón *"Ir a la plataforma"*.
  - Tampoco enlaza directamente al perfil del usuario.

---

## 2. Módulo de Inscripciones y Órdenes (`Inscriptions & Orders`)

### 2.1 `CursosInscritos.jsx` (`src/pages/application/CursosInscritos.jsx`)
- **Flujo de Carga:**
  - Consume `getUserEnrollments(userData.id)` desde `src/api/inscripciones.js:108-151`.
  - Endpoint backend invocado: `GET /api/v1/enrollments/user/{userId}`.
- **Pestañas y Filtrado:**
  - Pestañas con contadores numéricos calculados con `useMemo`: *Todos*, *Congresos*, *Cursos*, *Confirmados*, *Pendientes*.
  - Barra de búsqueda reactiva por título del programa o categoría.
  - Paginación local en bloques de 8 elementos por página.
- **Lógica de Acciones Contextuales (`CursosInscritos.jsx:442-504`):**
  - **Si es Congreso y estado es `PAGADO` o `FACTURADO`:** Renderiza botón *"Credencial"* con icono `QrCodeIcon`, abriendo `DigitalPassModal`.
  - **Si el estado NO es pagado (`INTERESADO` u `ORDEN_GENERADA`):**
    - Si ya subió comprobante (`payment_order_url` existe y no es `'local'`): Renderiza botón *"En Revisión"* (`ClockIcon`).
    - Si no ha subido comprobante: Renderiza botón *"Pagar / SRI"* (`DocumentArrowUpIcon`), abriendo `PaymentReceiptModal`.
  - **Si es Curso y está pagado:** Muestra badge *"Matriculado"*.
  - **Botón secundario:** *"Ver Detalle"* (`EyeIcon`) navegando a `/dashboard/curso/:id` o `/dashboard/congreso/:id`.
- **Deficiencias Detectadas:**
  - **Falta el botón de descarga del RIDE PDF:** No existe enlace o botón para descargar el comprobante oficial UTA RIDE con RUC `1860000880001` (Plan Oleada 2.1).
  - **Estructura fija en tabla HTML:** Usa `<table>` con anchos fijos porcentuales que colapsan en pantallas móviles (ver sección de auditoría responsive).

### 2.2 `Ordenes.jsx` (`src/pages/application/Ordenes.jsx`)
- **Propósito:** Divide las inscripciones en `pendientes` (`INTERESADO`, `ORDEN_GENERADA`) y `completadas` (`PAGADO`, `FACTURADO`).
- **Métricas:** 3 tarjetas KPI (*Total Órdenes*, *Estado del Módulo*, *Soporte Administrativo*).
- **Componente Tabla:** Usa el componente reutilizable `Table.jsx` (`src/components/ui/Table.jsx`).
- **Deficiencias Detectadas:**
  - En `Ordenes.jsx:75-99` existe una función `handleUploadReceipt` que intenta usar `setUploadingId`, variable de estado inexistente en este componente. Sin embargo, no es llamada por la interfaz (código muerto que abre `PaymentReceiptModal` en su lugar).
  - Al igual que en `CursosInscritos`, no existe opción de descarga del RIDE PDF para órdenes facturadas.

### 2.3 Modales de Apoyo
- **`DigitalPassModal.jsx` (`src/components/ui/DigitalPassModal.jsx`):**
  - Renderiza una credencial digital institucional con RUC UTA, facultad FISEI, sede Campus Huachi, código de pase `UTA-FISEI-PASS-{enrollment.id}-{enrollment.id_congress}`, categoría de arancel y código QR.
  - **Generación del QR:** Actualmente usa un servicio de imagen externo: `https://api.qrserver.com/v1/create-qr-code/?size=240x240&format=png&data=${qrData}`.
  - Incluye botón de impresión física (`window.print()`) con estilos específicos `@media print`.
- **`PaymentReceiptModal.jsx` (`src/components/ui/PaymentReceiptModal.jsx`):**
  - Muestra datos de la cuenta corriente institucional de la UTA en Banco del Pacífico (No. 7458129, RUC 1860000880001).
  - Formulario de facturación SRI con campos: `billing_identification`, `billing_name`, `billing_address`, `billing_phone`, `billing_email`.
  - Validador y subida de archivo (PDF, JPG, PNG, máx 5MB).
  - Actualiza datos de facturación invocando `updateUserBilling(userId, formData)` (`src/api/auth.js:121`) que hace `PUT /api/v1/users/{user_id}/billing`.
  - La subida del comprobante llama a `uploadPaymentReceiptTemp` (`src/api/inscripciones.js:188`), la cual actualmente apunta a `https://httpbin.org/post` en desarrollo.

---

## 3. Vistas de Congresos (`Congress Views & Capacity`)

### 3.1 `CongressDetail.jsx` (`src/pages/CongressDetail.jsx:1-854`)
- **Datos y Relaciones:**
  - Carga el congreso por ID numérico (`getCongressById`) obteniendo sesiones de cronograma, ponentes (speakers) y auspiciantes (sponsors).
  - Permite navegación fluida entre pestañas: *Visión General*, *Cronograma*, *Ponentes*, *Auspiciantes*.
- **Estado de Inscripción:**
  - Comprueba inscripción previa en `checkEnrollment` buscando `id_congress === id`.
  - Botón principal dinámico:
    - No inscrito: *"Inscribirme al Congreso"* → ejecuta `enrollInCongress(user.id, id)` y despliega `EnrollmentModal`.
    - Inscrito (Pendiente): *"Subir Comprobante y Datos SRI"* → abre `PaymentReceiptModal`.
    - Inscrito (Pagado): *"Ver Credencial de Acceso con QR"* → abre `DigitalPassModal`.
- **Hallazgo Crítico sobre Control de Aforo (`cupo_maximo`):**
  - En `CongressDetail.jsx:704-710` sólo se renderiza texto estático si `congress.cupo_maximo` existe:
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
  - **Falta implementación de Wave 1.1:**
    - No existe barra de progreso de aforo con porcentaje.
    - No calcula si `inscritos >= cupo_maximo`.
    - No muestra el badge rojo *"Convocatoria Completa / Cupos Agotados"*.
    - El botón *"Inscribirme al Congreso"* permanece habilitado aun si el aforo estuviese completo.
- **Faltantes de Wave 3 en `CongressDetail.jsx`:**
  - Pestaña *"Call for Papers"* y formulario de envío de papers en PDF.
  - Hub de transmisión en vivo (Zoom / Teams) accesible para inscritos pagados.
  - Encuesta de calidad académica CACES.

### 3.2 `CongresosDisponibles.jsx` (`src/pages/application/CongresosDisponibles.jsx:1-573`)
- **Opciones de Visualización:** Tarjetas (Cards) o Tabla (Table).
- **Filtros:** Por año (`availableYears`) y buscador de texto por nombre.
- **Control de Aforo:**
  - En la vista de tarjetas y tabla no se muestra el aforo disponible ni se bloquea el botón *"Inscribirme"* si el cupo está agotado.

---

## 4. Gestión de Perfil del Estudiante (`Profile Management`)

### 4.1 Estado Actual
- **Inexistencia del Componente:** No existe ningún archivo `Perfil.jsx` en `CTT-WEB/src/pages/application/` ni en ningún otro directorio.
- **Campos SRI en el Ecosistema:**
  - En el backend (`Back-CTT/src/models/user_platform.py:31-36`), el modelo `UserPlatform` ya cuenta con los 5 campos fiscales:
    - `billing_identification` (String 20)
    - `billing_name` (String 200)
    - `billing_address` (String 255)
    - `billing_phone` (String 20)
    - `billing_email` (EmailStr / String 320)
  - En `Back-CTT/src/routes/users_platform_router.py:21-59`, existe el endpoint `PUT /api/v1/users/{user_id}/billing` debidamente protegido por token.
  - En frontend, `src/api/auth.js:121` ya tiene la función `updateUserBilling(userId, billingData)`.

### 4.2 Requerimiento para Wave 1.3
Crear `CTT-WEB/src/pages/application/Perfil.jsx` con:
1. **Tabs horizontales deslizables:**
   - *Pestaña 1 (Datos Personales):* Nombres, Apellidos, Cédula (bloqueada en modo `disabled/readOnly` para evitar suplantación), Correo, Celular y Dirección.
   - *Pestaña 2 (Facturación Electrónica SRI):* Formulario de datos predeterminados para emisión ágil de facturas.
   - *Pestaña 3 (Seguridad):* Cambio de clave validando contraseña actual y nueva contraseña (backend con `bcrypt`).
   - *Pestaña 4 (Historial Financiero):* Lista de órdenes con estado y botón directo para descargar el RIDE PDF UTA.
2. **Rutas:** Registrar `<Route path="perfil" element={<Perfil />} />` en `App.jsx`.
3. **Navegación:** Agregar opción de acceso a *"Mi Perfil"* en `Menu.jsx` y en el menú móvil de `Nav.jsx`.

---

## 5. Auditoría Responsive y Mobile-First (`Mobile Audit`)

Siguiendo las directrices de `anti-ai-ui-design`: *"Responsive compuesto, no reducido. En móvil cambia jerarquía, densidad, navegación y disposición de acciones; no encoger una tabla ilegible"*.

### 5.1 Matriz de Viewports

| Breakpoint | Ancho Viewport | Estado Actual en CTT-WEB | Cumplimiento & Acciones Requeridas |
| :--- | :--- | :--- | :--- |
| **Móvil Vertical (`< 640px`)** | `375px` - `430px` (iPhone / Android) | ⚠️ **No conforme en tablas y modales.** `CursosInscritos.jsx` fuerza `<table>` con `overflow-x-auto` horizontal forzado. Botones de acción miden `h-8` (32px), por debajo del estándar táctil de 44px. Modales se centran en lugar de abrirse como Bottom Sheets. | **MANDATORIO:**<br>1. Transformar filas de `CursosInscritos` en **Tarjetas Estructuradas Apiladas** en pantallas `< sm`.<br>2. Ajustar botones de acción a altura táctil de `h-11` (44px).<br>3. Anclar modales a la base de la pantalla (`items-end sm:items-center`, `rounded-t-2xl sm:rounded-2xl`). |
| **Tablet / Horizontal (`640px - 1024px`)** | `sm` a `md` (iPad, Galaxy Tab) | ✅ **Conforme.** Grids de 2 columnas en tarjetas de congresos y formularios; barra lateral colapsable mediante `isCollapsed`. | Mantener layout de 2 columnas y tablas compactas con scroll contenido. |
| **Escritorio (`> 1024px`)** | `lg` a `2xl` (> 1024px) | ✅ **Conforme.** Layout completo de tabla con anchos proporcionales, sticky sidebar y navegación completa. | Mantener alta densidad de información limpia y contrastada. |

### 5.2 Detalle de Adaptabilidad en Tablas
- En `CursosInscritos.jsx:294-540`:
  - En `<sm`, el usuario tiene que desplazar horizontalmente una tabla de 5 columnas. Las etiquetas de los botones se ocultan con `hidden sm:inline`, dejando solo iconos pequeños de 14x14px dentro de botones de 32px de alto (`h-8 px-2.5`).
  - **Solución Mobile-First:**
    - Ocultar la tabla tradicional en `< 640px` (`hidden sm:block`).
    - Renderizar en `< 640px` (`block sm:hidden`) una lista de tarjetas limpias donde:
      - Encabezado: Título del curso/congreso + Badge semántico de estado a la derecha.
      - Metadatos: Tipo (Curso/Congreso), Fecha de registro y categoría en tipografía legible.
      - Barra de Acciones: Botones a ancho completo (`w-full h-11 text-sm`) con iconos claros y texto explícito (*"Descargar Credencial QR"*, *"Pagar / Subir Comprobante"*, *"Descargar RIDE"*).

### 5.3 Detalle de Adaptabilidad en Modales
- En `PaymentReceiptModal.jsx:185-202` y `DigitalPassModal.jsx:101-118`:
  - Están configurados como: `<div className="flex min-h-full items-center justify-center p-4">`.
  - En móvil, esto sitúa el diálogo en el centro de la pantalla, haciendo difícil alcanzar los campos superiores y el botón de cierre con una sola mano.
  - **Ajuste Bottom Sheet:**
    ```jsx
    <div className="flex min-h-full items-end sm:items-center justify-center p-0 sm:p-4">
      <Dialog.Panel className="w-full max-w-lg rounded-t-2xl sm:rounded-2xl ...">
    ```

---

## 6. Verificación de Reglas del Proyecto (`AGENTS.md`)

| Directiva en `AGENTS.md` | Estado en CTT-WEB | Evidencia / Observación |
| :--- | :---: | :--- |
| **1. Prohibición de Cápsulas y Eyebrow Badges Decorativos** | **100% CUMPLIDO** | Búsqueda exhaustiva de `SparklesIcon` no arrojó resultados. Encabezados `<h1>` y `<h2>` en `Home.jsx`, `Section.jsx`, `CourseSection.jsx`, `CoursesList.jsx`, `Nosotros.jsx`, `Events.jsx`, `CongressDetail.jsx`, `CursosInscritos.jsx` y `Ordenes.jsx` son directos, sin pastillas ni decoraciones superficiales sobre los títulos. |
| **2. Cero Desaparición de Elementos con GSAP** | **100% CUMPLIDO** | No existen animaciones con `toggleActions: "play reverse play reverse"` ni elementos animados hacia `opacity: 0` al hacer scroll inverso. |
| **3. Uso de `once: true` y `clearProps: "all"`** | **100% CUMPLIDO** | Verificado en `Section.jsx:19-23`, `Stats.jsx:25-29`, `CoursesList.jsx:171,188`. Todas las animaciones limpian propiedades CSS inline al completarse. |
| **4. Transición de Páginas (`ScrollToTop`)** | **100% CUMPLIDO** | `ScrollToTop.jsx` resetea inmediatamente el scroll con `window.scrollTo({ top: 0, left: 0, behavior: "instant" })` y refresca `ScrollTrigger.refresh()`. |
| **5. Identidad Institucional y Contraste** | **100% CUMPLIDO** | Azul institucional `#00345e` configurado como token primario en `index.css`. Sistema de tema dual (claro/oscuro) verificado con clases de Tailwind v4 y variables CSS de alto contraste. |

---

## 7. Configuración de Compilación y Dependencias (`Build Setup`)

### 7.1 Entorno de Node y Gestor de Paquetes
- **Ruta de binarios:** `/home/sebas/.nvm/versions/node/v26.13.0/bin/pnpm` y `/home/sebas/.nvm/versions/node/v26.13.0/bin/node` (Node.js v26.13.0).
- **Prueba de compilación de producción:**
  ```bash
  export PATH="/home/sebas/.nvm/versions/node/v26.13.0/bin:$PATH" && cd CTT-WEB && pnpm run build
  ```
  **Resultado:** `✓ built in 23.91s` sin errores. Bundle generado en `CTT-WEB/dist`.

### 7.2 Análisis de Dependencias en `package.json`
- **Frontend Core:**
  - `react`: `^19.1.1`
  - `react-dom`: `^19.1.1`
  - `react-router-dom`: `^7.9.1`
  - `vite`: `^7.1.2`
  - `@tailwindcss/vite`: `^4.1.13`
  - `tailwindcss`: `^4.1.13`
- **Animaciones e Iconos:**
  - `gsap`: `^3.15.1`
  - `@gsap/react`: `^2.1.2`
  - `framer-motion`: `^14.10.1`
  - `@headlessui/react`: `^2.2.8`
  - `@heroicons/react`: `^2.2.0`
  - `react-icons`: `^5.5.0`
- **Observaciones sobre librerías ausentes:**
  - `lucide-react`: **No está instalada** en `CTT-WEB`. Se utilizan exclusivamente `@heroicons/react` y `react-icons`. Toda nueva vista debe usar `@heroicons/react/24/outline` para mantener consistencia y evitar dependencias innecesarias.
  - Generador offline de QR: No hay librería local de QR (e.g. `qrcode.react`). El modal `DigitalPassModal` usa la API web de `api.qrserver.com`.
  - Librería PDF del cliente: No se requiere librería pesada de renderizado PDF en frontend (`jspdf`/`pdfmake`), ya que la arquitectura establece que el PDF oficial RIDE se genera de forma nativa en el backend (`Back-CTT`) y se descarga vía endpoint binario.

---

## 8. Matriz de Brechas & Hoja de Ruta de Implementación

| Requerimiento | Componentes Afectados | Brecha Detectada | Tareas para el Agente Implementador |
| :--- | :--- | :--- | :--- |
| **Control de Aforo (Wave 1.1)** | `CongressDetail.jsx`, `CongresosDisponibles.jsx` | No hay barra de porcentaje ni bloqueo si `inscritos >= cupo_maximo`. | Añadir cálculo de ocupación porcentual, barra de aforo, badge de cupos agotados y bloqueo del botón de inscripción. |
| **Perfil del Estudiante (Wave 1.3)** | `App.jsx`, `Menu.jsx`, `Nav.jsx`, `Perfil.jsx` (nuevo) | `Perfil.jsx` no existe. No hay rutas ni enlaces en menús. | Crear `Perfil.jsx` con 4 pestañas (Datos, SRI, Seguridad con bcrypt, Órdenes/RIDE), registrar ruta `/dashboard/perfil` y enlazar en menús. |
| **Descarga de RIDE PDF (Wave 2.1)** | `CursosInscritos.jsx`, `Ordenes.jsx`, `Perfil.jsx` | No existe botón de descarga de RIDE PDF en ninguna vista de estudiante. | Agregar botón de descarga táctil `GET /api/v1/enrollments/{id}/ride/download` para programas con estado `PAGADO`/`FACTURADO`. |
| **Transformación Tabla a Tarjetas (R4 Mobile)** | `CursosInscritos.jsx` | La tabla fuerza desplazamiento horizontal en celulares (`< 640px`). | Implementar vista condicional: Tarjetas estructuradas apiladas en `<sm`, tabla completa en `>=sm`. |
| **Modales tipo Bottom Sheet (R4 Mobile)** | `DigitalPassModal.jsx`, `PaymentReceiptModal.jsx` | Modales están centrados en móviles. | Ajustar posicionamiento en `<sm` con `items-end` y `rounded-t-2xl`. |
| **Dianas Táctiles >= 44px (R4 Mobile)** | `CursosInscritos.jsx`, modales | Botones de acción tienen altura `h-8` (32px). | Establecer altura mínima de `h-11` (44px) en botones interactivos móviles. |
| **Call for Papers & Streaming (Wave 3)** | `CongressDetail.jsx` | Faltan pestañas de CFP y Hub de streaming. | Implementar pestaña CFP para subida de paper científico y enlaces protegidos de Zoom/Teams. |
