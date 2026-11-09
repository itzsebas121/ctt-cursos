# Portal Web y Plataforma del Estudiante (CTT-WEB)

El portal público y plataforma del estudiante **CTT-WEB** es la aplicación cliente destinada a la difusión de la oferta académica de la Facultad de Ingeniería en Sistemas, Electrónica e Industrial (FISEI) de la Universidad Técnica de Ambato (UTA), la matriculación en cursos de capacitación continua y la participación en congresos científicos internacionales de alto impacto (tales como CSEI y CISED).

La plataforma permite a los usuarios explorar programas académicos, inscribirse con validación de aforo en tiempo real, generar órdenes de pago institucionales, gestionar comprobantes para facturación electrónica del SRI, rendir encuestas de acreditación CACES, postular ponencias científicas (Call for Papers) y consultar credenciales digitales de acceso con código QR.

---

## 1. Stack Tecnológico

El proyecto está construido sobre un stack de última generación optimizado para alto rendimiento, accesibilidad y fidelidad visual:

| Tecnología | Versión | Propósito Arquitectónico |
|---|---|---|
| **React** | `^19.1.1` | Librería principal de renderizado declarativo y gestión de interfaces basada en componentes. |
| **Vite** | `^7.1.2` | Entorno de compilación ultra-rápido con Hot Module Replacement (HMR) y empaquetado optimizado para producción. |
| **Tailwind CSS** | `^4.1.13` | Framework de utilidades CSS de última generación mediante `@tailwindcss/vite`, configurado con variables de tema nativas `@theme`. |
| **GSAP & @gsap/react** | `^3.15.1` / `^2.1.2` | Motor de animación profesional para transiciones suaves, contadores numéricos y revelado de tarjetas con ScrollTrigger. |
| **React Router DOM** | `^7.9.1` | Enrutamiento del lado del cliente con soporte de rutas anidadas, guardias de autenticación y redirecciones dinámicas. |
| **Axios** | `^1.12.2` | Cliente HTTP con configuración de timeout, baseURL e interceptores para peticiones REST a `Back-CTT`. |
| **Headless UI** | `^2.2.8` | Primitivas accesibles y sin estilos para diálogos modales, menús y transiciones de pantalla completa. |
| **Heroicons** | `^2.2.0` | Conjunto iconográfico institucional con variantes en línea y sólido para alta legibilidad. |

El servidor de desarrollo se ejecuta por defecto en el puerto **3000** (`http://localhost:3000`), comunicándose con la API backend de FastAPI en el puerto **3002** (`http://localhost:3002/api/v1`).

---

## 2. Arquitectura de Alto Nivel

La aplicación web opera como una Single Page Application (SPA) que desacopla la navegación del portal público institucional de la zona privada del dashboard del estudiante.

```mermaid
graph TD
    Client[Navegador del Estudiante / Participante] -->|Puerto 3000| SPA[SPA React 19 + Vite]
    
    subgraph Frontend CTT-WEB
        Router[React Router DOM v7]
        SPA --> Router
        
        Router --> PublicRoutes[Portal Público]
        Router --> PrivateRoutes[Dashboard Protegido]
        
        PublicRoutes --> Home[Home & Hero Carousel]
        PublicRoutes --> Catalog[Catálogo de Cursos y Eventos]
        PublicRoutes --> Details[Detalle Cursos / Congresos]
        PublicRoutes --> Auth[Login / Registro]
        
        PrivateRoutes --> Oferta[Oferta Académica Unificada]
        PrivateRoutes --> MisInscripciones[Cursos Inscritos & RIDE]
        PrivateRoutes --> MisOrdenes[Órdenes Pendientes y Completadas]
        PrivateRoutes --> MiPerfil[Perfil SRI / Seguridad / Finanzas]
        
        AuthState[Contexto de Autenticación & Storage] --> PrivateRoutes
        GSAPMotor[Motor GSAP + ScrollTrigger] --> PublicRoutes
    end

    subgraph Backend FastAPI
        API[Back-CTT :3002]
    end

    Frontend CTT-WEB -->|Peticiones HTTP Bearer JWT| API
```

---

## 3. Estructura del Proyecto (`src/`)

El código fuente de `CTT-WEB` está estructurado por responsabilidades funcionales y dominios de negocio:

```
CTT-WEB/src/
├── api/                             # Servicios de integración con endpoints REST
│   ├── api.js                       # Instancia base de Axios con baseURL y headers
│   ├── auth.js                      # Login local, login Microsoft 365, registro y perfil
│   ├── banners.js                   # Obtención de banners informativos y carruseles
│   ├── categories.js                # Listado de categorías académicas
│   ├── congresses.js                 # Consulta de congresos, sesiones y ponentes
│   ├── courses.js                   # Consulta de cursos, requisitos y contenidos
│   └── inscripciones.js             # Inscripciones, órdenes de pago y carga de comprobantes
├── assets/                          # Recursos gráficos institucionales (logos UTA y FISEI)
├── components/                      # Componentes reutilizables de UI y layout
│   ├── Banner.jsx                   # Banner animado institucional
│   ├── courses/                     # Componentes del catálogo y detalle de cursos
│   │   ├── CardCourse.jsx           # Tarjeta de curso con horas, modalidad y precio
│   │   ├── CourseContent.jsx        # Acordeón de unidades y temarios formativos
│   │   ├── CourseHeader.jsx         # Cabecera de curso con imagen y metadatos
│   │   ├── CourseMaterials.jsx      # Recursos didácticos y de laboratorio
│   │   ├── CourseObjetives.jsx      # Objetivos generales y específicos
│   │   ├── CourseRequirements.jsx   # Prerrequisitos de admisión
│   │   ├── CourseSection.jsx        # Bloque de cursos destacados en Home
│   │   ├── CourseSidebar.jsx        # Barra lateral con inversión, inscripción y aval
│   │   └── CoursesList.jsx          # Grid responsivo con filtros y animación stagger
│   ├── home/                        # Secciones de la página principal
│   │   ├── BannerImage.jsx          # Banner de vinculación con la colectividad
│   │   ├── Carousel.jsx             # Carrusel principal de diapositivas
│   │   ├── Hero.jsx                 # Bloque introductorio institucional
│   │   ├── Section.jsx              # Secciones con revelado visual
│   │   └── Stats.jsx                # Tarjetas numéricas con contador GSAP
│   ├── layout/                      # Estructura visual envolvente
│   │   ├── Footer.jsx               # Pie de página institucional UTA/FISEI
│   │   ├── Header.jsx               # Barra superior con enlaces a plataforma Moodle
│   │   ├── Menu.jsx                 # Menú lateral colapsable del dashboard
│   │   ├── Nav.jsx                  # Barra de navegación principal pública
│   │   └── ScrollToTop.jsx          # Reseteo de scroll e invalidación de ScrollTrigger
│   └── ui/                          # Primitivas de interfaz de usuario
│       ├── AppleAlert.jsx           # Mensajes de notificación estilizados
│       ├── ConfirmDialog.jsx        # Diálogo modal de confirmación
│       ├── DocumentUpload.jsx       # Componente de arrastre y carga de archivos
│       ├── EnrollmentModal.jsx      # Modal de confirmación de inscripción
│       ├── FilterSideBar.jsx        # Filtros laterales por categoría y modalidad
│       ├── ItemHeader.jsx           # Cabecera de módulo estándar
│       ├── PaymentReceiptModal.jsx  # Modal de carga de comprobante y datos SRI
│       ├── RoleSelection.jsx        # Selector de arancel (Estudiante, Profesional, etc.)
│       ├── SessionTimer.jsx         # Contador visual de expiración de sesión
│       ├── SurveyModal.jsx          # Encuesta de calidad académica CACES (1-5 estrellas)
│       └── Table.jsx                # Tabla de datos con filtros, búsqueda y paginación
├── constants/                       # Constantes globales
│   ├── navigation.js                # Definición de enlaces públicos y del dashboard
│   └── slides.js                    # Contenido de las diapositivas de portada
├── context/                         # Contextos globales de React
│   └── DashboardThemeContext.jsx    # Proveedor de tema claro/oscuro persistente
├── hooks/                           # Custom React Hooks
│   ├── useFetch.js                  # Manejo genérico de peticiones asíncronas
│   ├── useSessionExpiration.js      # Verificación y cierre automático tras 8 horas
│   └── useWindowSize.js             # Detección reactiva de dimensiones de pantalla
├── pages/                           # Páginas del sistema
│   ├── application/                 # Vistas privadas del Dashboard
│   │   ├── CongresosDisponibles.jsx # Catálogo de eventos científicos para estudiantes
│   │   ├── Cursos.jsx               # Listado de cursos para panel privado
│   │   ├── CursosInscritos.jsx      # Vista de mis cursos con estados y descarga RIDE
│   │   ├── Dashboard.jsx            # Layout principal del panel del estudiante
│   │   ├── OfertaAcademica.jsx      # Vista unificada de oferta con selector por tabs
│   │   ├── Ordenes.jsx              # Gestión de órdenes pendientes y completadas
│   │   └── Perfil.jsx               # Perfil con 4 pestañas: Personal, SRI, Clave, Finanzas
│   ├── auth/                        # Vistas de autenticación
│   │   ├── Login.jsx                # Inicio de sesión con pestañas Local y Office 365
│   │   ├── Register.jsx             # Formulario de registro de nuevas cuentas
│   │   └── typeLogin/               # Componentes específicos de cada método de login
│   ├── CongressDetail.jsx           # Vista completa de congreso con cupo y CFP
│   ├── CourseDetail.jsx             # Vista completa de curso con temario y requisitos
│   ├── Courses.jsx                  # Catálogo público de cursos
│   ├── Events.jsx                   # Catálogo público de congresos y talleres
│   ├── Home.jsx                     # Página de inicio institucional
│   ├── Nosotros.jsx                 # Misión, visión e historia del CTT-FISEI
│   └── Contact.jsx                  # Canales de atención ciudadana y ubicación
├── utils/                           # Utilidades y configuración
│   ├── auth.js                      # Helpers de localStorage, tokens y expiración
│   ├── gsapConfig.js                # Registro de plugins GSAP y configuración global
│   └── helpers.js                   # Formateo de fechas, moneda y resolución de URLs
├── App.jsx                          # Configuración de enrutamiento y guardias
├── index.css                        # Importación de Tailwind v4 y variables `@theme`
└── main.jsx                         # Punto de entrada de la aplicación en el DOM
```

---

## 4. Guía de Navegación de la Documentación

Para profundizar en la arquitectura y módulos específicos de `CTT-WEB`, consulte los siguientes documentos técnicos especializados:

1. [Arquitectura de Componentes y Sistema de Rutas](componentes-rutas.md):
   Estructura del árbol de navegación en React Router DOM v7, layout general, sistema de guardias (`PrivateRoute`), flujo de autenticación JWT y ciclo de vida de sesión de 8 horas.
2. [Catálogo Unificado, Inscripciones y Gestión Financiera](catalogo-pagos.md):
   Lógica del catálogo de cursos y congresos, control estricto de cupo máximo (`cupo_maximo`), máquina de estados de órdenes de pago, perfil con 4 pestañas, emisión de RIDE oficial UTA, pase digital QR, Call for Papers y encuesta CACES.
3. [Arquitectura de Animaciones con GSAP y ScrollTrigger](animaciones-gsap.md):
   Reglas institucionales estrictas (prohibición de `toggleActions: "play reverse"`, obligatoriedad de `clearProps: "all"` y `once: true`), reseteo de scroll con `ScrollToTop` y patrones con `useGSAP`.
4. [Estrategia de Diseño Responsive y Accesibilidad](diseno-responsive.md):
   Implementación mobile-first: transformación de tablas a tarjetas compactas apiladas en smartphones (< 640px), modales en formato bottom sheet, áreas táctiles mínimas de 44x44px, tokens de color institucional (`#00345e`) y contraste en modo claro y oscuro.
