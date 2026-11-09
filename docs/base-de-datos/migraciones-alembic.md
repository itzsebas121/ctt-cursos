# Migraciones con Alembic - Base de Datos CTT FISEI

Este documento detalla la arquitectura, el flujo de trabajo y los procedimientos operativos estándar (SOP) para la gestión y evolución del esquema de la base de datos relacional mediante **Alembic**, la herramienta de control de versiones DDL integrada con SQLAlchemy y SQLModel.

---

## 1. Arquitectura de Migraciones en el Proyecto

En entornos de producción y desarrollo contenerizados con Docker, la creación de tablas mediante `create_all()` no es suficiente, ya que no permite aplicar modificaciones estructurales (`ALTER TABLE`, adición de índices, cambios en restricciones o mutación de tipos de columna) sin destruir los datos existentes.

Por ello, el backend de CTT FISEI delega el 100% de la gestión DDL a Alembic:

```
[Código de Modelos]                 [Scripts Alembic]                 [PostgreSQL 16]
SQLModel / DeclarativeBase  ----->   alembic/versions/   ----->   alembic upgrade head
(src/models/*.py)                  (revisiones DDL)             (tabla alembic_version)
```

### Configuración Principal (`alembic.ini`)

El archivo `Back-CTT/alembic.ini` define los parámetros operativos:
- `script_location = %(here)s/alembic`: Ubicación de los scripts ejecutables y plantillas.
- `file_template = %%(year)d_%%(month).2d_%%(day).2d_%%(hour).2d%%(minute).2d-%%(rev)s_%%(slug)s`: Nomenclatura cronológica para que los archivos de migración queden ordenados cronológicamente en el sistema de archivos (ej. `2026_07_15_1724-5b10776f8ef0_...py`).
- `prepend_sys_path = .`: Inclusión de la raíz del proyecto en `sys.path` para permitir la importación directa del paquete `src`.
- `sqlalchemy.url`: Se mantiene como un placeholder estático en el archivo `.ini`, ya que la URL viva se inyecta dinámicamente en tiempo de ejecución a través de `alembic/env.py`.

### Configuración del Entorno de Ejecución (`alembic/env.py`)

El archivo `Back-CTT/alembic/env.py` actúa como puente entre la configuración del proyecto y el motor de migraciones:

1. **Resolución Dinámica de URL**:
   Lee la variable de entorno `ALEMBIC_DATABASE_URL`. Si no está presente, reutiliza automáticamente `DATABASE_URL` desde `src.config.db`:
   ```python
   override_url = os.getenv("ALEMBIC_DATABASE_URL")
   if override_url:
       config.set_main_option("sqlalchemy.url", override_url.replace("%", "%%"))
   else:
       from src.config.db import DATABASE_URL
       config.set_main_option("sqlalchemy.url", DATABASE_URL.replace("%", "%%"))
   ```
   *Nota*: El reemplazo `replace("%", "%%")` previene que el parser interno `configparser` interprete el símbolo `%` presente en contraseñas con codificación URL (URL-encoding) como una variable de sustitución.

2. **Unificación de Metadatos (`target_metadata`)**:
   Importa exhaustivamente todos los modelos antes de invocar la introspección de esquema:
   ```python
   from src.models.base import Base
   import src.models.congress_model
   import src.models.sponsor_model
   import src.models.speaker_model
   import src.models.sesion_cronograma_model
   import src.models.congreso_sponsor_model
   import src.models.user
   import src.models.category
   import src.models.congress_category
   import src.models.course
   import src.models.post
   import src.models.enrollment
   import src.models.user_platform
   
   target_metadata = Base.metadata
   ```
   Dado que `Base.metadata` y `SQLModel.metadata` comparten la misma instancia, todos los modelos del ecosistema quedan cubiertos.

3. **Detección Avanzada de Tipos**:
   Al ejecutar el modo online, se configuran las directivas:
   - `compare_type=True`: Detecta cambios en longitud de `VARCHAR`, cambios de `INTEGER` a `BIGINT` o modificaciones de `NUMERIC`.
   - `compare_server_default=True`: Detecta alteraciones en valores por defecto calculados a nivel de servidor.

---

## 2. Historial Cronológico de Revisiones

El árbol de revisiones en `Back-CTT/alembic/versions/` registra la evolución formal de la base de datos:

```
[None]
  │
  ▼
[5b10776f8ef0]  (Baseline inicial - 2026-07-15)
  │
  ▼
[98ea811add01]  (Adición columna activo en sponsors - 2026-07-16)
  │
  ▼
[c3f81e29a001]  (Creación tabla session_attendance y unicidad QR - 2026-11-08)  ◄ HEAD
```

### Detalle de Revisiones

#### 1. Revisión `5b10776f8ef0` (Baseline Esquema Inicial)
- **Fecha**: 2026-07-15 17:24:38
- **Archivo**: `2026_07_15_1724-5b10776f8ef0_baseline_esquema_inicial_desde_modelos_.py`
- **Revisión Previa (`down_revision`)**: `None`
- **Operaciones Principales**:
  - Creación de tablas fundamentales: `congresos`, `posts`, `sponsors`, `users`, `users_platform`, `categories`, `congreso_sponsors`, `congress_categories`, `speakers`, `course`, `sesiones_cronograma`, `coursecontent`, `courserequirement`, `enrollments`.
  - Definición de Enums PostgreSQL: `userplatformtype`, `categorystatus`, `congresscategorystatus`, `coursestatus`, `enrollmentstatus`.
  - Creación de la restricción de verificación XOR en `enrollments`:
    ```sql
    CHECK ((id_course IS NOT NULL AND id_congress IS NULL) OR (id_course IS NULL AND id_congress IS NOT NULL))
    ```
  - Creación de los índices únicos parciales:
    - `uq_enrollment_user_congress` en `(id_user_platform, id_congress)` donde `id_congress IS NOT NULL`.
    - `uq_enrollment_user_course` en `(id_user_platform, id_course)` donde `id_course IS NOT NULL`.

#### 2. Revisión `98ea811add01` (Adición de Columna `activo` en `sponsors`)
- **Fecha**: 2026-07-16 14:59:59
- **Archivo**: `2026_07_16_1459-98ea811add01_add_sponsors_activo_column.py`
- **Revisión Previa (`down_revision`)**: `5b10776f8ef0`
- **Operaciones Principales**:
  - Incorporación de columna booleana para soft-delete / activación de patrocinadores:
    ```python
    op.add_column(
        'sponsors',
        sa.Column('activo', sa.Boolean(), nullable=False, server_default=sa.true())
    )
    op.alter_column('sponsors', 'activo', server_default=None)
    ```
  - *Patrón de diseño seguro*: Se utiliza `server_default=sa.true()` transitoriamente para poblar registros preexistentes sin violar la restricción `NOT NULL`, retirándolo inmediatamente para que la aplicación mantenga el control del valor por defecto en tiempo de ejecución.

#### 3. Revisión `c3f81e29a001` (Creación de Tabla `session_attendance`)
- **Fecha**: 2026-11-08 04:15:00
- **Archivo**: `2026_11_08_0415-c3f81e29a001_create_session_attendance_table.py`
- **Revisión Previa (`down_revision`)**: `98ea811add01`
- **Operaciones Principales**:
  - Creación de la tabla `session_attendance` para registro de presencia mediante escaneo QR:
    ```sql
    CREATE TABLE IF NOT EXISTS session_attendance (
        id SERIAL PRIMARY KEY,
        id_enrollment INTEGER NOT NULL REFERENCES enrollments(id) ON DELETE CASCADE,
        id_sesion INTEGER NOT NULL REFERENCES sesiones_cronograma(id_sesion) ON DELETE CASCADE,
        check_in_time TIMESTAMP WITHOUT TIME ZONE DEFAULT (NOW() AT TIME ZONE 'utc'),
        scanned_by_user_id INTEGER,
        notes VARCHAR(255),
        CONSTRAINT uq_enrollment_session UNIQUE (id_enrollment, id_sesion)
    );
    CREATE INDEX IF NOT EXISTS idx_attendance_session ON session_attendance(id_sesion);
    ```
  - Claves foráneas vinculadas con `ON DELETE CASCADE` hacia `enrollments` y `sesiones_cronograma`.
  - Restricción única `uq_enrollment_session` que imposibilita registros de asistencia duplicados para un mismo participante en una misma sesión académica.

---

## 3. Procedimiento Operativo Estándar (SOP)

### 3.1 Aplicación de Migraciones (`upgrade`)

Para aplicar todas las migraciones pendientes y llevar la base de datos al estado más reciente (`head`):

```bash
# Dentro del contenedor o en el entorno virtual activo:
cd Back-CTT
alembic upgrade head
```

Para aplicar exactamente una revisión hacia adelante:
```bash
alembic upgrade +1
```

Para aplicar hasta una revisión específica:
```bash
alembic upgrade 98ea811add01
```

### 3.2 Reversión de Migraciones (`downgrade`)

Para revertir la última migración aplicada:
```bash
alembic downgrade -1
```

Para revertir hasta una revisión específica:
```bash
alembic downgrade 5b10776f8ef0
```

Para revertir todas las migraciones (dejar la base de datos completamente vacía):
```bash
alembic downgrade base
```

### 3.3 Creación de Nuevas Migraciones

Cuando se modifica un modelo en `src/models/`, el desarrollador debe generar el script de migración correspondiente:

1. **Generación Automática (Detección por Diferencias)**:
   ```bash
   alembic revision --autogenerate -m "descripcion_del_cambio"
   ```
2. **Revisión Obligatoria del Código Generado**:
   Alembic genera un archivo nuevo en `alembic/versions/`. Se debe inspeccionar exhaustivamente la función `upgrade()` y `downgrade()`. Verificar que:
   - Los nombres de tablas y columnas coincidan exactamente con la convención `snake_case`.
   - Se manejen correctamente los valores por defecto en columnas `NOT NULL` sobre tablas con datos existentes.
   - Si se crean claves foráneas, existan los índices correspondientes.
3. **Prueba Bidireccional de la Migración**:
   Antes de confirmar en el repositorio:
   ```bash
   alembic upgrade head
   alembic downgrade -1
   alembic upgrade head
   ```

### 3.4 Comprobación de Estado e Historial

Para verificar qué revisión se encuentra aplicada en la base de datos actual:
```bash
alembic current
```

Para visualizar la lista de todas las revisiones y su orden de dependencia:
```bash
alembic history --verbose
```

---

## 4. Automatización en el Despliegue con Docker

En el entorno de producción y desarrollo contenerizado, las migraciones se aplican automáticamente antes de que la aplicación FastAPI comience a recibir tráfico.

El script de arranque `Back-CTT/scripts/wait-for-db.sh` implementa el siguiente flujo secuencial:

1. **Espera Activa de PostgreSQL**: Sondea periódicamente a PostgreSQL mediante un script auxiliar en Python (`/tmp/check_db.py`) hasta confirmar que el puerto 5432 acepta conexiones.
2. **Inicialización de Carpetas de Datos**: Crea las rutas requeridas en volúmenes montados (`static/images/courses`, `static/pdf`, `logs`, etc.) y asigna permisos al usuario de sistema no privilegiado `appuser`.
3. **Ejecución Automática de Migraciones**:
   ```sh
   echo "Aplicando migraciones de base de datos (alembic upgrade head)..."
   gosu appuser alembic upgrade head
   if [ $? -ne 0 ]; then
       echo "ERROR: Fallo al aplicar migraciones. Abortando arranque."
       exit 1
   fi
   ```
   Si la migración falla por inconsistencias de esquema o bloqueos en la base de datos, el contenedor aborta el arranque de inmediato, impidiendo que una API desincronizada atienda solicitudes.
4. **Inicio de FastAPI**: Una vez que Alembic confirma la salida exitosa (`exit code 0`), se ejecuta `uvicorn` a través de `gosu appuser`.
