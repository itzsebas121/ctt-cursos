# Catálogo Exhaustivo de Endpoints de la API REST

Este documento constituye la referencia técnica completa y exhaustiva de todos los endpoints disponibles en la API REST de `Back-CTT`. Todos los endpoints están bajo el prefijo raíz `/api/v1`.

---

## 1. Módulo de Autenticación Administrativa (`/api/v1/auth`)

Gestiona el acceso para coordinadores, personal administrativo y docentes con perfil de gestión en `CTT-ADMIN`.

### 1.1 Registrar Usuario Administrador
- **Método**: `POST`
- **Ruta**: `/api/v1/auth/register`
- **Autenticación**: Pública
- **Cuerpo de la Petición (`application/json`)**:
  ```json
  {
    "name": "Sebastián",
    "last_name": "Vallejo",
    "email": "sebastian.vallejo@uta.edu.ec",
    "password": "ClaveSegura2026!"
  }
  ```
- **Respuestas**:
  - `200 OK`: `{"message": "User registered successfully"}`
  - `400 Bad Request`: Si el email ya se encuentra registrado o los datos no cumplen las validaciones.

### 1.2 Obtener Token de Acceso Administrador
- **Método**: `POST`
- **Ruta**: `/api/v1/auth/token`
- **Autenticación**: Pública
- **Cuerpo de la Petición (`application/x-www-form-urlencoded`)**:
  - `username`: Correo electrónico institucional (`string`).
  - `password`: Contraseña del usuario (`string`).
- **Respuestas**:
  - `200 OK`:
    ```json
    {
      "access_token": "eyJhbGciOiJIUzI1NiIsIn...",
      "token_type": "bearer"
    }
    ```
  - `400 Bad Request`: `{"detail": "Incorrect username or password"}`

### 1.3 Perfil de Administrador Autenticado
- **Método**: `GET`
- **Ruta**: `/api/v1/auth/profile`
- **Autenticación**: Bearer Token (Admin)
- **Respuestas**:
  - `200 OK`:
    ```json
    {
      "id": 1,
      "name": "Sebastián",
      "last_name": "Vallejo",
      "email": "sebastian.vallejo@uta.edu.ec"
    }
    ```
  - `401 Unauthorized`: Token inválido o expirado.

---

## 2. Módulo de Autenticación de Plataforma (`/api/v1/platform-auth`)

Gestiona el registro e inicio de sesión de estudiantes y participantes en `CTT-WEB`.

### 2.1 Registro de Participante
- **Método**: `POST`
- **Ruta**: `/api/v1/platform-auth/register`
- **Autenticación**: Pública
- **Cuerpo de la Petición (`application/json`)**:
  ```json
  {
    "identification": "1801234567",
    "first_name": "Carlos",
    "second_name": "Andrés",
    "first_last_name": "Mora",
    "second_last_name": "Sánchez",
    "cellphone": "0991234567",
    "email": "carlos.mora@uta.edu.ec",
    "address": "Campus Huachi, Ambato",
    "type": "Estudiante",
    "password": "Password123!"
  }
  ```
- **Respuestas**:
  - `200 OK`: `{"message": "Platform user registered successfully"}`
  - `400 Bad Request`: `{"detail": "User with this email already exists"}` o `{"detail": "User with this identification already exists"}`.

### 2.2 Obtener Token de Participante
- **Método**: `POST`
- **Ruta**: `/api/v1/platform-auth/token`
- **Autenticación**: Pública
- **Cuerpo de la Petición (`application/x-www-form-urlencoded`)**:
  - `username`: Correo electrónico del participante.
  - `password`: Contraseña.
- **Respuestas**:
  - `200 OK`:
    ```json
    {
      "access_token": "eyJhbGciOiJIUzI1NiIsIn...",
      "token_type": "bearer"
    }
    ```
  - `400 Bad Request`: `{"detail": "Incorrect username or password"}`

### 2.3 Perfil del Participante Autenticado
- **Método**: `GET`
- **Ruta**: `/api/v1/platform-auth/profile`
- **Autenticación**: Bearer Token (Platform)
- **Respuestas**:
  - `200 OK`: Datos completos del usuario incluyendo información de facturación SRI.
  - `401 Unauthorized`: Token ausente o caducado.

---

## 3. Módulo de Gestión de Perfiles y Seguridad (`/api/v1/users`)

### 3.1 Actualizar Perfil de Usuario
- **Método**: `PUT`
- **Ruta**: `/api/v1/users/{user_id}/profile`
- **Autenticación**: Bearer Token (Usuario titular o Admin)
- **Parámetros de Ruta**: `user_id` (`integer`)
- **Cuerpo de la Petición (`application/json`)**:
  ```json
  {
    "first_name": "Carlos Actualizado",
    "last_name": "Mora Sánchez",
    "phone": "0998887766",
    "address": "Campus Huachi Bloque 2"
  }
  ```
- **Regla**: La cédula (`identification`) es estrictamente inmutable; cualquier intento de sobrescritura se ignora silenciosamente.
- **Respuestas**:
  - `200 OK`: `{"message": "Perfil actualizado", "data": {...}}`
  - `404 Not Found`: Si el usuario no existe.

### 3.2 Actualizar Datos de Facturación Electrónica (SRI)
- **Método**: `PUT`
- **Ruta**: `/api/v1/users/{user_id}/billing`
- **Autenticación**: Bearer Token
- **Cuerpo de la Petición (`application/json`)**:
  ```json
  {
    "billing_identification": "1801234567001",
    "billing_name": "Corporación Mora S.A.",
    "billing_address": "Av. Los Chasquis y Río Guayas",
    "billing_phone": "032888999",
    "billing_email": "facturacion@mora.com"
  }
  ```
- **Respuestas**:
  - `200 OK`: Datos fiscales guardados exitosamente.

### 3.3 Actualizar Contraseña con Bcrypt
- **Método**: `PUT`
- **Ruta**: `/api/v1/users/{user_id}/password`
- **Autenticación**: Bearer Token (El usuario debe coincidir con `user_id`)
- **Cuerpo de la Petición (`application/json`)**:
  ```json
  {
    "current_password": "PasswordAnterior123!",
    "new_password": "NuevaPasswordSegura2026!"
  }
  ```
- **Respuestas**:
  - `200 OK`: `{"message": "Contraseña actualizada exitosamente"}`
  - `400 Bad Request`: `{"detail": "Contraseña actual incorrecta"}`
  - `403 Forbidden`: Si un usuario intenta modificar la clave de otro usuario.

### 3.4 Obtener Perfil y Datos Fiscales
- **Método**: `GET`
- **Ruta**: `/api/v1/users/{user_id}/profile`
- **Autenticación**: Bearer Token

---

## 4. Módulo de Inscripciones y Pagos (`/api/v1/enrollments`)

### 4.1 Crear Inscripción
- **Método**: `POST`
- **Ruta**: `/api/v1/enrollments/`
- **Autenticación**: Bearer Token (Platform o Admin)
- **Cuerpo de la Petición (`application/json`)**:
  ```json
  {
    "id_user_platform": 1,
    "id_congress": 2,
    "id_course": null,
    "status": "Interesado",
    "tariff_type": "Estudiante UTA",
    "amount": 20.0
  }
  ```
- **Validaciones de Negocio**:
  - Verifica que no exista una inscripción activa previa para el mismo curso o congreso.
  - Verifica que `active_count < congresos.cupo_maximo`. Si se llena el aforo, lanza `400 Bad Request`.
- **Respuestas**:
  - `201 Created`: `{"message": "Inscripción creada exitosamente.", "data": {...}}`
  - `400 Bad Request`: `"El cupo máximo (N participantes) ha sido alcanzado. Convocatoria cerrada."`

### 4.2 Listar Inscripciones Paginadas
- **Método**: `GET`
- **Ruta**: `/api/v1/enrollments/`
- **Query Params**: `page` (`default: 1`), `page_size` (`default: 10`), `status` (`optional`), `tariff_type` (`optional`)

### 4.3 Inscripciones de un Usuario Específico
- **Método**: `GET`
- **Ruta**: `/api/v1/enrollments/user/{user_id}`

### 4.4 Inscripciones por Congreso (Admin)
- **Método**: `GET`
- **Ruta**: `/api/v1/enrollments/congress/{congress_id}`

### 4.5 Generar Orden de Pago Institucional UTA
- **Método**: `POST`
- **Ruta**: `/api/v1/enrollments/{enrollment_id}/uta/generate-order`
- **Autenticación**: Bearer Token
- **Respuestas**:
  - `200 OK`: Actualiza el estado a `Generada la orden`, genera el identificador `UTA-FISEI-2026-XXXXXX` y despacha el correo transaccional.

### 4.6 Descargar Orden de Pago en PDF
- **Método**: `GET`
- **Ruta**: `/api/v1/enrollments/{enrollment_id}/payment-order/download`
- **Respuestas**:
  - `200 OK`: Archivo binario `application/pdf` con la Orden de Pago UTA.

### 4.7 Descargar Comprobante / RIDE Oficial SRI
- **Método**: `GET`
- **Ruta**: `/api/v1/enrollments/{enrollment_id}/ride/download`
- **Autenticación**: Bearer Token
- **Respuestas**:
  - `200 OK`: Archivo binario `application/pdf` con RUC `1860000880001`, clave de 49 dígitos y tarifa 0% IVA.
  - `400 Bad Request`: Si la inscripción no se encuentra en estado `Pagado` o `Facturado`.

### 4.8 Actualizar Inscripción (Aprobar Pago / Subir Comprobante)
- **Método**: `PUT`
- **Ruta**: `/api/v1/enrollments/{enrollment_id}`
- **Cuerpo de la Petición (`application/json`)**:
  ```json
  {
    "status": "Pagado",
    "payment_order_url": "https://storage.uta.edu.ec/comprobantes/deposito_123.jpg",
    "amount": 20.0
  }
  ```

---

## 5. Módulo de Congresos (`/api/v1/congresses` y `/api/v1/congresses-new`)

> **Nota Técnica**: Por retrocompatibilidad con integraciones previas, todas las rutas de este módulo responden indistintamente bajo el prefijo canónico `/api/v1/congresses-new` o el alias `/api/v1/congresses`.

### 5.1 Listar Congresos
- **Método**: `GET`
- **Ruta**: `/api/v1/congresses-new` o `/api/v1/congresses`
- **Autenticación**: Pública
- **Query Params**: `page`, `page_size`, `anio`, `search`

### 5.2 Obtener Detalle de un Congreso por ID
- **Método**: `GET`
- **Ruta**: `/api/v1/congresses-new/{congress_id}`

### 5.3 Registrar Asistencia por Escaneo QR
- **Método**: `POST`
- **Ruta**: `/api/v1/congresses-new/{id_congreso}/attendance/scan`
- **Autenticación**: Bearer Token (Admin / Operador de acceso)
- **Cuerpo de la Petición (`application/json`)**:
  ```json
  {
    "qr_payload": "UTA-FISEI-PASS-12-1",
    "id_sesion": 8,
    "notes": "Acceso puerta principal auditorio"
  }
  ```
- **Respuestas**:
  - `200 OK`:
    ```json
    {
      "status": "OK",
      "attendee_name": "Carlos Mora Sánchez",
      "tariff_type": "Estudiante UTA",
      "total_sessions_attended": 3
    }
    ```
  - `400 Bad Request`: Si el QR es ilegible, no pertenece al congreso o la inscripción no está pagada.
  - `409 Conflict`:
    ```json
    {
      "detail": "El participante ya registró asistencia a esta sesión el 2026-11-10T09:15:00"
    }
    ```

### 5.4 Consultar Asistencias de una Sesión en Vivo
- **Método**: `GET`
- **Ruta**: `/api/v1/congresses-new/{id_congreso}/attendance/session/{id_sesion}`
- **Respuestas**:
  - `200 OK`: Total de asistentes y lista nominal con horas exactas de ingreso.

### 5.5 Resumen Financiero y KPIs del Congreso
- **Método**: `GET`
- **Ruta**: `/api/v1/congresses-new/{id_congreso}/financial-summary`
- **Autenticación**: Bearer Token (Admin)
- **Respuestas**:
  - `200 OK`:
    ```json
    {
      "id_congreso": 1,
      "nombre_congreso": "CIIA 2026",
      "total_revenue": 4500.0,
      "total_budget": 5000.0,
      "total_active_enrollments": 120,
      "total_paid_enrollments": 95,
      "conversion_rate": 79.17,
      "tariffs_breakdown": {
        "estudiante_uta": {"count": 60, "revenue": 1200.0, "unit_price": 20.0},
        "profesional": {"count": 25, "revenue": 1250.0, "unit_price": 50.0},
        "ponente": {"count": 10, "revenue": 800.0, "unit_price": 80.0}
      }
    }
    ```

### 5.6 Exportación de Participantes en Formato Moodle LMS
- **Método**: `GET`
- **Ruta**: `/api/v1/congresses-new/{id_congreso}/export/moodle-csv`
- **Autenticación**: Bearer Token (Admin)
- **Respuestas**:
  - `200 OK`: Archivo CSV con cabeceras `username,firstname,lastname,email,course1,role1`.

### 5.7 Postulación de Artículo Científico (Call for Papers)
- **Método**: `POST`
- **Ruta**: `/api/v1/congresses-new/{id_congreso}/papers`
- **Autenticación**: Bearer Token
- **Cuerpo de la Petición (`application/json`)**:
  ```json
  {
    "title": "Machine Learning Applied to Smart Grid Security",
    "abstract": "This research presents a novel architecture...",
    "track": "Inteligencia Artificial y Seguridad",
    "keywords": "AI, Smart Grids, Cybersecurity",
    "file_url": "https://storage.uta.edu.ec/papers/csei_2026_paper_4.pdf"
  }
  ```
- **Respuestas**:
  - `200 OK`: Registro del paper en estado `ENVIADO`.

### 5.8 Dictaminar / Calificar Artículo Científico
- **Método**: `PUT`
- **Ruta**: `/api/v1/congresses-new/{id_congreso}/papers/{paper_id}/review`
- **Autenticación**: Bearer Token (Exclusivo Administrador / Comité)
- **Cuerpo de la Petición (`application/json`)**:
  ```json
  {
    "status": "APROBADO",
    "reviewer_comments": "Excelente rigor metodológico. Aceptado para presentación oral."
  }
  ```
- **Efecto de Negocio**: Al dictaminar `APROBADO`, la tarifa del autor se actualiza automáticamente a `PONENTE` ($80.00 USD).

### 5.9 Acceso a la Sala de Streaming y Materiales
- **Método**: `GET`
- **Ruta**: `/api/v1/congresses-new/{id_congreso}/streaming`
- **Autenticación**: Bearer Token
- **Respuestas**:
  - `200 OK`: Enlace seguro de Zoom/Teams, clave de acceso y enlaces a PDFs de memorias.
  - `403 Forbidden`: Si el participante no cuenta con pago confirmado (`PAGADO` o `FACTURADO`).

### 5.10 Enviar Encuesta de Calidad CACES
- **Método**: `POST`
- **Ruta**: `/api/v1/congresses-new/{id_congreso}/survey`
- **Autenticación**: Bearer Token
- **Cuerpo de la Petición (`application/json`)**:
  ```json
  {
    "q1_rating": 5,
    "q2_rating": 4,
    "q3_rating": 5,
    "comments": "Excelente organización y ponencias magistrales internacionales."
  }
  ```
- **Respuestas**:
  - `201 Created`: Encuesta registrada exitosamente.
  - `400 Bad Request`: Si alguna calificación se encuentra fuera del rango `[1, 5]`.

### 5.11 Estadísticas de la Encuesta CACES
- **Método**: `GET`
- **Ruta**: `/api/v1/congresses-new/{id_congreso}/survey/stats`

---

## 6. Módulo de Cursos de Capacitación Continua (`/api/v1/courses`)

- `POST /api/v1/courses`: Crear curso académico (Admin).
- `GET /api/v1/courses`: Listado paginado de cursos.
- `GET /api/v1/courses/available`: Catálogo público de cursos disponibles con inscripciones abiertas.
- `GET /api/v1/courses/{course_id}`: Detalle de curso, temario y prerrequisitos.
- `PATCH /api/v1/courses/{course_id}`: Edición parcial de curso.
- `DELETE /api/v1/courses/{course_id}`: Eliminación lógica o desactivación.
- `POST /api/v1/courses/{course_id}/survey`: Encuesta CACES para cursos de capacitación.
- `GET /api/v1/courses/{course_id}/survey/stats`: Estadísticas de calidad del curso.

---

## 7. Módulo de Cronograma y Sesiones (`/api/v1/sessions`)

- `POST /api/v1/sessions`: Registrar nueva sesión en el cronograma.
- `GET /api/v1/sessions/by-congress/{congress_id}`: Cronograma completo de un evento.
- `GET /api/v1/sessions/daily-schedule/{congress_id}/{fecha}`: Agenda filtrada por día específico.
- `GET /api/v1/sessions/{session_id}`: Detalle de la sesión (horario, salón, ponente asignado).
- `PATCH /api/v1/sessions/{session_id}`: Modificación de horario o lugar.
- `DELETE /api/v1/sessions/{session_id}`: Cancelación de sesión.

---

## 8. Módulos Auxiliares y Gestión de Archivos

- **Ponentes (`/api/v1/speakers`)**: CRUD de conferencistas, biografía y asignación institucional.
- **Auspiciantes (`/api/v1/sponsors` y `/api/v1/sponsorships`)**: Gestión de patrocinadores por niveles (*Platinum, Gold, Silver*).
- **Categorías (`/api/v1/categories` y `/api/v1/congress-categories`)**: Taxonomía temática de eventos.
- **Imágenes (`/api/v1/images/upload`)**: Carga de fotos de ponentes y posters promocionales.
- **Documentos PDF (`/api/v1/pdf/upload`)**: Almacenamiento de sílabos y artículos científicos.
- **Íconos SVG (`/api/v1/svg/upload`)**: Gestión de gráficos vectoriales para categorías temáticas.
