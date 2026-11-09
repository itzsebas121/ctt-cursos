# Análisis Arquitectónico y Auditoría Técnica: CTT-ADMIN

**Fecha**: 2026-11-08  
**Agente**: `teamwork_explorer_survey_3`  
**Integridad**: Desarrollo / Auditoría Read-Only  
**Repositorio auditado**: `/home/sebas/Workspace/Desarrollo/ctt-cursos/CTT-ADMIN`

---

## 1. Arquitectura de Rutas y Navegación

### 1.1 Estructura de Enrutamiento (`src/App.jsx`)
- **Librería**: `react-router-dom` v7.9.1.
- **Flujo de Rutas**:
  - `/login`: Ruta pública de autenticación con `Login.jsx`.
  - `/dashboard/*`: Contenedor privado envuelto en `<PrivateRoute>`, que renderiza `Dashboard.jsx` y su jerarquía de subrutas vía `<Outlet />`.
  - Redirección comodín `*`: Redirige cualquier ruta no reconocida hacia `/login`.
- **Subrutas Registradas**:
  - `/dashboard/cursos` (`Cursos.jsx`)
  - `/dashboard/cursos/:id` (`CourseDetail.jsx`)
  - `/dashboard/cursos/:id/editar` (`FormCourse.jsx`)
  - `/dashboard/cursos/:course_id/inscritos` (`Inscritos.jsx`)
  - `/dashboard/estudiantes` (`Estudiantes.jsx`)
  - `/dashboard/categoriasCursos` (`CategoriasCursos.jsx`)
  - `/dashboard/configuracion/home` (`ConfiguracionHome.jsx`)
  - `/dashboard/congresos` (`Congresos.jsx`)
  - `/dashboard/categoriasCongresos` (`CategoriasCongresos.jsx`)
  - `/dashboard/congresos/crear` (`FormularioCongreso.jsx`)
  - `/dashboard/congresos/:id` (`CongresosDetalle.jsx`)
  - `/dashboard/congresos/:id/editar` (`FormularioCongreso.jsx`)

### 1.2 Mecanismo de Autenticación y Contexto (`src/utils/auth.js`)
- **Almacenamiento**: `localStorage` (`token`, `tokenTimestamp`).
- **Expiración de Sesión**: 8 horas (28,800,000 ms) validado en tiempo de ejecución.
- **Decodificación**: Utiliza `jwt-decode` (v4.0.0) para extraer datos básicos (`username`, `email`, rol).
- **Gestión de Estado**: No existe un React Context centralizado para autenticación (`AuthContext`); las funciones utilitarias se invocan directamente (`isAuthenticated()`, `getUserData()`, `clearAuthData()`, `getAuthHeader()`).
- **Inyección en API (`src/api/api.js`)**: Axios interceptor inyecta automáticamente el token en `headers.Authorization: Bearer <token>`.

### 1.3 Layout y Navegación Responsive (`src/pages/Dashboard.jsx` y `src/components/layout/Menu.jsx`)
- **Layout Contenedor**: `Dashboard.jsx` proporciona un contenedor flotante con esquinas redondeadas (`rounded-2xl`) y conmutador de tema claro/oscuro soportado por `DashboardThemeContext`.
- **Breadcrumbs**: Se generan de forma reactiva inspeccionando `location.pathname`.
- **Sidebar (`Menu.jsx`)**:
  - Ancho de escritorio: `w-64` (expandido) o `w-[72px]` (colapsado).
  - Responsive móvil (`< 1024px`): La barra se oculta fuera de pantalla (`-translate-x-full lg:translate-x-0`).
  - Botón flotante móvil: Ubicado en `fixed bottom-4 right-4 z-50 p-3 rounded-full lg:hidden`, activa un drawer con backdrop oscurecido (`bg-black/75` en modo oscuro, `bg-black/40` en modo claro).

---

## 2. Vista de Detalle de Congreso (`src/pages/congress/CongresosDetalle.jsx`)

### 2.1 Carga de Datos y Estado
- **Ruta**: `/dashboard/congresos/:id`
- **Llamadas concurrentes al montar**:
  - `getCongressById(id)`: Datos generales, afiche y fechas.
  - `getCongressSessions(id)`: Lista de sesiones del cronograma (`sessions`).
  - `getCongressSponsorships(id)`: Lista de auspiciantes (`sponsors`).
  - `getCongressStatistics(id)`: Estadísticas de inscritos (`stats`).
  - `getCongressEnrolledUsers(id, 1, 50)`: Inscripciones con estado (`enrollments`).

### 2.2 Hero Header y Pestañas Actuales
- **Hero**:
  - Afiche oficial (`poster_cover_url`) con fallback para imágenes ausentes.
  - Badges de edición y año.
  - Fechas de inauguración y clausura.
  - Contadores rápidos: `Sesiones` (`sessions.length`), `Auspiciantes` (`sponsors.length`), `Inscritos` (`stats?.total_inscriptions`).
- **Pestañas Existentes (`activeTab`)**:
  1. `general`: Muestra "Resumen Académico" (`congress.descripcion_general`).
  2. `cronograma`: Lista tarjetas de sesiones académicas (título, lugar, jornada, fecha y horas).
  3. `patrocinadores`: Grid con empresas auspiciantes, categorías y aportes económicos.
  4. `inscritos`: Tabla de estudiantes y participantes inscritos, con badges de estado y botón *"Revisar Pago"* que activa `ValidatePaymentModal`.

### 2.3 Brechas Identificadas frente al Plan Maestro
1. **Ausencia de Pestaña Financiera**: No existe desglose de recaudación por aranceles ($20 UTA, $50 Profesional, $80 Ponente), ni balance presupuestario.
2. **Cronograma Pasivo**: Las sesiones en el cronograma son de solo lectura; no permiten iniciar escaneo de asistencia QR ni ver la lista de asistentes por sesión.
3. **Ausencia de Call for Papers (CFP)**: No hay visualización de artículos enviados ni acciones de dictamen (Aprobar/Rechazar).
4. **Ausencia de Exportación Moodle**: No hay enlace ni botón para la descarga masiva de CSV de Moodle.

---

## 3. Auditoría del Módulo de Control de Asistencia QR

### 3.1 Estado de `AttendanceScannerModal.jsx`
- **Hallazgo**: El archivo `AttendanceScannerModal.jsx` **NO EXISTE** actualmente en el repositorio.
- **Ubicación propuesta**: `src/components/congress/AttendanceScannerModal.jsx` (y exportado en `src/components/congress/index.js`).

### 3.2 Librerías de Escaneo de Cámara QR
- **Dependencias actuales (`package.json`)**: Ninguna librería de escaneo (`@zxing/browser`, `html5-qrcode`, etc.) está instalada.
- **Evaluación técnica de librerías**:
  - `html5-qrcode`: Recomendada. Soporta escaneo directo en navegadores móviles (iOS Safari, Android Chrome), selector de cámara trasera (`facingMode: "environment"`), control de linterna (torch) y feedback de lectura rápido.
  - `@zxing/browser`: Alternativa robusta pero con mayor boilerplate para gestionar streams de video HTML5 en React 19.

### 3.3 Soporte de Pistolas Lectoras USB (Barcode / QR HID)
- Los lectores USB actúan como dispositivos de teclado HID, emitiendo ráfagas de caracteres seguidas de una tecla `Enter`.
- **Implementación requerida en el modal**:
  - Buffer de captura de pulsaciones (`keydown`) con detector de cadencia (< 80ms entre teclas).
  - Al recibir `Enter`, validar el formato del payload (ej. `UTA-FISEI-PASS-{enrollment_id}-{congress_id}`).
  - Despachar petición inmediata a `POST /api/v1/congresses/{id_congreso}/attendance/scan`.

### 3.4 Ergonomía Móvil y Feedback
- En smartphone (`< 640px`):
  - El modal debe expandirse a pantalla completa o Bottom Sheet alto (`h-[90vh]`).
  - Botón para alternar cámara trasera/frontal.
  - Feedback sonoro mediante Web Audio API sintetizado (`AudioContext` con tono de 880Hz / 200ms) y vibración háptica (`navigator.vibrate([100, 50, 100])`).
  - Alerta inmediata en caso de duplicado (`409 Conflict`), mostrando la hora exacta del registro anterior.

---

## 4. Auditoría de Finanzas y Reportes

### 4.1 Estado Actual
- No hay ningún cálculo financiero en `CTT-ADMIN`.
- Las únicas métricas consumidas son conteos numéricos simples de inscripciones.

### 4.2 Requerimientos de Implementación (Wave 2)
- **Consumo de Endpoint**: `GET /api/v1/congresses/{id}/financial-summary`.
- **Diseño de Tarjetas KPI Fluidas**:
  - Total Recaudado (USD) vs. Meta Presupuestaria.
  - Desglose por categoría arancelaria:
    - Estudiante UTA ($20.00)
    - Profesional / General ($50.00)
    - Ponente Aprobado ($80.00)
  - Tasa de Conversión: Interesados vs. Órdenes vs. Pagados/Facturados.
  - Adaptabilidad de cuadrícula: 1 columna en móvil (`< 640px`), 2 columnas en tablet (`640px - 1024px`), 4 columnas en desktop (`> 1024px`).
- **Botones de Exportación**:
  - Botón masivo *"Descargar CSV Moodle"* que apunte a `GET /api/v1/congresses/{id}/export/moodle-csv`.
  - Enlaces de auditoría para comprobantes RIDE oficiales.

---

## 5. Auditoría de Call for Papers (CFP) en Administración

### 5.1 Estado Actual
- No existe componente ni pestaña de gestión de artículos científicos en `CongresosDetalle.jsx`.
- Solo existen referencias textuales menores al rol *"Ponente Paper"* en `CreateSpeakerModal.jsx` y `StepGeneralData.jsx`.

### 5.2 Requerimientos de Implementación (Wave 3)
- **Pestaña en `CongresosDetalle.jsx`**: *"Call for Papers"* o *"Artículos Científicos"*.
- **Listado de Propuestas**:
  - Datos: Título del paper, track/área temática, autor principal (usuario plataforma), palabras clave, fecha de envío, estado actual (`ENVIADO`, `APROBADO`, `RECHAZADO`).
  - Visor/Descarga de PDF del artículo (`file_url`).
  - Modal o controles de dictamen:
    - Botón *"Aprobar Artículo"*: Asigna tarifa Ponente ($80) y actualiza estado a `APROBADO`.
    - Botón *"Rechazar Artículo"*: Permite ingresar observaciones y comentarios del comité evaluador (`reviewer_comments`).

---

## 6. Auditoría Mobile-First y Adaptabilidad Responsive

| Criterio | Estado Actual en CTT-ADMIN | Severidad | Acción Requerida |
| :--- | :--- | :--- | :--- |
| **Menú y Navegación Móvil** | Drawer colapsable con botón flotante en `fixed bottom-4 right-4` y backdrop. | Cumple | Mantener estructura y accesibilidad. |
| **Tablas en Móvil (`< 640px`)** | `Table.jsx` y tabla de `CongresosDetalle.jsx` usan `<table>` dentro de `overflow-x-auto`. Forzan scroll horizontal eterno. | Media | Implementar vista condicional o adaptativa que renderice tarjetas estructuradas en móvil (`hidden sm:table` vs `block sm:hidden`). |
| **Dianas Táctiles (Touch Targets)** | Botones de paginación (`h-8`), botones de acciones en tablas (`py-1` o `py-1.5`, altura efectiva ~28-32px). | Media | Asegurar dianas de mínimo `44x44px` en móvil (`min-h-[44px]` o padding aumentado en pantallas táctiles). |
| **Modales como Bottom Sheets** | `ValidatePaymentModal.jsx` y `ConfirmDialog.jsx` son modales centrados (`items-center justify-center p-4`). | Media | Adaptar a Bottom Sheet en `< 640px`: `items-end sm:items-center p-0 sm:p-4`, con bordes redondeados superiores `rounded-t-2xl sm:rounded-xl`. |
| **Soporte de Modo Oscuro en Diálogos** | `ConfirmDialog.jsx` utiliza fondos duros `bg-white` y textos `text-gray-900` sin soporte oscuro. | Baja | Integrar clases oscuras (`dark:bg-[#16161a]`, `dark:text-zinc-100`, bordes sutiles). |

---

## 7. Verificación de Cumplimiento con `AGENTS.md`

1. **Prohibición Estricta de Cápsulas y Eyebrow Badges Decorativos**:
   - **Resultado**: **100% CUMPLE**.
   - Se auditaron todas las ocurrencias de encabezados `<h1>`, `<h2>`, `<h3>` y `rounded-full`. No existen pastillas decorativas (`CAPACITACIÓN PROFESIONAL`, `SOBRE NOSOTROS`, etc.) sobre títulos.
   - En `StepDescriptionPoster.jsx:8` se identificó un import no utilizado de `SparklesIcon` sin renderizado alguno.
2. **Animaciones con GSAP y ScrollTrigger**:
   - **Resultado**: **CUMPLE**.
   - `src/utils/gsapConfig.js` registra `ScrollTrigger` y `useGSAP` con `ignoreMobileResize: true`. No se detectaron animaciones que oculten elementos a `opacity: 0` al salir del viewport.
3. **Identidad de Marca y Contraste**:
   - **Resultado**: **CUMPLE**.
   - Azul institucional `#00345e` aplicado como color maestro en botones principales, acentos y loaders.
   - Buen contraste general soportado por `DashboardThemeContext`.

---

## 8. Configuración de Build y Dependencias

- **Herramientas de Build**:
  - Vite v7.3.7
  - Tailwind CSS v4.1.13 con `@tailwindcss/vite`
  - React v19.1.1 / React-DOM v19.1.1
  - React Router v7.9.1
- **Entorno de Ejecución**:
  - Node.js: `v26.13.0` (ubicado en `/home/sebas/.nvm/versions/node/v26.13.0/bin/node`)
  - PNPM: `v12.18.0` (ubicado en `/home/sebas/.nvm/versions/node/v26.13.0/bin/pnpm`)
- **Prueba de Compilación**:
  - Comando ejecutado: `pnpm run build`
  - Resultado: **Exitoso (`✓ built in 19.27s`)**.
  - Salida: `dist/index.html` (0.46 kB), `dist/assets/index.css` (153.11 kB), `dist/assets/index.js` (812.43 kB).
