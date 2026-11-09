# Subsistema de Base de Datos y Persistencia - CTT FISEI

Este módulo documenta la arquitectura de persistencia, modelado de datos, ciclo de vida de migraciones y estrategia de inicialización del ecosistema del Centro de Transferencia y Desarrollo de Tecnologías de la Facultad de Ingeniería en Sistemas, Electrónica e Industrial (FISEI) de la Universidad Técnica de Ambato (UTA).

El subsistema está construido sobre PostgreSQL 16 como motor relacional primario y utiliza una capa de abstracción de datos basada en Python 3.12, SQLModel y SQLAlchemy 2.0.

---

## 1. Visión General del Motor y Stack Tecnológico

El esquema de persistencia unifica la gestión académica de congresos científicos internacionales (CSEI, CISED), catálogo de cursos de formación continua, control de aforo en tiempo real, registro biométrico/QR de asistencia por sesión, flujo de pagos con emisión de RIDE (SRI Ecuador), Call for Papers (CFP) y auditoría de calidad académica según estándares CACES.

### Tecnologías Principales

| Componente | Tecnología | Versión | Rol Arquitectónico |
|---|---|---|---|
| Motor de Base de Datos | PostgreSQL | 16-alpine | Motor relacional transaccional (ACID), soporte nativo JSON y UUID |
| Conector DBAPI | Psycopg2-binary | >= 2.9.9 | Adaptador de conexión de alto rendimiento para PostgreSQL |
| ORM / Capa Híbrida | SQLModel + SQLAlchemy | SQLModel >= 0.0.25 / SQLAlchemy >= 2.0.43 | Mapeo objeto-relacional con validación Pydantic v2 integrada |
| Control de Esquema | Alembic | >= 1.13.0 | Versionamiento declarativo y automatizado de esquemas DDL |
| Pooling de Conexiones | SQLAlchemy Connection Pool | NullPool / QueuePool | Gestión de conexiones persistentes con reconexión automática |

### Unificación de Metadatos (SQLModel y DeclarativeBase)

El proyecto resuelve la interoperabilidad entre modelos definidos con SQLModel y modelos SQLAlchemy declarativos clásicos mediante la sincronización explícita del objeto de metadatos en `src/models/base.py`:

```python
class Base(DeclarativeBase):
    """Base unificada para modelos SQLAlchemy que comparte metadatos con SQLModel"""
    metadata = SQLModel.metadata
```

Esta estrategia garantiza que:
1. Las claves foráneas cruzadas (por ejemplo, entre `categories` definida en SQLAlchemy y `course` o `congresos` en SQLModel) resuelvan sin errores de inicialización.
2. La herramienta de control de versiones de esquema Alembic (`target_metadata = Base.metadata`) detecte la totalidad de las 17 tablas del ecosistema en una única pasada de introspección.

---

## 2. Índice del Módulo de Documentación

La documentación se organiza en cinco guías técnicas exhaustivas:

1. **[Modelos Relacionales](./modelos-relacionales.md)**  
   Especificación formal de los 17 modelos de datos del sistema, desglosando campos, tipos PostgreSQL, restricciones de integridad referencial, índices y reglas de validación en tiempo de ejecución.

2. **[Diagramas Entidad-Relación](./diagramas-er.md)**  
   Diagramas relacionales en sintaxis Mermaid estricta (`erDiagram`). Incluye la vista macro de todo el ecosistema y vistas modulares acotadas a los flujos operativos (Autenticación, Congresos/Sesiones/Asistencia, Inscripciones/RIDE, Papers/CFP y Encuestas CACES).

3. **[Diccionario de Datos](./diccionario-datos.md)**  
   Catálogo exhaustivo en tablas Markdown de cada tabla física, columna, tipo de dato SQL, nulabilidad, llaves (PK, FK, UQ, IDX), valores por defecto y justificación funcional con contexto institucional UTA.

4. **[Migraciones con Alembic](./migraciones-alembic.md)**  
   Arquitectura del subsistema de migraciones, configuración de `alembic.ini` y `env.py`, historial cronológico de revisiones en `alembic/versions/` y procedimiento operativo estándar (SOP) para despliegues seguros (`upgrade` y `downgrade`).

5. **[Semillas y Datos Iniciales](./semillas-datos-iniciales.md)**  
   Documentación del motor de seeds en `src/utils/seeds/`, datos de eventos emblemáticos FISEI (CSEI 2026, CISED 2025), esquema de tarifas oficiales ($20 Estudiante UTA, $50 Profesional, $80 Ponente), catálogo de cursos y sincronización de secuencias SERIAL.

---

## 3. Convenciones de Nomenclatura y Estándares

El diseño del esquema de base de datos sigue las siguientes directrices institucionales:

### Nombres de Tablas y Columnas

- **Tablas**: Se utilizan identificadores en minúsculas en formato `snake_case`. Las tablas siguen una convención mixta justificada por su evolución histórica:
  - Tablas del módulo de congresos y gobernanza académica: Pluralizadas en español (`congresos`, `speakers`, `sponsors`, `congreso_sponsors`, `sesiones_cronograma`, `session_attendance`, `scientific_papers`, `caces_surveys`).
  - Tablas del módulo de usuarios y administración: Pluralizadas en inglés/español (`users`, `users_platform`).
  - Tablas del catálogo educativo heredado: Nombres singulares o compuestos (`course`, `courserequirement`, `coursecontent`, `categories`, `congress_categories`).
- **Claves Primarias (PK)**:
  - Tablas principales de eventos: Prefijadas con el nombre de entidad (`id_congreso`, `id_speaker`, `id_sponsor`, `id_sesion`).
  - Tablas de usuarios e inscripciones: Estándar `id` como entero autoincremental (`SERIAL` o `IDENTITY`).
  - Tablas asociativas Many-to-Many: Clave primaria compuesta por las foráneas involucradas (ej. `(id_congreso, id_sponsor)` en `congreso_sponsors`).
- **Claves Foráneas (FK)**:
  - Se nombran explícitamente con el prefijo `id_` seguido de la entidad referenciada (ej. `id_congreso`, `id_speaker`, `id_user_platform`, `id_course`, `id_enrollment`) o con sufijo `_id` (`category_id`, `created_by`).
- **Restricciones e Índices**:
  - Claves únicas: Prefijo `uq_` (ej. `uq_enrollment_session`, `uq_enrollment_user_course`).
  - Índices estándar: Prefijo `ix_` o `idx_` (ej. `ix_users_email`, `idx_attendance_session`).
  - Restricciones de verificación (CHECK): Prefijo `chk_` (ej. `chk_enrollment_course_xor_congress`).

### Manejo Temporal y Husos Horarios

- Todos los timestamps de auditoría (`created_at`, `enrollment_date`, `check_in_time`, `submitted_at`, `updated_at`) se almacenan en tiempo universal coordinado (**UTC**).
- En PostgreSQL, los campos que requieren registro de fecha/hora se declaran como `TIMESTAMP WITHOUT TIME ZONE` inicializados en servidor mediante `(NOW() AT TIME ZONE 'utc')` o a nivel de ORM con `datetime.utcnow`.
- Las fechas de eventos y sesiones se almacenan como tipo `DATE` estricto (`YYYY-MM-DD`) y las horas de agenda como tipo `TIME` (`HH:MM:SS`) sin huso horario, garantizando precisión en la presentación de cronogramas locales en horario ecuatoriano (ECT / UTC-5).

### Tipos Numéricos y Monetarios

- Los valores monetarios de recaudación, tarifas y presupuestos institucionales se modelan en PostgreSQL como `NUMERIC(10, 2)` o `DOUBLE PRECISION` (`float` en SQLModel), asegurando 2 decimales para evitar desajustes en el cálculo de impuestos y desgloses RIDE ante el SRI.
