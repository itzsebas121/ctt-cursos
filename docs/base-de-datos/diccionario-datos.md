# Diccionario de Datos - Ecosistema CTT FISEI

Este documento constituye el diccionario formal de datos de la base de datos relacional PostgreSQL 16 del sistema CTT FISEI. Se detallan todas las columnas, tipos nativos de PostgreSQL, restricciones de nulabilidad, llaves (primarias, foráneas, únicas, índices), valores por defecto y justificación funcional institucional para cada una de las 17 tablas físicas.

---

## 1. Tabla `users`

Tabla que almacena las credenciales y perfiles de los usuarios administrativos de la plataforma CTT FISEI.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id` | `INTEGER` | No | PK | `nextval('users_id_seq')` | Clave primaria autoincremental de la cuenta de administrador. |
| `name` | `VARCHAR(100)` | No | - | Ninguno | Nombres de pila del administrador o docente coordinador. |
| `last_name` | `VARCHAR(100)` | No | - | Ninguno | Apellidos completos del administrador. |
| `email` | `VARCHAR(320)` | No | UQ, IDX (`ix_users_email`) | Ninguno | Correo electrónico institucional único. Longitud máxima compatible con RFC 5321. |
| `password` | `VARCHAR(255)` | No | - | Ninguno | Hash de contraseña generado mediante algoritmo bcrypt con salt individual. |

---

## 2. Tabla `users_platform`

Directorio de usuarios registrados en el portal público estudiantil y profesional `CTT-WEB`.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id` | `INTEGER` | No | PK | `nextval('users_platform_id_seq')` | Identificador único de usuario de plataforma. |
| `identification` | `VARCHAR(20)` | No | - | Ninguno | Número de cédula ecuatoriana, RUC de persona natural o pasaporte. |
| `first_name` | `VARCHAR(100)` | No | - | Ninguno | Primer nombre del usuario. |
| `second_name` | `VARCHAR(100)` | Sí | - | `NULL` | Segundo nombre del usuario (opcional). |
| `first_last_name` | `VARCHAR(100)` | No | - | Ninguno | Primer apellido del usuario. |
| `second_last_name` | `VARCHAR(100)` | Sí | - | `NULL` | Segundo apellido del usuario (opcional). |
| `cellphone` | `VARCHAR(15)` | No | - | Ninguno | Teléfono celular de contacto con código de área o formato nacional. |
| `email` | `VARCHAR(320)` | No | UQ, IDX (`ix_users_platform_email`) | Ninguno | Correo electrónico principal para autenticación y notificaciones SMTP. |
| `address` | `VARCHAR(255)` | Sí | - | `NULL` | Dirección de domicilio del participante. |
| `type` | `VARCHAR(50)` | No | - | Ninguno | Tipo de usuario según Enum (`ESTUDIANTE`, `EXTERNO`, `ADMINISTRATIVO`). |
| `password` | `VARCHAR(255)` | No | - | Ninguno | Hash criptográfico bcrypt para inicio de sesión en `CTT-WEB`. |
| `billing_identification` | `VARCHAR(20)` | Sí | - | `NULL` | RUC o cédula autorizada para la emisión de comprobantes RIDE SRI. |
| `billing_name` | `VARCHAR(200)` | Sí | - | `NULL` | Razón social o nombres fiscales que constarán en la factura electrónica. |
| `billing_address` | `VARCHAR(255)` | Sí | - | `NULL` | Dirección física tributaria para el Servicio de Rentas Internas (SRI). |
| `billing_phone` | `VARCHAR(20)` | Sí | - | `NULL` | Teléfono tributario de contacto para la factura electrónica. |
| `billing_email` | `VARCHAR(320)` | Sí | - | `NULL` | Correo electrónico donde el SRI y la UTA envían el XML y PDF de la factura. |

---

## 3. Tabla `congresos`

Registro maestro de congresos científicos internacionales promovidos por la FISEI UTA.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id_congreso` | `INTEGER` | No | PK | `nextval('congresos_id_congreso_seq')` | Clave primaria institucional del congreso. |
| `nombre` | `VARCHAR(200)` | No | - | Ninguno | Denominación oficial del evento científico (ej. VII CSEI 2026). |
| `edicion` | `VARCHAR(10)` | No | - | Ninguno | Identificador de edición académica (ej. `VII-2026`, `CISED-2025`). |
| `anio` | `INTEGER` | No | - | Ninguno | Año calendario de realización del congreso (validado entre 2000 y 2100). |
| `fecha_inicio` | `DATE` | No | - | Ninguno | Fecha de apertura oficial del congreso (`YYYY-MM-DD`). |
| `fecha_fin` | `DATE` | No | - | Ninguno | Fecha de clausura oficial del congreso (`YYYY-MM-DD`). |
| `descripcion_general` | `TEXT` | No | - | Ninguno | Resumen temático, indexación en Scopus/IEEE y alcance institucional. |
| `poster_cover_url` | `VARCHAR(300)` | No | - | Ninguno | URL del afiche oficial y material gráfico promocional. |
| `lugar_sede` | `VARCHAR(150)` | Sí | - | `'Campus Huachi - FISEI, Ambato'` | Campus universitario o auditorio donde se celebra el evento. |
| `modalidad` | `VARCHAR(30)` | Sí | - | `'Presencial'` | Modalidad de participación: `Presencial`, `Híbrida` o `Virtual`. |
| `duracion_horas` | `INTEGER` | Sí | - | `40` | Carga horaria académica oficial certificada. |
| `sitio_web_oficial` | `VARCHAR(300)` | Sí | - | `''` | Enlace al portal web oficial indexado del congreso. |
| `cupo_maximo` | `INTEGER` | Sí | - | `250` | Aforo máximo de participantes permitido por normativa de seguridad. |
| `costo_estudiante_uta` | `DOUBLE PRECISION` | Sí | - | `0.00` | Tarifa subvencionada para alumnos regulares de la UTA ($20.00). |
| `costo_profesional` | `DOUBLE PRECISION` | Sí | - | `0.00` | Tarifa general para profesionales y participantes externos ($50.00). |
| `costo_ponente` | `DOUBLE PRECISION` | Sí | - | `0.00` | Tarifa para autores y expositores de artículos científicos ($80.00). |

---

## 4. Tabla `speakers`

Directorio de expositores, instructores y ponentes magistrales de congresos.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id_speaker` | `INTEGER` | No | PK | `nextval('speakers_id_speaker_seq')` | Clave primaria del ponente. |
| `id_congreso` | `INTEGER` | Sí | FK (`congresos.id_congreso`) | `NULL` | Referencia al congreso específico donde diserta el expositor. |
| `nombres_completos` | `VARCHAR(200)` | No | - | Ninguno | Nombres y apellidos completos del conferencista. |
| `titulo_academico` | `VARCHAR(50)` | No | - | Ninguno | Grado académico formal (ej. `PhD en Robótica`, `MSc en Redes`). |
| `institucion` | `VARCHAR(200)` | No | - | Ninguno | Universidad, centro de investigación o empresa internacional a la que pertenece. |
| `pais` | `VARCHAR(100)` | No | - | Ninguno | País de origen o residencia académica del expositor. |
| `foto_url` | `VARCHAR(300)` | No | - | Ninguno | Fotografía profesional de alta resolución para la agenda del congreso. |
| `tipo_speaker` | `VARCHAR(20)` | No | - | Ninguno | Clasificación del conferencista: `keynote`, `conferencia`, `taller` o `panel`. |

---

## 5. Tabla `sponsors`

Registro de marcas, empresas y entidades aliadas patrocinadoras de eventos FISEI.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id_sponsor` | `INTEGER` | No | PK | `nextval('sponsors_id_sponsor_seq')` | Clave primaria del patrocinador. |
| `nombre` | `VARCHAR(150)` | No | - | Ninguno | Nombre comercial o razón social de la entidad auspiciante. |
| `logo_url` | `VARCHAR(300)` | No | - | Ninguno | Enlace al logotipo oficial en alta resolución o vectorial. |
| `sitio_web` | `VARCHAR(200)` | No | - | Ninguno | Enlace al sitio corporativo oficial del sponsor. |
| `descripcion` | `TEXT` | No | - | Ninguno | Resumen institucional del patrocinador y sus servicios/productos. |
| `activo` | `BOOLEAN` | No | - | `true` | Indicador de vigencia operativa del sponsor (Alembic `98ea811add01`). |

---

## 6. Tabla `congreso_sponsors`

Tabla asociativa Many-to-Many entre congresos y empresas patrocinadoras.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id_congreso` | `INTEGER` | No | PK, FK (`congresos.id_congreso`) | Ninguno | Clave foránea al congreso patrocinado. Parte de la PK compuesta. |
| `id_sponsor` | `INTEGER` | No | PK, FK (`sponsors.id_sponsor`) | Ninguno | Clave foránea a la empresa auspiciante. Parte de la PK compuesta. |
| `categoria` | `VARCHAR(50)` | No | - | Ninguno | Rango de auspicio: `platino`, `oro`, `plata`, `bronce` o `colaborador`. |
| `aporte` | `NUMERIC(10, 2)` | No | - | Ninguno | Cuantía monetaria o valorización en USD de los aportes realizados. |

---

## 7. Tabla `sesiones_cronograma`

Estructura detallada del cronograma de ponencias, mesas redondas y talleres.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id_sesion` | `INTEGER` | No | PK | `nextval('sesiones_cronograma_id_sesion_seq')` | Clave primaria de la sesión de cronograma. |
| `id_congreso` | `INTEGER` | No | FK (`congresos.id_congreso`) | Ninguno | Congreso al que está vinculada la sesión. |
| `id_speaker` | `INTEGER` | No | FK (`speakers.id_speaker`) | Ninguno | Ponente asignado a dictar la sesión. |
| `fecha` | `DATE` | No | - | Ninguno | Fecha en la que tendrá lugar la exposición (`YYYY-MM-DD`). |
| `hora_inicio` | `TIME` | No | - | Ninguno | Hora de inicio programada (`HH:MM:SS`). |
| `hora_fin` | `TIME` | No | - | Ninguno | Hora de conclusión programada (`HH:MM:SS`). |
| `titulo_sesion` | `VARCHAR(200)` | No | - | Ninguno | Título temático formal de la conferencia magistral o ponencia. |
| `jornada` | `VARCHAR(20)` | No | - | Ninguno | Franja horaria de la jornada: `mañana`, `tarde` o `noche`. |
| `lugar` | `VARCHAR(150)` | No | - | Ninguno | Aula magna, auditorio o laboratorio donde se desarrolla la sesión. |

---

## 8. Tabla `session_attendance`

Registro transaccional de escaneos de código QR para control biométrico y de asistencia por sesión.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id` | `SERIAL` (`INTEGER`) | No | PK | `nextval('session_attendance_id_seq')` | Clave primaria del evento de marcaje. |
| `id_enrollment` | `INTEGER` | No | FK (`enrollments.id`), UQ (`uq_enrollment_session`) | Ninguno | Inscripción activa y pagada del asistente. Cascada al eliminar. |
| `id_sesion` | `INTEGER` | No | FK (`sesiones_cronograma.id_sesion`), UQ (`uq_enrollment_session`), IDX (`idx_attendance_session`) | Ninguno | Sesión a la que asiste el usuario. Cascada al eliminar. |
| `check_in_time` | `TIMESTAMP WITHOUT TIME ZONE` | No | - | `(NOW() AT TIME ZONE 'utc')` | Marca temporal UTC en la que se realizó la lectura óptica del QR. |
| `scanned_by_user_id` | `INTEGER` | Sí | - | `NULL` | ID del usuario administrativo u operador que operó el lector QR. |
| `notes` | `VARCHAR(255)` | Sí | - | `NULL` | Incidencias técnicas o notas de ingreso tardío o justificado. |

---

## 9. Tabla `categories`

Áreas de especialidad y líneas de conocimiento para cursos de capacitación continua.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id` | `INTEGER` | No | PK | `nextval('categories_id_seq')` | Clave primaria de la categoría de curso. |
| `name` | `VARCHAR(150)` | No | UQ, IDX (`ix_categories_name`) | Ninguno | Denominación temática única (ej. `TICS`, `Automatización`). |
| `description` | `VARCHAR(500)` | Sí | - | `NULL` | Alcance y perfil de las temáticas incluidas. |
| `svgurl` | `VARCHAR(500)` | Sí | - | `NULL` | Enlace al archivo vectorial SVG del icono temático. |
| `status` | `VARCHAR(50)` | No | - | `'ACTIVO'` | Estado operativo según Enum (`ACTIVO`, `INACTIVO`). |
| `created_at` | `TIMESTAMP` | No | - | `now()` | Estampa temporal de registro de la categoría. |
| `updated_at` | `TIMESTAMP` | Sí | - | `NULL` | Estampa temporal de la última actualización. |
| `created_by` | `INTEGER` | No | FK (`users.id`) | Ninguno | Usuario administrativo que creó la categoría. |

---

## 10. Tabla `congress_categories`

Clasificación temática de los congresos institucionales de la FISEI.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id` | `INTEGER` | No | PK | `nextval('congress_categories_id_seq')` | Clave primaria de la categoría de congreso. |
| `name` | `VARCHAR(150)` | No | UQ, IDX (`ix_congress_categories_name`) | Ninguno | Nombre del eje científico (ej. `Inteligencia Artificial`, `Robótica`). |
| `description` | `VARCHAR(500)` | Sí | - | `NULL` | Resumen de los temas y líneas de investigación abarcadas. |
| `svgurl` | `VARCHAR(500)` | Sí | - | `NULL` | Enlace al icono SVG ilustrativo. |
| `status` | `VARCHAR(50)` | No | - | `'ACTIVO'` | Estado de disponibilidad (`ACTIVO`, `INACTIVO`). |
| `created_at` | `TIMESTAMP` | No | - | `now()` | Fecha de creación del registro. |
| `updated_at` | `TIMESTAMP` | Sí | - | `NULL` | Fecha de modificación del registro. |
| `created_by` | `INTEGER` | No | FK (`users.id`) | Ninguno | Administrador responsable del alta del registro. |

---

## 11. Tabla `course`

Oferta formativa de cursos de transferencia y actualización técnica.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id` | `INTEGER` | No | PK | `nextval('course_id_seq')` | Clave primaria del curso. |
| `title` | `VARCHAR(200)` | No | - | Ninguno | Título formal del curso de capacitación continua. |
| `description` | `TEXT` | No | - | Ninguno | Descripción sintética del programa, metodología y perfil de egreso. |
| `place` | `VARCHAR(200)` | No | - | Ninguno | Ubicación de las clases (ej. Laboratorios Campus Huachi). |
| `course_image` | `VARCHAR(500)` | No | - | Ninguno | Imagen representativa utilizada en las tarjetas del catálogo. |
| `course_image_detail` | `VARCHAR(500)` | No | - | Ninguno | Banner horizontal para el encabezado de la ficha de detalle. |
| `category_id` | `INTEGER` | No | FK (`categories.id`), IDX (`ix_course_category_id`) | Ninguno | Categoría temática a la que pertenece el curso. |
| `status` | `VARCHAR(50)` | No | - | `'ACTIVO'` | Estado de oferta pública (`ACTIVO`, `INACTIVO`). |
| `objectives` | `TEXT` | No | - | Ninguno | Array de objetivos de aprendizaje serializado en JSON string. |
| `organizers` | `TEXT` | No | - | Ninguno | Lista de ingenieros coordinadores e instructores en JSON string. |
| `materials` | `TEXT` | No | - | Ninguno | Equipamiento y software requerido por el participante en JSON string. |
| `target_audience` | `TEXT` | No | - | Ninguno | Perfil sugerido de los postulantes en formato JSON string. |

---

## 12. Tabla `courserequirement`

Condiciones logísticas, de aforo, fechas de apertura y precios para un curso (Relación 1:1 con `course`).

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id` | `INTEGER` | No | PK | `nextval('courserequirement_id_seq')` | Clave primaria del registro de requerimientos. |
| `course_id` | `INTEGER` | No | FK (`course.id`), UQ | Ninguno | Relación 1:1 con el curso. Restricción única sobre la clave foránea. |
| `start_date_registration` | `DATE` | No | - | Ninguno | Fecha de apertura de inscripciones (`YYYY-MM-DD`). |
| `end_date_registration` | `DATE` | No | - | Ninguno | Fecha límite para registrarse en el curso. |
| `start_date_course` | `DATE` | No | - | Ninguno | Fecha de inicio de clases presenciales o virtuales. |
| `end_date_course` | `DATE` | No | - | Ninguno | Fecha de finalización del curso. |
| `days` | `TEXT` | No | - | Ninguno | Días de impartición serializados en JSON (ej. `["Lunes", "Miércoles"]`). |
| `start_time` | `TIME` | No | - | Ninguno | Hora de inicio diaria de las clases (`HH:MM:SS`). |
| `end_time` | `TIME` | No | - | Ninguno | Hora de cierre diario de las clases (`HH:MM:SS`). |
| `location` | `VARCHAR(200)` | No | - | Ninguno | Laboratorio especializado asignado (ej. Lab. de Redes 2). |
| `min_quota` | `INTEGER` | No | - | Ninguno | Aforo mínimo de alumnos para abrir el curso (típicamente 15). |
| `max_quota` | `INTEGER` | No | - | Ninguno | Aforo máximo de capacidad del laboratorio de cómputo. |
| `in_person_hours` | `INTEGER` | No | - | `0` | Horas cronológicas de docencia asistida presencial. |
| `autonomous_hours` | `INTEGER` | No | - | `0` | Horas estimadas de desarrollo de prácticas autónomas. |
| `modality` | `VARCHAR(100)` | No | - | Ninguno | Modalidad de dictado: `Presencial`, `Semipresencial` o `En línea`. |
| `certification` | `VARCHAR(200)` | No | - | Ninguno | Tipo de certificado emitido (ej. 'Aprobación por 40 horas académicas'). |
| `prerequisites` | `TEXT` | No | - | Ninguno | Requisitos académicos previos serializados en JSON string. |
| `prices` | `TEXT` | No | - | Ninguno | Estructura arancelaria por estamento (Estudiantes, Graduados, Externos) en JSON. |

---

## 13. Tabla `coursecontent`

Desglose de unidades curriculares y sílabo del curso (Relación 1:N con `course`).

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id` | `INTEGER` | No | PK | `nextval('coursecontent_id_seq')` | Clave primaria del módulo temático. |
| `course_id` | `INTEGER` | No | FK (`course.id`) | Ninguno | Clave foránea al curso educativo propietario. |
| `unit` | `VARCHAR(100)` | No | - | Ninguno | Código o nombre de la unidad didáctica (ej. `Unidad 1`). |
| `title` | `VARCHAR(200)` | No | - | Ninguno | Título temático principal de la unidad. |
| `topics` | `TEXT` | No | - | Ninguno | Desglose pormenorizado de tópicos y prácticas en JSON string. |

---

## 14. Tabla `enrollments`

Transacciones de inscripción, pagos y generación de comprobantes RIDE para congresos o cursos.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id` | `INTEGER` | No | PK | `nextval('enrollments_id_seq')` | Identificador único de la inscripción o matrícula. |
| `id_user_platform` | `INTEGER` | No | FK (`users_platform.id`), IDX | Ninguno | Participante matriculado en el evento o curso. |
| `id_course` | `INTEGER` | Sí | FK (`course.id`), IDX (`uq_enrollment_user_course`) | `NULL` | Curso seleccionado. Excluyente con `id_congress` vía CheckConstraint. |
| `id_congress` | `INTEGER` | Sí | FK (`congresos.id_congreso`), IDX (`uq_enrollment_user_congress`) | `NULL` | Congreso seleccionado. Excluyente con `id_course` vía CheckConstraint. |
| `enrollment_date` | `TIMESTAMP` | No | - | `utcnow()` | Fecha y hora UTC del registro de matrícula. |
| `status` | `VARCHAR(50)` | No | - | `'INTERESADO'` | Estado transaccional (`INTERESADO`, `ORDEN_GENERADA`, `PAGADO`, `FACTURADO`, `ANULADO`). |
| `payment_order_url` | `VARCHAR(500)` | Sí | - | `NULL` | URL del PDF oficial de la orden de pago UTA generada con ReportLab. |
| `tariff_type` | `VARCHAR(50)` | Sí | - | `'GENERAL'` | Tipo de tarifa aplicada (`ESTUDIANTE_UTA`, `PROFESIONAL`, `PONENTE`). |
| `amount` | `DOUBLE PRECISION` | Sí | - | `0.00` | Valor monetario a pagar en dólares de los Estados Unidos (USD). |

---

## 15. Tabla `scientific_papers`

Sistema Call for Papers (CFP) para postulación y arbitraje ciego de artículos científicos.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id` | `INTEGER` | No | PK | `nextval('scientific_papers_id_seq')` | Identificador único del artículo científico. |
| `id_congress` | `INTEGER` | No | FK (`congresos.id_congreso`) | Ninguno | Congreso al cual se postula la investigación. |
| `id_user_platform` | `INTEGER` | No | FK (`users_platform.id`) | Ninguno | Autor principal o de correspondencia del manuscrito. |
| `title` | `VARCHAR(300)` | No | - | Ninguno | Título completo y oficial de la ponencia científica. |
| `abstract` | `TEXT` | No | - | Ninguno | Resumen técnico del manuscrito estructurado según formato IMRyD. |
| `track` | `VARCHAR(100)` | No | - | Ninguno | Eje temático o mesa de trabajo (ej. `Inteligencia Artificial y Robótica`). |
| `keywords` | `VARCHAR(255)` | Sí | - | `NULL` | Descriptores y palabras clave normalizadas (Thesaurus IEEE). |
| `file_url` | `VARCHAR(500)` | No | - | Ninguno | Ubicación en el servidor del archivo PDF del artículo postulado. |
| `status` | `VARCHAR(50)` | No | - | `'ENVIADO'` | Dictamen editorial: `ENVIADO`, `EN_REVISION`, `ACEPTADO`, `RECHAZADO`. |
| `reviewer_comments` | `TEXT` | Sí | - | `NULL` | Observaciones y dictamen fundamentado de los revisores académicos. |
| `submitted_at` | `TIMESTAMP` | No | - | `utcnow()` | Marca temporal de la primera postulación formal. |
| `updated_at` | `TIMESTAMP` | No | - | `utcnow()` | Marca temporal de la última actualización de versión o dictamen. |

---

## 16. Tabla `caces_surveys`

Encuestas de calidad académica y percepción de servicios estudiantiles (Normativa CACES).

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id` | `INTEGER` | No | PK | `nextval('caces_surveys_id_seq')` | Clave primaria de la boleta de evaluación. |
| `id_congress` | `INTEGER` | Sí | FK (`congresos.id_congreso`) | `NULL` | Congreso evaluado por el participante (si aplica). |
| `id_course` | `INTEGER` | Sí | FK (`course.id`) | `NULL` | Curso evaluado por el participante (si aplica). |
| `id_user_platform` | `INTEGER` | No | FK (`users_platform.id`) | Ninguno | Participante que completó el cuestionario. |
| `q1_rating` | `INTEGER` | No | - | Ninguno | Calificación Likert (1-5 estrellas) sobre la calidad de los contenidos científicos. |
| `q2_rating` | `INTEGER` | No | - | Ninguno | Calificación Likert (1-5 estrellas) sobre la logística, tecnología y cumplimiento de horarios. |
| `q3_rating` | `INTEGER` | No | - | Ninguno | Calificación Likert (1-5 estrellas) sobre la competencia y claridad de los expositores. |
| `comments` | `TEXT` | Sí | - | `NULL` | Comentarios cualitativos libres de retroalimentación. |
| `submitted_at` | `TIMESTAMP` | No | - | `utcnow()` | Fecha y hora UTC del envío de la encuesta. |

---

## 17. Tabla `posts`

Almacenamiento de configuraciones dinámicas para el portal web institucional.

| Nombre del Campo | Tipo de Dato | Permite Nulo | Clave | Valor por Defecto | Descripción Técnica y Funcional Institucional |
|---|---|---|---|---|---|
| `id` | `INTEGER` | No | PK | `nextval('posts_id_seq')` | Clave primaria del registro de configuración. |
| `config` | `JSON` | Sí | - | `'{}'::json` | Estructura JSON con imágenes, enlaces, subtítulos y botones de los banners de portada. |
