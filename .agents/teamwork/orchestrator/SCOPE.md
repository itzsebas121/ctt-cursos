# Scope: Generación de Documentación Técnica Completa en `docs/`

## Architecture & Overview
Documentar el ecosistema completo CTT-FISEI en español, organizado en:
- `docs/arquitectura/`: Visión global, arquitectura integrada backend-frontends, flujos de datos.
- `docs/base-de-datos/`: Modelos relacionales, diagramas ER Mermaid, diccionario de datos, migraciones Alembic, datos iniciales/semillas (CSEI, CISED).
- `docs/backend/`: Arquitectura modular FastAPI, routers, controladores, autenticación JWT dual, reportes ReportLab (RIDE SRI, órdenes UTA), catálogo de endpoints.
- `docs/frontend-web/`: Arquitectura React 19 + Vite + Tailwind + GSAP (puerto 3000), catálogo, pagos, órdenes, rutas, animaciones.
- `docs/frontend-admin/`: Arquitectura React 19 + Vite + Tailwind (puerto 3001), gestión de pagos, asistencia QR, dashboard financiero, CFP.
- `docs/despliegue/`: Guía de despliegue local y Docker/producción, variables de entorno.
- `docs/README.md`: Índice central y navegación unificada.

## Feature Inventory
| # | Documento / Área | Descripción | Asignado | Estado |
|---|-------------------|-------------|----------|--------|
| 1 | Base de Datos: Modelos y ER | Modelos SQLModel/SQLAlchemy, relaciones, diagramas Mermaid ER | worker_docs_db | DONE |
| 2 | Base de Datos: Diccionario & Alembic | Diccionario de tablas, columnas, tipos, constraints; guía Alembic | worker_docs_db | DONE |
| 3 | Base de Datos: Semillas & Eventos | Seeds oficiales FISEI UTA (CSEI, CISED, tarifas, ponentes) | worker_docs_db | DONE |
| 4 | Backend: Arquitectura Modular | FastAPI, estructura de carpetas, inyección de dependencias, configuración | worker_docs_backend | DONE |
| 5 | Backend: Autenticación Dual JWT | Autenticación Admin (bcrypt/JWT) y Plataforma (estudiantes) | worker_docs_backend | DONE |
| 6 | Backend: Catálogo de Endpoints | Routers, métodos HTTP, parámetros, contratos de petición/respuesta | worker_docs_backend | DONE |
| 7 | Backend: Reportes ReportLab | Generador RIDE SRI (RUC 1860000880001) y Órdenes de Pago UTA | worker_docs_backend | DONE |
| 8 | Arquitectura Global | Diagramas Mermaid de arquitectura integrada, capas y flujos | worker_docs_backend | DONE |
| 9 | Guía de Despliegue | Configuración local, Docker Compose, variables de entorno | worker_docs_backend | DONE |
| 10 | Frontend Web (Estudiante) | CTT-WEB: React 19, componentes, rutas, GSAP, catálogo, pagos | worker_docs_frontend | DONE |
| 11 | Frontend Admin | CTT-ADMIN: React 19, dashboard, QR scanner, reportes, CFP | worker_docs_frontend | DONE |
| 12 | Validación y Auditoría | Verificación de sintaxis Mermaid, enlaces relativos, cumplimiento AGENTS.md | reviewer_docs | DONE |

## Milestones
| # | Name | Scope | Dependencies | Status |
|---|------|-------|-------------|--------|
| M1 | Base de Datos | `docs/base-de-datos/` | none | DONE |
| M2 | Backend, Arquitectura y Despliegue | `docs/backend/`, `docs/arquitectura/`, `docs/despliegue/`, `docs/README.md` | none | DONE |
| M3 | Frontends (Web y Admin) | `docs/frontend-web/`, `docs/frontend-admin/` | none | DONE |
| M4 | Revisión Cruzada y Auditoría | Validación completa de `docs/` | M1, M2, M3 | DONE |

## Interface & Format Rules
- Idioma: 100% Español.
- Regla AGENTS.md: NUNCA usar cápsulas, pill badges ni overlines decorativos sobre títulos (`#`, `##`, `###`).
- Color institucional de referencia: `#00345e`.
- Diagramas Mermaid: sintaxis estricta y limpia, sin errores de renderizado.
- Enlaces relativos: deben apuntar a rutas existentes dentro de `docs/`.
