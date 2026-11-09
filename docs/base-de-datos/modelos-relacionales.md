# Modelos Relacionales del Ecosistema CTT FISEI

Este documento detalla la especificación técnica y de negocio de todos los modelos de datos que componen la plataforma CTT FISEI. El sistema combina modelos declarativos de SQLAlchemy con clases SQLModel que proporcionan soporte unificado de serialización y validación bidireccional mediante Pydantic v2.

---

## 1. Módulo de Administración y Usuarios

### 1.1 Modelo `User` (Tabla `users`)

- **Archivo Fuente**: `src/models/user.py`
- **Tipo de Definición**: SQLAlchemy DeclarativeBase clásico con metadatos compartidos.
- **Propósito Institucional**: Representa a los usuarios con privilegios administrativos (coordinadores de carrera, administradores del CTT FISEI y personal de soporte) con acceso al panel administrativo `CTT-ADMIN`.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Descripción |
|---|---|---|---|---|---|
| `id` | `INTEGER` | `int` | No | PK (Autoincrement) | Identificador único del usuario administrativo |
| `name` | `VARCHAR(100)` | `str` | No | - | Nombres del usuario |
| `last_name` | `VARCHAR(100)` | `str` | No | - | Apellidos del usuario |
| `email` | `VARCHAR(320)` | `str` | No | UQ, IDX (`ix_users_email`) | Correo electrónico institucional (RFC 5321) |
| `password` | `VARCHAR(255)` | `str` | No | - | Hash bcrypt con salt de la contraseña |

#### Relaciones
- **Categorías Creadas**: `User.categories` (1:N hacia `Category`, relación inversa `Category.creator`).
- **Categorías de Congresos Creadas**: `User.congress_categories` (1:N hacia `CongressCategory`, relación inversa `CongressCategory.creator`).

#### Métodos de Seguridad Criptográfica
- `verify_password(plain_password: str) -> bool`: Valida el texto plano contra el hash almacenado, truncando a 72 bytes según la especificación RFC de bcrypt.
- `hash_password(password: str) -> str`: Genera una sal segura con `bcrypt.gensalt()` y produce el hash UTF-8 de longitud 60.

---

### 1.2 Modelo `UserPlatform` (Tabla `users_platform`)

- **Archivo Fuente**: `src/models/user_platform.py`
- **Tipo de Definición**: SQLModel (`SQLModel, table=True`).
- **Propósito Institucional**: Cuenta de usuario final (estudiantes UTA, docentes, profesionales y público general externo) utilizada en el portal público `CTT-WEB` para inscripciones, pagos, consulta de pases QR y descarga de facturas RIDE.

#### Enumeración `UserPlatformType`
- `ESTUDIANTE`: Estudiante regular matriculado en la Universidad Técnica de Ambato.
- `EXTERNO`: Participante independiente o profesional externo.
- `ADMINISTRATIVO`: Personal administrativo o docente universitario.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id` | `INTEGER` | `Optional[int]` | No | PK (Autoincrement) | `None` | Identificador único de usuario de plataforma |
| `identification` | `VARCHAR(20)` | `str` | No | - | - | Cédula ecuatoriana (10 dígitos), RUC (13 dígitos) o pasaporte |
| `first_name` | `VARCHAR(100)` | `str` | No | - | - | Primer nombre |
| `second_name` | `VARCHAR(100)` | `Optional[str]` | Sí | - | `None` | Segundo nombre |
| `first_last_name` | `VARCHAR(100)` | `str` | No | - | - | Primer apellido |
| `second_last_name` | `VARCHAR(100)` | `Optional[str]` | Sí | - | `None` | Segundo apellido |
| `cellphone` | `VARCHAR(15)` | `str` | No | - | - | Teléfono celular de contacto |
| `email` | `VARCHAR(320)` | `str` | No | UQ, IDX (`ix_users_platform_email`) | - | Correo electrónico personal o institucional |
| `address` | `VARCHAR(255)` | `Optional[str]` | Sí | - | `None` | Dirección física de residencia |
| `type` | `VARCHAR(50)` (SQLEnum) | `UserPlatformType` | No | - | - | Tipo de participante en la plataforma |
| `password` | `VARCHAR(255)` | `str` | No | - | - | Contraseña hasheada con bcrypt |
| `billing_identification` | `VARCHAR(20)` | `Optional[str]` | Sí | - | `None` | RUC o cédula fiscal para comprobantes RIDE SRI |
| `billing_name` | `VARCHAR(200)` | `Optional[str]` | Sí | - | `None` | Razón social o nombres completos de facturación |
| `billing_address` | `VARCHAR(255)` | `Optional[str]` | Sí | - | `None` | Domicilio tributario para emisión de factura |
| `billing_phone` | `VARCHAR(20)` | `Optional[str]` | Sí | - | `None` | Teléfono de contacto tributario |
| `billing_email` | `VARCHAR(320)` | `Optional[str]` | Sí | - | `None` | Correo electrónico para entrega del comprobante electrónico |

---

## 2. Módulo de Congresos Científicos y Gestión de Eventos

### 2.1 Modelo `Congress` (Tabla `congresos`)

- **Archivo Fuente**: `src/models/congress_model.py`
- **Tipo de Definición**: SQLModel (`SQLModel, table=True`).
- **Propósito Institucional**: Entidad principal que almacena los congresos científicos insignia de la facultad (ej. CSEI - Congreso de Ciencia de la Computación, Electrónica e Ingeniería Industrial; CISED - Jornadas de Investigación y Transferencia Tecnológica).

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id_congreso` | `INTEGER` | `Optional[int]` | No | PK (Autoincrement) | `None` | Identificador institucional del congreso |
| `nombre` | `VARCHAR(200)` | `str` | No | - | - | Nombre oficial del evento científico |
| `edicion` | `VARCHAR(10)` | `str` | No | - | - | Código identificador de la edición (ej. `VII-2026`) |
| `anio` | `INTEGER` | `int` | No | - | - | Año calendario del evento (validado entre 2000 y 2100) |
| `fecha_inicio` | `DATE` | `date` | No | - | - | Fecha de inauguración del congreso |
| `fecha_fin` | `DATE` | `date` | No | - | - | Fecha de clausura (debe ser mayor a `fecha_inicio`) |
| `descripcion_general` | `TEXT` | `str` | No | - | - | Resumen ejecutivo, alcance, temáticas y memorias |
| `poster_cover_url` | `VARCHAR(300)` | `str` | No | - | - | URL del afiche oficial en alta resolución |
| `lugar_sede` | `VARCHAR(150)` | `Optional[str]` | Sí | - | `'Campus Huachi - FISEI, Ambato'` | Ubicación física o auditorio principal |
| `modalidad` | `VARCHAR(30)` | `Optional[str]` | Sí | - | `'Presencial'` | Modalidad: Presencial, Híbrida o Virtual |
| `duracion_horas` | `INTEGER` | `Optional[int]` | Sí | - | `40` | Total de horas académicas certificadas |
| `sitio_web_oficial` | `VARCHAR(300)` | `Optional[str]` | Sí | - | `''` | Portal oficial web del congreso |
| `cupo_maximo` | `INTEGER` | `Optional[int]` | Sí | - | `250` | Aforo máximo permitido de participantes |
| `costo_estudiante_uta` | `DOUBLE PRECISION` | `Optional[float]` | Sí | - | `0.00` | Tarifa subvencionada para estudiantes UTA ($20.00) |
| `costo_profesional` | `DOUBLE PRECISION` | `Optional[float]` | Sí | - | `0.00` | Tarifa regular para profesionales ($50.00) |
| `costo_ponente` | `DOUBLE PRECISION` | `Optional[float]` | Sí | - | `0.00` | Tarifa especial para ponentes de papers ($80.00) |

#### Relaciones
- **Speakers**: `Congress.speakers` (1:N hacia `Speaker`, relación inversa `Speaker.congreso`).
- **Sesiones de Cronograma**: `Congress.sesiones` (1:N hacia `SesionCronograma`, relación inversa `SesionCronograma.congreso`).
- **Auspiciantes**: `Congress.congreso_sponsors` (1:N hacia `CongresoSponsor`, relación inversa `CongresoSponsor.congreso`).

---

### 2.2 Modelo `Speaker` (Tabla `speakers`)

- **Archivo Fuente**: `src/models/speaker_model.py`
- **Tipo de Definición**: SQLModel (`SQLModel, table=True`).
- **Propósito Institucional**: Registro de expositores nacionales e internacionales participantes en ponencias magistrales, talleres o paneles.

#### Enumeración `SpeakerType`
- `KEYNOTE`: Ponente magistral invitado de alto impacto internacional.
- `CONFERENCIA`: Conferencista de tracks temáticos.
- `TALLER`: Instructor práctico en laboratorios especializados.
- `PANEL`: Panelista en mesas redondas de debate.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id_speaker` | `INTEGER` | `Optional[int]` | No | PK (Autoincrement) | `None` | Identificador único del expositor |
| `id_congreso` | `INTEGER` | `Optional[int]` | Sí | FK (`congresos.id_congreso`) | `None` | Congreso en el que participa el ponente |
| `nombres_completos` | `VARCHAR(200)` | `str` | No | - | - | Nombre y apellidos del conferencista |
| `titulo_academico` | `VARCHAR(50)` | `str` | No | - | - | Grado profesional (ej. PhD, MSc, Ing.) |
| `institucion` | `VARCHAR(200)` | `str` | No | - | - | Universidad, centro de I+D o empresa de procedencia |
| `pais` | `VARCHAR(100)` | `str` | No | - | - | País de origen o radicación profesional |
| `foto_url` | `VARCHAR(300)` | `str` | No | - | - | URL de la fotografía de perfil del expositor |
| `tipo_speaker` | `VARCHAR(20)` | `str` | No | - | - | Rol académico (`keynote`, `conferencia`, etc.) |

---

### 2.3 Modelo `Sponsor` (Tabla `sponsors`)

- **Archivo Fuente**: `src/models/sponsor_model.py`
- **Tipo de Definición**: SQLModel (`SQLModel, table=True`).
- **Propósito Institucional**: Directorio maestro de entidades auspiciantes, patrocinadores industriales y aliados estratégicos del CTT FISEI.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id_sponsor` | `INTEGER` | `Optional[int]` | No | PK (Autoincrement) | `None` | Identificador del auspiciante |
| `nombre` | `VARCHAR(150)` | `str` | No | - | - | Razón social o marca del patrocinador |
| `logo_url` | `VARCHAR(300)` | `str` | No | - | - | URL del logotipo vectorial o bitmap |
| `sitio_web` | `VARCHAR(200)` | `str` | No | - | - | Enlace al portal corporativo |
| `descripcion` | `TEXT` | `str` | No | - | - | Resumen de actividad empresarial y aporte |
| `activo` | `BOOLEAN` | `bool` | No | - | `True` | Estado de vigencia (migración `98ea811add01`) |

---

### 2.4 Modelo `CongresoSponsor` (Tabla `congreso_sponsors`)

- **Archivo Fuente**: `src/models/congreso_sponsor_model.py`
- **Tipo de Definición**: SQLModel con clave primaria compuesta.
- **Propósito Institucional**: Tabla de unión Many-to-Many entre `congresos` y `sponsors`, registrando la categoría de auspicio y el aporte financiero o en especie.

#### Enumeración `SponsorCategory`
- `PLATINO`: Auspicio premium superior a $15,000 USD.
- `ORO`: Auspicio principal ($10,000 - $15,000 USD).
- `PLATA`: Patrocinio intermedio ($5,000 - $9,999 USD).
- `BRONCE`: Auspicio base ($1,000 - $4,999 USD).
- `COLABORADOR`: Alianza institucional, logística o mediática.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id_congreso` | `INTEGER` | `int` | No | PK, FK (`congresos.id_congreso`) | - | Identificador del congreso patrocinado |
| `id_sponsor` | `INTEGER` | `int` | No | PK, FK (`sponsors.id_sponsor`) | - | Identificador del sponsor asociado |
| `categoria` | `VARCHAR(50)` | `str` | No | - | - | Nivel de auspicio (`platino`, `oro`, etc.) |
| `aporte` | `NUMERIC(10, 2)` | `Decimal` | No | - | - | Monto económico en USD aportado al evento |

---

### 2.5 Modelo `SesionCronograma` (Tabla `sesiones_cronograma`)

- **Archivo Fuente**: `src/models/sesion_cronograma_model.py`
- **Tipo de Definición**: SQLModel (`SQLModel, table=True`).
- **Propósito Institucional**: Define cada una de las sesiones magistrales, ponencias y talleres dentro del cronograma del congreso, asociadas a una fecha, franja horaria, ponente y aula/auditorio.

#### Enumeración `Jornada`
- `mañana`: Sesiones matutinas (08:00 - 13:00).
- `tarde`: Sesiones vespertinas (14:00 - 19:00).
- `noche`: Eventos nocturnos de clausura o networking.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id_sesion` | `INTEGER` | `Optional[int]` | No | PK (Autoincrement) | `None` | Identificador de la sesión en el cronograma |
| `id_congreso` | `INTEGER` | `int` | No | FK (`congresos.id_congreso`) | - | Congreso al que pertenece la sesión |
| `id_speaker` | `INTEGER` | `int` | No | FK (`speakers.id_speaker`) | - | Ponente responsable de la sesión |
| `fecha` | `DATE` | `date` | No | - | - | Fecha en que se dicta la ponencia |
| `hora_inicio` | `TIME` | `time` | No | - | - | Hora de inicio planificada |
| `hora_fin` | `TIME` | `time` | No | - | - | Hora de finalización (debe ser mayor a `hora_inicio`) |
| `titulo_sesion` | `VARCHAR(200)` | `str` | No | - | - | Título técnico o tema de la ponencia |
| `jornada` | `VARCHAR(20)` | `str` | No | - | - | Jornada horaria (`mañana`, `tarde`, `noche`) |
| `lugar` | `VARCHAR(150)` | `str` | No | - | - | Auditorio, aula magna o sala de videoconferencia |

---

### 2.6 Modelo `CongressCategory` (Tabla `congress_categories`)

- **Archivo Fuente**: `src/models/congress_category.py`
- **Tipo de Definición**: SQLAlchemy DeclarativeBase clásico.
- **Propósito Institucional**: Clasificador temático institucional para los congresos (ej. Inteligencia Artificial, Ciberseguridad, Energías Renovables, Robótica).

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id` | `INTEGER` | `int` | No | PK (Autoincrement) | - | Identificador de la categoría de congreso |
| `name` | `VARCHAR(150)` | `str` | No | UQ, IDX (`ix_congress_categories_name`) | - | Nombre de la categoría |
| `description` | `VARCHAR(500)` | `Optional[str]` | Sí | - | `None` | Alcance y líneas de investigación asociadas |
| `svgurl` | `VARCHAR(500)` | `Optional[str]` | Sí | - | `None` | URL del icono en formato SVG |
| `status` | `VARCHAR(50)` (SQLEnum) | `CongressCategoryStatus` | No | - | `ACTIVO` | Estado operativo (`activo`, `inactivo`) |
| `created_at` | `TIMESTAMP` | `datetime` | No | - | `datetime.now` | Fecha de creación del registro |
| `updated_at` | `TIMESTAMP` | `Optional[datetime]` | Sí | - | `None` | Fecha de última modificación |
| `created_by` | `INTEGER` | `int` | No | FK (`users.id`) | - | Usuario administrativo que creó la categoría |

---

## 3. Módulo de Cursos de Formación Continua

### 3.1 Modelo `Category` (Tabla `categories`)

- **Archivo Fuente**: `src/models/category.py`
- **Tipo de Definición**: SQLAlchemy DeclarativeBase clásico.
- **Propósito Institucional**: Áreas del conocimiento de los cursos de capacitación continua (ej. TICS, Automatización, Gestión Empresarial).

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id` | `INTEGER` | `int` | No | PK (Autoincrement) | - | Identificador de la categoría de cursos |
| `name` | `VARCHAR(150)` | `str` | No | UQ, IDX (`ix_categories_name`) | - | Nombre único de la categoría |
| `description` | `VARCHAR(500)` | `Optional[str]` | Sí | - | `None` | Descripción de la oferta formativa |
| `svgurl` | `VARCHAR(500)` | `Optional[str]` | Sí | - | `None` | Icono representativo SVG |
| `status` | `VARCHAR(50)` (SQLEnum) | `CategoryStatus` | No | - | `ACTIVO` | Estado (`activo`, `inactivo`) |
| `created_at` | `TIMESTAMP` | `datetime` | No | - | `datetime.now` | Registro temporal de creación |
| `updated_at` | `TIMESTAMP` | `Optional[datetime]` | Sí | - | `None` | Registro temporal de actualización |
| `created_by` | `INTEGER` | `int` | No | FK (`users.id`) | - | Usuario administrativo responsable |

---

### 3.2 Modelo `Course` (Tabla `course`)

- **Archivo Fuente**: `src/models/course.py`
- **Tipo de Definición**: SQLAlchemy DeclarativeBase clásico.
- **Propósito Institucional**: Catálogo general de cursos de transferencia y capacitación técnica ofrecidos por la FISEI.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id` | `INTEGER` | `int` | No | PK (Autoincrement) | - | Identificador del curso |
| `title` | `VARCHAR(200)` | `str` | No | - | - | Título de la acción formativa |
| `description` | `TEXT` | `str` | No | - | - | Resumen detallado del programa académico |
| `place` | `VARCHAR(200)` | `str` | No | - | - | Sede o campus de ejecución |
| `course_image` | `VARCHAR(500)` | `str` | No | - | - | Imagen principal de portada (tarjeta de catálogo) |
| `course_image_detail` | `VARCHAR(500)` | `str` | No | - | - | Banner para la página de detalle |
| `category_id` | `INTEGER` | `int` | No | FK (`categories.id`), IDX | - | Categoría académica a la que pertenece |
| `status` | `VARCHAR(50)` (SQLEnum) | `CourseStatus` | No | - | `ACTIVO` | Estado del curso (`activo`, `inactivo`) |
| `objectives` | `TEXT` | `str` | No | - | - | Objetivos de aprendizaje serializados en JSON |
| `organizers` | `TEXT` | `str` | No | - | - | Instructores y organizadores en formato JSON |
| `materials` | `TEXT` | `str` | No | - | - | Requerimientos de hardware/software en JSON |
| `target_audience` | `TEXT` | `str` | No | - | - | Perfil de ingreso del estudiante en JSON |

---

### 3.3 Modelo `CourseRequirement` (Tabla `courserequirement`)

- **Archivo Fuente**: `src/models/course.py`
- **Tipo de Definición**: SQLAlchemy DeclarativeBase clásico con relación 1:1 estricta.
- **Propósito Institucional**: Especificaciones operativas, fechas de postulación, aforos, horarios y precios para un curso.

#### Restricción de Unicidad
- Clave foránea única: `course_id` posee restricción `UNIQUE`, asegurando que cada curso tenga a lo sumo un registro de requerimientos técnicos y logísticos.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id` | `INTEGER` | `int` | No | PK (Autoincrement) | - | Identificador de requerimiento |
| `course_id` | `INTEGER` | `int` | No | FK (`course.id`), UQ | - | Identificador del curso vinculado (1:1) |
| `start_date_registration` | `DATE` | `date` | No | - | - | Inicio de inscripciones |
| `end_date_registration` | `DATE` | `date` | No | - | - | Cierre de inscripciones |
| `start_date_course` | `DATE` | `date` | No | - | - | Fecha de inicio de clases |
| `end_date_course` | `DATE` | `date` | No | - | - | Fecha de finalización |
| `days` | `TEXT` | `str` | No | - | - | Días de impartición serializados en JSON |
| `start_time` | `TIME` | `time` | No | - | - | Hora de inicio diaria |
| `end_time` | `TIME` | `time` | No | - | - | Hora de salida diaria |
| `location` | `VARCHAR(200)` | `str` | No | - | - | Laboratorio específico de docencia |
| `min_quota` | `INTEGER` | `int` | No | - | - | Cupo mínimo para apertura del curso |
| `max_quota` | `INTEGER` | `int` | No | - | - | Cupo máximo permitido por laboratorio |
| `in_person_hours` | `INTEGER` | `int` | No | - | `0` | Horas presenciales en aula |
| `autonomous_hours` | `INTEGER` | `int` | No | - | `0` | Horas de trabajo autónomo guiado |
| `modality` | `VARCHAR(100)` | `str` | No | - | - | Modalidad (Presencial / Híbrida / Virtual) |
| `certification` | `VARCHAR(200)` | `str` | No | - | - | Tipo de certificación institucional emitida |
| `prerequisites` | `TEXT` | `str` | No | - | - | Conocimientos previos serializados en JSON |
| `prices` | `TEXT` | `str` | No | - | - | Tarifas por segmento serializadas en JSON |

---

### 3.4 Modelo `CourseContent` (Tabla `coursecontent`)

- **Archivo Fuente**: `src/models/course.py`
- **Tipo de Definición**: SQLAlchemy DeclarativeBase clásico (Relación 1:N con `Course`).
- **Propósito Institucional**: Representa las unidades temáticas y el sílabo curricular de cada curso.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id` | `INTEGER` | `int` | No | PK (Autoincrement) | - | Identificador de la unidad didáctica |
| `course_id` | `INTEGER` | `int` | No | FK (`course.id`) | - | Curso al que corresponde la unidad |
| `unit` | `VARCHAR(100)` | `str` | No | - | - | Número o denominación de unidad (ej. 'Unidad 1') |
| `title` | `VARCHAR(200)` | `str` | No | - | - | Título de la unidad temática |
| `topics` | `TEXT` | `str` | No | - | - | Desglose de tópicos y subtemas en formato JSON |

---

## 4. Módulo de Inscripciones, Asistencia QR y Facturación RIDE

### 4.1 Modelo `Enrollment` (Tabla `enrollments`)

- **Archivo Fuente**: `src/models/enrollment.py`
- **Tipo de Definición**: SQLModel con polimorfismo exclusivo mediante `CheckConstraint` e índices únicos parciales.
- **Propósito Institucional**: Administra las inscripciones tanto a congresos como a cursos de capacitación continua, controlando el ciclo de vida del pago, la tarifa aplicada y los comprobantes electrónicos.

#### Enumeración `EnrollmentStatus`
- `INTERESADO`: El usuario ha manifestado interés o seleccionado la tarifa en la interfaz.
- `ORDEN_GENERADA`: Se emitió la orden de pago institucional UTA con número de trámite.
- `PAGADO`: El pago fue validado en tesorería o tesorería virtual.
- `FACTURADO`: Se emitió la factura electrónica con comprobante RIDE generado por ReportLab.
- `ANULADO`: Registro dado de baja por impago o anulación expresa.

#### Reglas de Integridad Estricta en Base de Datos

1. **Exclusión Mutua Curso vs. Congreso (`chk_enrollment_course_xor_congress`)**:
   ```sql
   CHECK (
       (id_course IS NOT NULL AND id_congress IS NULL) OR
       (id_course IS NULL AND id_congress IS NOT NULL)
   )
   ```
   Garantiza que una inscripción apunte a un curso O a un congreso, nunca a ambos simultáneamente y nunca a ninguno.

2. **Índices Únicos Parciales (Prevención de Duplicados en PostgreSQL)**:
   - `uq_enrollment_user_course`: `UNIQUE (id_user_platform, id_course) WHERE id_course IS NOT NULL`
   - `uq_enrollment_user_congress`: `UNIQUE (id_user_platform, id_congress) WHERE id_congress IS NOT NULL`
   Impide que un usuario se inscriba dos veces en el mismo evento, permitiendo la coexistencia limpia de valores nulos en PostgreSQL.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id` | `INTEGER` | `Optional[int]` | No | PK (Autoincrement) | `None` | Identificador único de inscripción |
| `id_user_platform` | `INTEGER` | `int` | No | FK (`users_platform.id`) | - | Usuario participante registrado |
| `id_course` | `INTEGER` | `Optional[int]` | Sí | FK (`course.id`) | `None` | Curso (exclusivo con `id_congress`) |
| `id_congress` | `INTEGER` | `Optional[int]` | Sí | FK (`congresos.id_congreso`) | `None` | Congreso (exclusivo con `id_course`) |
| `enrollment_date` | `TIMESTAMP` | `datetime` | No | - | `datetime.utcnow` | Marca temporal UTC del registro |
| `status` | `VARCHAR(50)` (SQLEnum) | `EnrollmentStatus` | No | - | `INTERESADO` | Estado actual de la inscripción |
| `payment_order_url` | `VARCHAR(500)` | `Optional[str]` | Sí | - | `None` | Enlace al PDF oficial de orden de pago UTA |
| `tariff_type` | `VARCHAR(50)` | `Optional[str]` | Sí | - | `'GENERAL'` | Tipo de tarifa (`ESTUDIANTE_UTA`, `PROFESIONAL`, `PONENTE`) |
| `amount` | `DOUBLE PRECISION` | `Optional[float]` | Sí | - | `0.00` | Monto liquidado en USD |

---

### 4.2 Modelo `SessionAttendance` (Tabla `session_attendance`)

- **Archivo Fuente**: `src/models/session_attendance_model.py`
- **Tipo de Definición**: SQLModel (`SQLModel, table=True`).
- **Migración Origen**: `2026_11_08_0415-c3f81e29a001_create_session_attendance_table.py`
- **Propósito Institucional**: Control de presencia en sala en tiempo real. Se alimenta del lector de códigos QR (óptico USB o cámara móvil) para acreditar el porcentaje mínimo de asistencia requerido para la certificación (80%).

#### Restricciones e Índices
- **Restricción de Unicidad Compuesta**: `CONSTRAINT uq_enrollment_session UNIQUE (id_enrollment, id_sesion)`. Un participante no puede marcar asistencia más de una vez en la misma sesión (el backend responde `409 Conflict`).
- **Índice de Aceleración**: `CREATE INDEX idx_attendance_session ON session_attendance(id_sesion)`. Optimiza las consultas de aforo de sala y reportes por conferencia.
- **Integridad Referencial con Cascada**: Claves foráneas vinculadas con `ON DELETE CASCADE` hacia `enrollments` y `sesiones_cronograma`.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id` | `SERIAL` (`INTEGER`) | `Optional[int]` | No | PK (Autoincrement) | `None` | Identificador de marcaje de asistencia |
| `id_enrollment` | `INTEGER` | `int` | No | FK (`enrollments.id`), UQ | - | Inscripción activa del participante |
| `id_sesion` | `INTEGER` | `int` | No | FK (`sesiones_cronograma.id_sesion`), UQ | - | Sesión académica escaneada |
| `check_in_time` | `TIMESTAMP WITHOUT TIME ZONE` | `datetime` | No | - | `(NOW() AT TIME ZONE 'utc')` | Estampa de tiempo exacta de escaneo |
| `scanned_by_user_id` | `INTEGER` | `Optional[int]` | Sí | - | `None` | ID del operador o admin que realizó el escaneo |
| `notes` | `VARCHAR(255)` | `Optional[str]` | Sí | - | `None` | Observaciones operativas o incidencias |

---

## 5. Módulo de Calidad Académica e Investigación

### 5.1 Modelo `ScientificPaper` (Tabla `scientific_papers`)

- **Archivo Fuente**: `src/models/scientific_paper_model.py`
- **Tipo de Definición**: SQLModel (`SQLModel, table=True`).
- **Propósito Institucional**: Gestión del Call for Papers (CFP) para congresos científicos internacionales (CSEI). Permite a los autores postular artículos en formato PDF y a los revisores dictaminar su aceptación o rechazo.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id` | `INTEGER` | `Optional[int]` | No | PK (Autoincrement) | `None` | Identificador único del artículo científico |
| `id_congress` | `INTEGER` | `int` | No | FK (`congresos.id_congreso`) | - | Congreso científico al que postula |
| `id_user_platform` | `INTEGER` | `int` | No | FK (`users_platform.id`) | - | Autor de correspondencia |
| `title` | `VARCHAR(300)` | `str` | No | - | - | Título completo del artículo científico |
| `abstract` | `TEXT` | `str` | No | - | - | Resumen del paper en español/inglés |
| `track` | `VARCHAR(100)` | `str` | No | - | - | Eje temático (ej. 'Inteligencia Artificial', 'Ciberseguridad') |
| `keywords` | `VARCHAR(255)` | `Optional[str]` | Sí | - | `None` | Palabras clave separadas por comas |
| `file_url` | `VARCHAR(500)` | `str` | No | - | - | Ruta o URL al archivo PDF del manuscrito |
| `status` | `VARCHAR(50)` | `str` | No | - | `'ENVIADO'` | Estado del artículo (`ENVIADO`, `EN_REVISION`, `ACEPTADO`, `RECHAZADO`) |
| `reviewer_comments` | `TEXT` | `Optional[str]` | Sí | - | `None` | Retroalimentación de los pares evaluadores |
| `submitted_at` | `TIMESTAMP` | `datetime` | No | - | `datetime.utcnow` | Fecha y hora de envío del paper |
| `updated_at` | `TIMESTAMP` | `datetime` | No | - | `datetime.utcnow` | Fecha y hora de la última revisión |

---

### 5.2 Modelo `CacesSurvey` (Tabla `caces_surveys`)

- **Archivo Fuente**: `src/models/caces_survey_model.py`
- **Tipo de Definición**: SQLModel (`SQLModel, table=True`).
- **Propósito Institucional**: Instrumento de evaluación de calidad y satisfacción académica exigido por el Consejo de Aseguramiento de la Calidad de la Educación Superior (CACES) de Ecuador.

#### Estructura de Preguntas Likert (Escala 1 a 5)
- `q1_rating`: Evaluación del contenido científico y pertinencia técnica de las exposiciones.
- `q2_rating`: Evaluación de la organización, puntualidad e infraestructura física/tecnológica.
- `q3_rating`: Evaluación del desempeño didáctico y dominio de los ponentes/instructores.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id` | `INTEGER` | `Optional[int]` | No | PK (Autoincrement) | `None` | Identificador de la encuesta respondida |
| `id_congress` | `INTEGER` | `Optional[int]` | Sí | FK (`congresos.id_congreso`) | `None` | Congreso evaluado |
| `id_course` | `INTEGER` | `Optional[int]` | Sí | FK (`course.id`) | `None` | Curso evaluado (si aplica) |
| `id_user_platform` | `INTEGER` | `int` | No | FK (`users_platform.id`) | - | Participante encuestado |
| `q1_rating` | `INTEGER` | `int` | No | - | - | Calificación de pertinencia académica (1-5) |
| `q2_rating` | `INTEGER` | `int` | No | - | - | Calificación de organización y logística (1-5) |
| `q3_rating` | `INTEGER` | `int` | No | - | - | Calificación de calidad de ponentes (1-5) |
| `comments` | `TEXT` | `Optional[str]` | Sí | - | `None` | Sugerencias u observaciones cualitativas |
| `submitted_at` | `TIMESTAMP` | `datetime` | No | - | `datetime.utcnow` | Momento de envío de la encuesta |

---

## 6. Módulo de Configuración de Portal y Contenido

### 6.1 Modelo `Post` (Tabla `posts`)

- **Archivo Fuente**: `src/models/post.py`
- **Tipo de Definición**: SQLModel (`SQLModel, table=True`).
- **Propósito Institucional**: Configuración dinámica de la página de inicio (Landing Page) de `CTT-WEB`, almacenando en formato JSON la estructura jerárquica de los tres banners institucionales de la portada.

#### Estructura de Campos

| Campo | Tipo SQL | Python Type | Nulo | Clave / Índice | Valor por Defecto | Descripción |
|---|---|---|---|---|---|---|
| `id` | `INTEGER` | `Optional[int]` | No | PK (Autoincrement) | `None` | Identificador del bloque de configuración |
| `config` | `JSON` | `dict` | Sí | - | `{}` | Estructura JSON con imágenes, títulos, subtítulos y botones de los banners |
