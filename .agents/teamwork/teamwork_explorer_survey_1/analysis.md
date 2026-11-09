# Reporte de Exploración de Arquitectura Backend (`Back-CTT`)

**Fecha**: 2026-11-08  
**Agente**: `teamwork_explorer` (`teamwork_explorer_survey_1`)  
**Misión**: Auditoría integral de arquitectura, esquemas, controladores, rutas e infraestructura de pruebas de `Back-CTT` en relación con las 3 Oleadas del Plan Maestro Ecosistema Empresarial CTT-FISEI.

---

## 1. Resumen Ejecutivo de Hallazgos

| Dimensión Auditada | Estado Actual en `Back-CTT` | Brecha a Implementar (Wave 1, 2, 3) | Archivos Clave Involucrados |
| :--- | :--- | :--- | :--- |
| **1. Control de Aforo (`cupo_maximo`)** | El campo `cupo_maximo` existe en el modelo `Congress` (`congresos.cupo_maximo`, default 250). Sin embargo, `EnrollmentController.create_enrollment` **no valida** cupos activos. | Agregar conteo de estados activos (`PAGADO`, `FACTURADO`, `INTERESADO`, `ORDEN_GENERADA`) y arrojar `400 Bad Request` si se alcanza `cupo_maximo`. | `src/models/congress_model.py:31`, `src/controllers/enrollment_controller.py:40-105` |
| **2. Asistencia QR (`session_attendance`)** | Existe la tabla `sesiones_cronograma` (`SesionCronograma`). **No existe** tabla ni modelo `session_attendance` ni endpoint de escaneo QR. Frontend genera QR con formato `UTA-FISEI-PASS-{enrollment.id}-{congress_id}`. | Crear modelo/tabla `session_attendance` con constraint único `(id_enrollment, id_sesion)`, endpoints `POST /attendance/scan` (manejo de `409 Conflict`) y `GET /attendance/session/{id_sesion}`. | `src/models/sesion_cronograma_model.py:13`, `src/routes/congress_router.py`, `CTT-WEB/src/components/ui/DigitalPassModal.jsx:67` |
| **3. Perfil de Usuario y Datos Fiscales SRI** | La tabla `users_platform` **ya cuenta** con los 5 campos SRI (`billing_identification`, `billing_name`, `billing_address`, `billing_phone`, `billing_email`). Ya existe `PUT /api/v1/users/{user_id}/billing`. | Implementar actualización de datos personales (`PUT /api/v1/users/{user_id}/profile`) con cédula bloqueada contra suplantación y cambio seguro de contraseña con `bcrypt` (`PUT /api/v1/users/{user_id}/password`). | `src/models/user_platform.py:31-55`, `src/routes/users_platform_router.py:21-58`, `src/routes/platform_auth_router.py:75-93` |
| **4. Generador RIDE PDF UTA** | Las librerías `reportlab 5.1.0`, `qrcode 8.2` y `pillow 13.0.0` **ya están instaladas** en el entorno Python del contenedor backend (`ctt-backend`). No existe generador RIDE ni endpoint de descarga. | Implementar módulo de renderizado PDF estructurado según SRI (RUC `1860000880001`, desglose aranceles $20/$50/$80, IVA 0% exento, QR de autenticidad) y endpoint `GET /api/v1/enrollments/{id}/ride/download`. | `src/utils/pdf_utils.py`, `src/routes/enrollments_router.py` |
| **5. Exportador & Sync Moodle LMS** | Inexistente en la base de código. | Implementar endpoint `GET /api/v1/congresses/{id}/export/moodle-csv` con esquema `username,firstname,lastname,email,course1,role1` para inscritos `PAGADO`/`FACTURADO` y cliente REST opcional para `core_enrol_manual_enrol_users`. | `src/routes/congress_router.py`, nuevo `src/utils/moodle_service.py` |
| **6. Servicio de Correo SMTP** | Inexistente (`.env` no tiene variables SMTP, no hay `email_service.py`). | Crear `src/utils/email_service.py` con 4 plantillas HTML responsive (orden generada, pago validado con pase QR, comprobante rechazado, recordatorio 24h) y modo fallback asíncrono/mock para desarrollo. | `src/config/`, `src/utils/email_service.py` |
| **7. CFP, Streaming Hub y Encuesta CACES** | Inexistentes en modelos y rutas. | Definir modelos `ScientificPaper` (tabla `scientific_papers`), hub de enlaces de streaming protegido por estado de pago, y `CACESQualitySurvey` (tabla `caces_surveys`) con 3 preguntas Likert (1-5) + comentarios. | Nuevos modelos en `src/models/`, routers en `src/routes/` |
| **8. Infraestructura de Pruebas** | Configurado con `pytest`, fixtures de BD SQLite en memoria (`StaticPool`, `PRAGMA foreign_keys=ON`) y `TestClient`. 531 tests pasando. | Crear la suite `Back-CTT/tests/test_enterprise_features.py` cubriendo control de aforo, escaneo QR de asistencia, RIDE PDF, exportación Moodle y perfil/password. | `Back-CTT/pytest.ini`, `Back-CTT/tests/conftest.py`, `Back-CTT/tests/test_enterprise_features.py` |

---

## 2. Detalle Exhaustivo por Dimensión

### 2.1 Enrollment Controller & Control de Aforo (`cupo_maximo`)

- **Modelo `Congress` (`src/models/congress_model.py`)**:
  - Línea 31: `cupo_maximo: Optional[int] = Field(default=250)` en tabla `congresos`.
  - Líneas 32-34: `costo_estudiante_uta` ($20), `costo_profesional` ($50), `costo_ponente` ($80) como `Optional[float] = Field(default=0.00)`.
- **Modelo `Enrollment` (`src/models/enrollment.py`)**:
  - Tabla `enrollments` en PostgreSQL.
  - Campos: `id`, `id_user_platform`, `id_course`, `id_congress`, `enrollment_date`, `status` (`EnrollmentStatus`: `INTERESADO`, `ORDEN_GENERADA`, `PAGADO`, `FACTURADO`, `ANULADO`), `payment_order_url`, `tariff_type` (default "GENERAL"), `amount` (default 0.00).
  - Constraints: `chk_enrollment_course_xor_congress` (exactamente uno de `id_course` o `id_congress` debe ser no-nulo), e índices únicos parciales `uq_enrollment_user_course` y `uq_enrollment_user_congress`.
- **Implementación Actual en `EnrollmentController.create_enrollment` (`src/controllers/enrollment_controller.py:40-110`)**:
  - Valida que el recurso exista (`CongressRepository.get_congress_with_relations`).
  - Valida que el usuario exista (`UserPlatform`).
  - Valida duplicados (`check_existing_congress_enrollment`).
  - **Falta**: No existe comprobación de `cupo_maximo`. Tampoco se transfieren los campos `tariff_type` y `amount` de `EnrollmentCreate` hacia la instancia creada de `Enrollment`.
- **Intervención Requerida**:
  ```python
  if not is_course_enrollment:
      if resource.cupo_maximo and resource.cupo_maximo > 0:
          active_count = db.exec(
              select(func.count(Enrollment.id)).where(
                  Enrollment.id_congress == enrollment_data.id_congress,
                  Enrollment.status.in_([
                      EnrollmentStatus.PAGADO,
                      EnrollmentStatus.FACTURADO,
                      EnrollmentStatus.INTERESADO,
                      EnrollmentStatus.ORDEN_GENERADA
                  ])
              )
          ).one()
          if active_count >= resource.cupo_maximo:
              raise ValueError(f"El cupo máximo ({resource.cupo_maximo} participantes) ha sido alcanzado. Convocatoria cerrada.")
  ```

---

### 2.2 Control de Asistencia a Sesiones con QR (`session_attendance`)

- **Modelo de Sesión Existente (`src/models/sesion_cronograma_model.py`)**:
  - Tabla: `sesiones_cronograma` (nótese el plural en la base de datos PostgreSQL real).
  - Clave primaria: `id_sesion: Optional[int] = Field(default=None, primary_key=True)`.
  - Llaves foráneas: `id_congreso` (`congresos.id_congreso`), `id_speaker` (`speakers.id_speaker`).
  - Campos: `fecha`, `hora_inicio`, `hora_fin`, `titulo_sesion`, `jornada`, `lugar`.
- **Estructura del Payload QR**:
  - Generado en `CTT-WEB/src/components/ui/DigitalPassModal.jsx`:
    ```javascript
    const passCode = `UTA-FISEI-PASS-${enrollment.id}-${enrollment.id_congress || 1}`;
    ```
  - Formato del QR encodeado:
    - String directo: `"UTA-FISEI-PASS-{enrollment_id}-{id_congreso}"`
    - O JSON embebido: `{"credential": "UTA-FISEI-PASS-...", "attendee": "...", "identification": "...", "congress_id": ..., "status": "PAGADO"}`.
  - El backend debe admitir robustamente ambos formatos extrayendo `enrollment_id` y `id_congreso`.
- **Esquema de Base de Datos Necesario**:
  ```python
  class SessionAttendance(SQLModel, table=True):
      __tablename__ = "session_attendance"
      __table_args__ = (
          UniqueConstraint("id_enrollment", "id_sesion", name="uq_enrollment_session"),
          Index("idx_attendance_session", "id_sesion"),
      )
      id: Optional[int] = Field(default=None, primary_key=True)
      id_enrollment: int = Field(foreign_key="enrollments.id", nullable=False)
      id_sesion: int = Field(foreign_key="sesiones_cronograma.id_sesion", nullable=False)
      check_in_time: datetime = Field(default_factory=datetime.utcnow, nullable=False)
      scanned_by_user_id: Optional[int] = Field(default=None, nullable=True)
      notes: Optional[str] = Field(default=None, max_length=255)
  ```
- **Endpoints a Exponer en `src/routes/congress_router.py`**:
  - `POST /api/v1/congresses/{id_congreso}/attendance/scan`:
    - Valida que la inscripción pertenezca al congreso y tenga estado `PAGADO` o `FACTURADO`.
    - Detecta duplicados para la misma sesión: arroja `HTTP 409 Conflict` con timestamp del escaneo previo.
    - Inserta registro y retorna `{"status": "OK", "attendee_name": "...", "tariff_type": "...", "total_sessions_attended": X}`.
  - `GET /api/v1/congresses/{id_congreso}/attendance/session/{id_sesion}`: lista en vivo con nombres, cédula, arancel y hora de check-in.

---

### 2.3 Perfil de Usuario y Datos Fiscales SRI

- **Modelo `UserPlatform` (`src/models/user_platform.py`)**:
  - Tabla `users_platform` en PostgreSQL.
  - Líneas 31-36: Ya incluye columnas SRI:
    - `billing_identification: str | None = Field(default=None, max_length=20)`
    - `billing_name: str | None = Field(default=None, max_length=200)`
    - `billing_address: str | None = Field(default=None, max_length=255)`
    - `billing_phone: str | None = Field(default=None, max_length=20)`
    - `billing_email: str | None = Field(default=None, max_length=320)`
  - Se verificó directamente en la base de datos PostgreSQL en vivo (`\d users_platform`) y **todas las 5 columnas existen**.
- **Hashing de Contraseñas**:
  - Implementado con `bcrypt`:
    ```python
    def verify_password(self, plain_password: str) -> bool:
        password_bytes = plain_password[:72].encode('utf-8')
        hashed_bytes = self.password.encode('utf-8')
        return bcrypt.checkpw(password_bytes, hashed_bytes)

    @staticmethod
    def hash_password(password: str) -> str:
        password_bytes = password[:72].encode('utf-8')
        salt = bcrypt.gensalt()
        hashed = bcrypt.hashpw(password_bytes, salt)
        return hashed.decode('utf-8')
    ```
- **Rutas Existentes y Faltantes**:
  - `PUT /api/v1/users/{user_id}/billing` **ya existe** en `src/routes/users_platform_router.py:21-58`.
  - **Falta**:
    - `PUT /api/v1/users/{user_id}/profile`: Actualización de datos personales (`first_name`, `second_name`, `first_last_name`, `second_last_name`, `cellphone`, `address`). Debe bloquear la modificación de `identification` (cédula institucional UTA protegida contra suplantación).
    - `PUT /api/v1/users/{user_id}/password`: Cambio seguro de contraseña exigiendo validación de contraseña actual antes de actualizar con nuevo hash `bcrypt`.

---

### 2.4 Generación de RIDE PDF Oficial UTA

- **Librerías de Renderizado en Entorno**:
  - `reportlab 5.1.0` está instalado y activo en `ctt-backend`.
  - `qrcode 8.2` y `pillow 13.0.0` están instalados y activos.
- **Datos Institucionales Requeridos por SRI**:
  - Razón Social: *Universidad Técnica de Ambato - FISEI (CTT)*
  - RUC Institucional: `1860000880001`
  - Dirección Matriz: *Campus Huachi, Av. Los Chasquis y Río Payamino, Ambato*
  - Número de Comprobante: `001-002-{enrollment.id:09d}`
  - Desglose: Concepto, Categoría arancelaria ($20 / $50 / $80), Subtotal, IVA 0% (Servicios educativos exentos conforme Ley de Régimen Tributario Interno), Total a pagar.
  - Código QR de autenticidad contable integrado.
- **Ruta a Exponer**:
  - `GET /api/v1/enrollments/{enrollment_id}/ride/download`:
    - Valida que la inscripción tenga estado `PAGADO` o `FACTURADO`.
    - Genera el PDF en memoria mediante `io.BytesIO` y `reportlab.platypus` (`SimpleDocTemplate`, `Table`, `Paragraph`).
    - Retorna `StreamingResponse` con `media_type="application/pdf"` y header `Content-Disposition: inline; filename="RIDE_UTA_{enrollment_id}.pdf"`.

---

### 2.5 Exportador y Sincronización Moodle LMS

- **CSV de Matrícula Masiva para Moodle**:
  - Especificación Moodle Oficial: encabezados `username,firstname,lastname,email,course1,role1`.
  - Lógica de asignación:
    - `username`: Cédula del estudiante o prefijo de email institucional.
    - `firstname`: `user.first_name`
    - `lastname`: `user.first_last_name`
    - `email`: `user.email`
    - `course1`: Código corto de curso o edición del congreso (ej: `congress.edicion` o `congress.nombre`).
    - `role1`: `student` (o `editingteacher` si arancel es Ponente).
    - Filtro: Solo inscripciones con estado `PAGADO` o `FACTURADO`.
- **Ruta a Exponer**:
  - `GET /api/v1/congresses/{id_congreso}/export/moodle-csv`:
    - Retorna `StreamingResponse` con `text/csv` y `Content-Disposition: attachment; filename="moodle_enrolment_congreso_{id_congreso}.csv"`.
- **Sincronización Opcional REST**:
  - Si `MOODLE_URL` y `MOODLE_TOKEN` están configurados en `.env`, llamada HTTP a `core_enrol_manual_enrol_users`. Si no, fallback transparente que indica sincronización lista vía CSV.

---

### 2.6 Motor de Notificaciones SMTP

- **Estado Actual**:
  - No existe configuración SMTP en `.env`.
  - No existe `email_service.py` en `Back-CTT/src/utils/`.
- **4 Momentos Transaccionales**:
  1. *Orden Generada*: Detalles de cuenta corriente bancaria UTA y comprobante de inscripción.
  2. *Pago Validado*: Confirmación oficial y enlace directo al pase QR digital.
  3. *Comprobante Rechazado / Observado*: Justificación del rechazo emitida por tesorería e instrucciones para volver a cargar el comprobante legible.
  4. *Recordatorio 24 Horas*: Indicaciones de acceso, auditorio y horario de apertura de puertas.
- **Manejo de Fallback**:
  - `EmailService` debe detectar si `SMTP_HOST` está vacío o no configurado. En dicho caso, registrar el envío en los logs de la aplicación sin abortar la transacción de la base de datos ni bloquear al usuario.

---

### 2.7 CFP, Hub de Streaming y Encuestas CACES (Wave 3)

1. **Call for Papers (`scientific_papers`)**:
   - Modelo `ScientificPaper`: `id`, `id_congress`, `id_user_platform`, `title`, `abstract`, `track`, `keywords`, `file_url`, `status` (`ENVIADO`, `EN_REVISION`, `APROBADO`, `RECHAZADO`), `reviewer_comments`, `submitted_at`, `updated_at`.
   - Lógica de negocio: Al pasar un paper a estado `APROBADO` en `CTT-ADMIN`, se actualiza la inscripción del usuario al arancel de Ponente ($80).
2. **Hub de Streaming Híbrido**:
   - Campos en `Congress` o endpoint protegido: Enlace de streaming (Zoom, Teams, YouTube Live) y materiales descargables, accesible únicamente si `enrollment.status in ['PAGADO', 'FACTURADO']`.
3. **Encuesta de Calidad CACES / UTA (`caces_surveys`)**:
   - Modelo `CACESQualitySurvey`: `id`, `id_congress`, `id_user_platform`, `rating_academic_content` (1 a 5), `rating_speakers` (1 a 5), `rating_organization` (1 a 5), `comments`, `created_at`.
   - Constraint único `(id_congress, id_user_platform)` para evitar duplicidad de encuestas por participante.

---

### 2.8 Infraestructura de Pruebas (`tests/`)

- **Estructura y Fixtures (`tests/conftest.py`)**:
  - Base de datos de prueba: SQLite en memoria `sqlite:///:memory:` con `StaticPool` y `PRAGMA foreign_keys=ON`.
  - Comparte metadata entre SQLAlchemy y SQLModel (`SQLModel.metadata.create_all(engine)`).
  - Fixtures disponibles: `session`, `client` (FastAPI `TestClient` con dependency override de `get_db`), `sample_congress`, `sample_user_platform`, `sample_category`, `sample_course`.
- **Estado de Ejecución**:
  - En la suite general existen 533 tests. 531 tests pasan exitosamente en el entorno del contenedor.
- **Requerimiento del Plan Maestro**:
  - Crear `Back-CTT/tests/test_enterprise_features.py` con cobertura completa para:
    1. Límite de cupo (`cupo_maximo`) arrojando excepción / 400.
    2. Escaneo QR de asistencia a sesión registrando check-in y bloqueando duplicados con 409 Conflict.
    3. Generación de RIDE PDF oficial UTA con RUC `1860000880001`.
    4. Exportación de CSV para Moodle con columnas oficiales.
    5. Actualización de perfil y cambio de contraseña con `bcrypt`.

---

## 3. Conclusión de Factibilidad para la Implementación

El backend de `Back-CTT` posee una arquitectura limpia y desacoplada basada en FastAPI, SQLModel y repositorios. Los requerimientos de las Oleadas 1, 2 y 3 encajan orgánicamente en la estructura existente:
- Las columnas fiscales SRI ya residen en la base de datos PostgreSQL real.
- Las librerías críticas (`reportlab`, `qrcode`, `pillow`, `bcrypt`) ya están instaladas y disponibles en el entorno de ejecución.
- La ejecución de pruebas unitarias sobre SQLite en memoria permite validar rápidamente toda la nueva funcionalidad en `tests/test_enterprise_features.py`.
