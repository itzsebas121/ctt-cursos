# Backend del Sistema CTT-FISEI (FastAPI)

El backend del ecosistema CTT-FISEI está implementado en **Python 3.12** utilizando **FastAPI** como framework web de alto rendimiento asíncrono, **SQLModel / SQLAlchemy** como capa de modelado objeto-relacional (ORM) y **PostgreSQL 16** como motor principal de base de datos relacional.

## Visión General de la API

La aplicación `Back-CTT` está diseñada bajo principios de arquitectura limpia y modular, separando las responsabilidades de enrutamiento HTTP, lógica de negocio y acceso a datos:

- **Rendimiento Asíncrono**: Basado en Starlette y Pydantic, garantizando serialización tipada y validación de esquemas en tiempo de compilación e interpretación.
- **Transaccionalidad Segura**: Inyección de dependencias (`Depends(get_db)`) que suministra sesiones SQLModel aisladas por petición HTTP con cierre y rollback determinista.
- **Autenticación Dual**: Soporte simultáneo para administradores universitarios y estudiantes mediante tokens JWT independientes cifrados con `HS256`.
- **Generación Binaria de Documentos**: Motores integrados con ReportLab para la emisión de facturas electrónicas RIDE autorizadas bajo normativa del SRI ecuatoriano y Órdenes de Pago institucionales UTA.
- **Integración Académica**: Conectores para sincronización con plataformas Moodle LMS y módulos de acreditación de calidad CACES.

## Estructura de Directorios en `Back-CTT/`

A continuación se detalla la organización del código fuente en el backend:

```
Back-CTT/
├── src/
│   ├── main.py                     # Punto de entrada de la aplicación, configuración de FastAPI, CORS y ciclo de vida lifespan
│   ├── config/
│   │   ├── db.py                   # Configuración del motor SQLModel (create_engine), variables de PostgreSQL y conexión
│   │   └── logging_config.py       # Configuración de RotatingFileHandler para logs/app.log y logs/error.log
│   ├── dependencies/
│   │   └── db_session.py           # Inyección de dependencias para sesiones de base de datos (get_db, SessionDep)
│   ├── middleware/
│   │   └── error_handler.py        # Middleware global para captura y sanitización de excepciones de SQLAlchemy
│   ├── models/                     # Modelos de base de datos SQLModel / SQLAlchemy (tablas y esquemas Pydantic)
│   │   ├── user.py                 # Modelo User (administradores) con métodos de hashing bcrypt
│   │   ├── user_platform.py        # Modelo UserPlatform (estudiantes/participantes) con datos fiscales SRI
│   │   ├── congress_model.py       # Modelo Congress (tarifas, fechas, cupo_maximo, sedes)
│   │   ├── enrollment.py           # Modelo Enrollment (estados, montos, tarifas institucionales)
│   │   ├── sesion_cronograma_model.py # Sesiones de cronograma asociadas a congresos y ponentes
│   │   ├── session_attendance_model.py # Registros de escaneo QR de asistencia a sesiones
│   │   ├── scientific_paper_model.py   # Propuestas de ponencias científicas (Call for Papers)
│   │   └── caces_survey_model.py   # Encuestas de evaluación de calidad académica (CACES)
│   ├── controllers/                # Capa de lógica de negocio pura
│   │   ├── enrollment_controller.py # Control de cupos máximos, validación de fechas y cálculo de tarifas
│   │   ├── congress_controller.py   # Gestión operativa de eventos, cronogramas y memorias
│   │   ├── user_controller.py       # Operaciones sobre administradores
│   │   └── user_platform_controller.py # Operaciones sobre participantes y perfiles
│   ├── routes/                     # Routers de FastAPI con endpoints REST
│   │   ├── auth_router.py          # Autenticación administrativa (/api/v1/auth)
│   │   ├── platform_auth_router.py # Autenticación de participantes (/api/v1/platform-auth)
│   │   ├── users_platform_router.py# Perfil de usuario, cambio de contraseña y datos fiscales (/api/v1/users)
│   │   ├── enrollments_router.py   # Gestión de inscripciones, órdenes UTA y RIDE PDF (/api/v1/enrollments)
│   │   ├── congress_router.py      # Congresos, asistencia QR, Moodle, CFP, encuestas (/api/v1/congresses-new)
│   │   ├── courses_router.py       # Gestión de cursos y encuestas CACES de cursos (/api/v1/courses)
│   │   ├── sesion_cronograma_router.py # Cronogramas y sesiones magistrales (/api/v1/sessions)
│   │   ├── speaker_router.py       # Ponentes y conferencistas (/api/v1/speakers)
│   │   ├── sponsor_router.py       # Patrocinadores y auspiciantes (/api/v1/sponsors)
│   │   └── congreso_sponsor_router.py # Relaciones de auspicio (/api/v1/sponsorships)
│   └── utils/                      # Utilidades, servicios y generadores
│       ├── jwt_utils.py            # Generación y validación de tokens JWT de administradores
│       ├── platform_jwt_utils.py   # Generación y validación de tokens JWT de plataforma
│       ├── ride_generator.py       # Generador de Comprobante / RIDE Oficial SRI con ReportLab
│       ├── uta_order_generator.py  # Generador de Órdenes de Pago Oficiales UTA con ReportLab
│       ├── email_service.py        # Motor de notificaciones SMTP con fallback local
│       └── seeds/                  # Semillas iniciales de desarrollo y pruebas
├── alembic/                        # Versiones de migración de base de datos
│   ├── env.py                      # Configuración de metadatos de SQLModel para migraciones
│   └── versions/                   # Scripts de migración individuales
├── tests/                          # Suite automatizada con pytest y TestClient
│   ├── test_enterprise_features.py # Pruebas de cupo, asistencia QR 409, bcrypt, RIDE y Moodle
│   └── conftest.py                 # Fixtures de base de datos en memoria (SQLite) y clientes mock
├── Dockerfile                      # Imagen Docker de producción basada en python:3.12-slim-bookworm
└── requirements.txt                # Dependencias de producción fijadas
```

## Guía de Navegación de la Documentación del Backend

Para consultar en detalle cada aspecto técnico del backend, acceda a los siguientes capítulos:

1. [Arquitectura Modular y Middlewares](arquitectura-modular.md): Ciclo de vida `lifespan`, inyección de dependencias `SessionDep`, manejo de errores de base de datos y logging rotativo.
2. [Sistema de Autenticación Dual JWT](autenticacion-jwt.md): Tokens para administración y participantes, algoritmo HS256, hashing con bcrypt y validación polimórfica.
3. [Controladores y Lógica de Negocio](controladores-servicios.md): Validación de cupo máximo, escaneo QR con control de colisiones HTTP 409, exportador Moodle LMS, notificaciones SMTP y módulo CACES.
4. [Reportes y Generación PDF](reportes-pdf.md): Motor ReportLab, estructura del comprobante electrónico RIDE SRI (RUC `1860000880001`) y Orden de Pago UTA.
5. [Catálogo Completo de Endpoints](catalogo-endpoints.md): Especificación exhaustiva de métodos HTTP, rutas, parámetros, esquemas de entrada y respuestas de todos los routers de la API.
6. [Volver al Portal Principal](../README.md)
