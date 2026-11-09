Trabajar con varios subagentes: uno para base de datos, otro para backend, otro para los frontends y otro que vaya revisando todo el trabajo. Generar en `docs/` toda la documentación técnica de la aplicación tanto backend como frontend en español, organizada en carpetas estructuradas y basada en el código existente en `Back-CTT/`, `CTT-WEB/` y `CTT-ADMIN/`.

Integrity mode: demo
The team must NOT: ejecutar scripts externos o delegar la ejecución a herramientas externas

## Requirements

### R1. Documentación de Base de Datos y Modelos
Documentar los modelos relacionales, diagramas entidad-relación en Mermaid, diccionario de datos completo, esquemas de migración con Alembic y semillas/datos iniciales (incluyendo modelos de eventos oficiales FISEI UTA como CSEI y CISED).

### R2. Documentación del Backend (FastAPI)
Documentar la arquitectura modular de FastAPI en `Back-CTT/` (Python 3.12, SQLModel/SQLAlchemy, PostgreSQL 16), routers, controladores, sistema de autenticación JWT dual (Admin y Plataforma), generador de reportes PDF con ReportLab (RIDE SRI y Órdenes de Pago UTA), catálogo de endpoints y contratos de datos.

### R3. Documentación de los Frontends (Estudiante y Administrativo)
Documentar la arquitectura de `CTT-WEB/` (React 19 + Vite + Tailwind CSS + GSAP, puerto 3000) y `CTT-ADMIN/` (React 19 + Vite + Tailwind CSS, puerto 3001), detallando el catálogo unificado, gestión de pagos y órdenes, sistema de rutas, animaciones GSAP y diseño responsive.

### R4. Guías de Despliegue y Arquitectura Global
Documentar la visión general del sistema, arquitectura integrada entre backend y frontends, y guías de despliegue para desarrollo local y configuración estándar con Docker/producción según los archivos existentes en el repositorio.

### R5. Cumplimiento de Reglas Institucionales y Organización
Organizar toda la documentación en `docs/` en carpetas estructuradas (`arquitectura/`, `base-de-datos/`, `backend/`, `frontend-web/`, `frontend-admin/`, `despliegue/`), redactada íntegramente en español, cumpliendo estrictamente con las reglas de `AGENTS.md` (sin cápsulas, pill badges o overlines decorativos en encabezados, identidad visual con azul institucional `#00345e`).

### R6. Validación Cruzada y Auditoría
El subagente revisor / auditor debe validar de forma cruzada toda la documentación generada en `docs/`, verificando coherencia técnica con el código fuente, validez del formato Markdown, renderizado de diagramas Mermaid e integridad de enlaces.

## Acceptance Criteria

### Integridad y Cobertura Técnica
- [ ] La estructura de directorios en `docs/` contiene las carpetas `arquitectura/`, `base-de-datos/`, `backend/`, `frontend-web/`, `frontend-admin/` y `despliegue/`.
- [ ] Toda la documentación técnica está redactada en español.
- [ ] Los modelos relacionales, migraciones Alembic y diagramas ER en Mermaid coinciden con las definiciones en `Back-CTT/app/models/`.
- [ ] Todos los routers y endpoints del backend (`Back-CTT/`) están documentados con sus métodos HTTP, rutas, parámetros y contratos de respuesta.
- [ ] Los dos frontends (`CTT-WEB/` y `CTT-ADMIN/`) cuentan con documentación de su arquitectura de componentes, enrutamiento, integración de APIs y estilos.
- [ ] La guía de despliegue detalla los requisitos, variables de entorno y pasos de ejecución local y con Docker.
- [ ] Los encabezados markdown son limpios y directos, sin cápsulas ni overlines decorativos, en estricto cumplimiento de `AGENTS.md`.
- [ ] Todos los bloques Mermaid de diagramas de arquitectura y ER compilan y renderizan sin errores de sintaxis.
- [ ] Todos los enlaces relativos entre archivos markdown dentro de `docs/` resuelven a rutas existentes sin enlaces rotos.
