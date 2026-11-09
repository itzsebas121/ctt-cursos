# Panel Administrativo y de Gestión de Congresos (CTT-ADMIN)

El panel administrativo **CTT-ADMIN** es la plataforma de gestión académica, financiera y operativa del Centro de Transferencia y Desarrollo de Tecnologías (CTT) de la Facultad de Ingeniería en Sistemas, Electrónica e Industrial (FISEI) de la Universidad Técnica de Ambato (UTA).

Está diseñado para coordinadores académicos, directores de eventos, revisores del comité científico y personal de tesorería institucional, permitiendo administrar la oferta de cursos, configurar congresos internacionales (cronogramas, salas y ponentes), validar comprobantes de pago bancarios, registrar asistencia presencial a sesiones mediante lectura QR (con cámara o lector USB), evaluar artículos científicos (Call for Papers), monitorizar KPIs financieros de recaudación y exportar la nómina oficial de matriculados a Moodle LMS.

---

## 1. Stack Tecnológico

La plataforma administrativa está construida con tecnologías modernas que garantizan robustez, velocidad y compatibilidad multiplataforma:

| Tecnología | Versión | Propósito Arquitectónico |
|---|---|---|
| **React** | `^19.1.1` | Motor de interfaz de usuario reactivo y modular basado en componentes. |
| **Vite** | `^7.1.2` | Compilador y empaquetador ultrarrápido con soporte nativo de módulos ES. |
| **Tailwind CSS** | `^4.1.13` | Sistema de estilos utilitarios mediante `@tailwindcss/vite` y variables de tema `@theme`. |
| **html5-qrcode** | `^2.3.8` | Biblioteca de lectura óptica de códigos QR para escaneo mediante cámara trasera de dispositivos móviles. |
| **jwt-decode** | `^4.0.0` | Decodificación del token JWT en el cliente para extraer rol institucional y expiración de permisos. |
| **React Toastify** | `^11.0.5` | Notificaciones flotantes tipo toast para feedback de operaciones administrativas. |
| **React Router DOM** | `^7.9.1` | Gestión de rutas dinámicas, navegación declarativa y protección de endpoints administrativos. |
| **Axios** | `^1.12.2` | Cliente HTTP con interceptor automático para inyección de credenciales `Bearer <token>`. |
| **Heroicons** | `^2.2.0` | Iconografía estándar de alta legibilidad para dashboards y tablas. |

El servidor de desarrollo se ejecuta en el puerto **3001** (`http://localhost:3001`), comunicándose con el API backend en el puerto **3002** (`http://localhost:3002/api/v1`).

---

## 2. Arquitectura de Alto Nivel

CTT-ADMIN opera como una SPA protegida donde el 100% de las operaciones de negocio requieren un token Bearer con privilegios de administrador o coordinador:

```mermaid
graph TD
    AdminUser[Coordinador / Tesorero / Revisor] -->|Puerto 3001| SPA[SPA React 19 + Vite]
    
    subgraph Frontend CTT-ADMIN
        AuthGuard[Guardia PrivateRoute + jwt-decode]
        SPA --> AuthGuard
        
        AuthGuard --> DashLayout[Dashboard.jsx Layout]
        DashLayout --> SideMenu[Sidebar Menu.jsx con Submenús]
        DashLayout --> HeaderBar[Header con Breadcrumbs y Tema]
        
        DashLayout --> ModCourses[Gestión de Cursos y Categorías]
        DashLayout --> ModCongress[Gestión de Congresos y Cronogramas]
        DashLayout --> ModFinance[Dashboard Financiero & KPIs]
        DashLayout --> ModAttendance[Escáner Asistencia QR Dual]
        DashLayout --> ModCFP[Arbitraje de Artículos Call for Papers]
        DashLayout --> ModLMS[Exportador Oficial Moodle CSV]
    end

    subgraph Backend FastAPI
        API[Back-CTT :3002]
    end

    Frontend CTT-ADMIN -->|Axios con Interceptor JWT| API
```

---

## 3. Estructura del Proyecto (`src/`)

El código fuente de `CTT-ADMIN` organiza los componentes por dominio funcional:

```
CTT-ADMIN/src/
├── api/                             # Servicios de comunicación con el backend FastAPI
│   ├── api.js                       # Instancia Axios con interceptor de autorización Bearer
│   ├── auth.js                      # Autenticación administrativa y cierre de sesión
│   ├── banners.js                   # Administración de banners informativos
│   ├── categories.js                # CRUD de categorías de cursos
│   ├── congressCategories.js        # CRUD de categorías de congresos
│   ├── congresses.js                 # CRUD de congresos, sesiones, afiches y ponentes
│   ├── courses.js                   # CRUD de cursos, contenidos, materiales y requisitos
│   ├── enrolled_users.js            # Validación de comprobantes y gestión de matriculados
│   └── users.js                     # Administración de usuarios y roles
├── assets/                          # Logotipos institucionales UTA en versiones clara y oscura
├── components/                      # Componentes reutilizables de administración
│   ├── congress/                    # Componentes específicos de congresos
│   │   ├── AttendanceScannerModal.jsx # Escáner dual QR (Cámara + Lector USB)
│   │   ├── CongressCategoryModal.jsx  # Modal de gestión de categorías de congreso
│   │   ├── CreateSpeakerModal.jsx   # Modal de creación y asignación de ponentes
│   │   ├── CreateSponsorModal.jsx   # Modal de vinculación de auspiciantes/sponsors
│   │   ├── FinancialDashboardTab.jsx # Métricas financieras, aranceles y exportación LMS
│   │   ├── PaperReviewModal.jsx     # Dictamen de manuscritos CFP y asignación de arancel
│   │   ├── StepDescriptionPoster.jsx # Paso 2: Afiche oficial y descripción académica
│   │   ├── StepGeneralData.jsx      # Paso 1: Datos generales, cupo máximo y fechas
│   │   ├── StepSessions.jsx         # Paso 3: Cronograma de sesiones y salas
│   │   └── StepSponsors.jsx         # Paso 4: Configuración de patrocinadores
│   ├── courses/                     # Asistente multi-paso para creación de cursos
│   │   ├── StepGeneralData.jsx      # Datos generales, costos y fechas
│   │   ├── StepContents.jsx         # Temarios y unidades formativas
│   │   ├── StepRequirements.jsx     # Prerrequisitos de admisión
│   │   └── StepImages.jsx           # Carga de portada e imágenes descriptivas
│   ├── enrollment/                  # Gestión de matrículas
│   │   └── ValidatePaymentModal.jsx # Visor de comprobantes bancarios y aprobación
│   ├── layout/                      # Estructura del panel
│   │   ├── Footer.jsx               # Pie de página administrativo
│   │   ├── Header.jsx               # Cabecera con selector de tema y datos de sesión
│   │   ├── Menu.jsx                 # Menú lateral colapsable con submenús desplegables
│   │   ├── Nav.jsx                  # Barra de navegación superior
│   │   └── TablePageLayout.jsx      # Plantilla estándar para vistas tabulares con header
│   └── ui/                          # Componentes atómicos de interfaz
│       ├── ConfirmDialog.jsx        # Diálogo modal de confirmación crítica
│       ├── FilterSideBar.jsx        # Filtros laterales por estado y categoría
│       ├── ItemHeader.jsx           # Cabecera de entidad estándar
│       ├── Modal.jsx                # Primitiva modal accesible
│       ├── Select.jsx               # Selector accesible estilizado
│       └── Table.jsx                # Tabla de datos con filtros, búsqueda y paginación
├── constants/                       # Constantes del sistema
│   └── navigation.js                # Definición de rutas del menú administrativo
├── context/                         # Contextos globales
│   └── DashboardThemeContext.jsx    # Estado persistente del tema claro/oscuro
├── hooks/                           # Hooks de formulario y estado
│   ├── useBannerForm.js             # Lógica de carga de banners
│   ├── useCourseData.js             # Carga reactiva de cursos
│   └── useCourseForm.js             # Estado del wizard de cursos
├── pages/                           # Vistas de la aplicación
│   ├── CategoriasCongresos.jsx      # Administración de categorías temáticas de congreso
│   ├── CategoriasCursos.jsx         # Administración de áreas del conocimiento de cursos
│   ├── ConfiguracionHome.jsx        # Gestión de carruseles y banners de portada
│   ├── Congresos.jsx                # Listado general de congresos institucionales
│   ├── Cursos.jsx                   # Listado general de cursos de capacitación
│   ├── Dashboard.jsx                # Layout contenedor principal con Outlet
│   ├── Estudiantes.jsx              # Directorio de estudiantes y participantes registrados
│   ├── Home.jsx                     # Página principal administrativa
│   ├── Inscritos.jsx                # Listado de matriculados en cursos
│   ├── Login.jsx                    # Inicio de sesión con credenciales administrativas
│   ├── congress/                    # Vistas detalladas de congresos
│   │   ├── CongresosDetalle.jsx     # Ficha con pestañas: General, Cronograma, Finanzas, etc.
│   │   └── FormularioCongreso.jsx   # Wizard de creación y edición de congresos
│   └── courses/                     # Vistas detalladas de cursos
│       ├── CourseDetail.jsx         # Vista completa de curso
│       └── FormCourse.jsx           # Wizard de creación y edición de cursos
├── utils/                           # Utilidades auxiliares
│   ├── auth.js                      # Validación de token JWT, timestamp y expiración
│   ├── gsapConfig.js                # Configuración de transiciones
│   └── helpers.js                   # Resolución de URLs estáticas y formateo
├── App.jsx                          # Definición de rutas y ToastContainer
├── index.css                        # Importación de Tailwind v4 y variables institucionales
└── main.jsx                         # Renderizado en el elemento DOM raíz
```

---

## 4. Guía de Navegación de la Documentación

Consulte los siguientes manuales para el detalle operativo de los subsistemas de `CTT-ADMIN`:

1. [Arquitectura de Componentes y Sistema de Rutas](componentes-rutas.md):
   Estructura del layout administrativo, sidebar interactivo con submenús desplegables, migas de pan reactivas, interceptor Axios para JWT y componentes comunes (`Table.jsx`, `TablePageLayout.jsx`, `ValidatePaymentModal.jsx`).
2. [Gestión Académica, Acreditación y Dashboard Financiero](gestion-academica-financiera.md):
   Módulo de congresos y cronogramas, escáner de asistencia QR dual (`AttendanceScannerModal.jsx` con Web Audio API y háptica), control de duplicados (409 Conflict), dashboard de recaudación con desglose de aranceles ($20, $50, $80) y exportador en 1 clic para Moodle LMS.
3. [Módulo de Call for Papers y Arbitraje Científico](call-for-papers.md):
   Flujo de recepción y revisión de ponencias científicas, modal de dictamen editorial (`PaperReviewModal.jsx`), asignación automática de tarifa preferencial de Ponente ($80) y actualización en la orden de matrícula.
