# Diagramas Entidad-Relación - Base de Datos CTT FISEI

Este documento contiene los diagramas Entidad-Relación (ER) del modelo de datos de la plataforma CTT FISEI, renderizados mediante sintaxis estricta de **Mermaid**. Se presentan tanto el diagrama macro de todo el ecosistema como los diagramas modulares de cada área de negocio.

---

## 1. Diagrama Macro del Ecosistema Relacional

El siguiente diagrama modela las 17 tablas relacionales y sus vínculos cardinales en el sistema de gestión del CTT FISEI:

```mermaid
erDiagram
    USERS ||--o{ CATEGORIES : "administra"
    USERS ||--o{ CONGRESS_CATEGORIES : "administra"
    
    CATEGORIES ||--o{ COURSE : "clasifica"
    COURSE ||--|| COURSEREQUIREMENT : "define_requisitos"
    COURSE ||--o{ COURSECONTENT : "desglosa_silabo"
    
    CONGRESOS ||--o{ SPEAKERS : "participa"
    CONGRESOS ||--o{ SESIONES_CRONOGRAMA : "programa"
    CONGRESOS ||--o{ CONGRESO_SPONSORS : "reune"
    SPONSORS ||--o{ CONGRESO_SPONSORS : "patrocina"
    SPEAKERS ||--o{ SESIONES_CRONOGRAMA : "imparte"
    
    USERS_PLATFORM ||--o{ ENROLLMENTS : "realiza"
    COURSE ||--o{ ENROLLMENTS : "recibe"
    CONGRESOS ||--o{ ENROLLMENTS : "recibe"
    
    ENROLLMENTS ||--o{ SESSION_ATTENDANCE : "genera"
    SESIONES_CRONOGRAMA ||--o{ SESSION_ATTENDANCE : "valida_qr"
    
    CONGRESOS ||--o{ SCIENTIFIC_PAPERS : "convoca"
    USERS_PLATFORM ||--o{ SCIENTIFIC_PAPERS : "somete_articulo"
    
    CONGRESOS ||--o{ CACES_SURVEYS : "recibe_evaluacion"
    COURSE ||--o{ CACES_SURVEYS : "recibe_evaluacion"
    USERS_PLATFORM ||--o{ CACES_SURVEYS : "contesta"

    USERS {
        int id PK
        string name
        string last_name
        string email UK
        string password
    }

    USERS_PLATFORM {
        int id PK
        string identification
        string first_name
        string second_name
        string first_last_name
        string second_last_name
        string cellphone
        string email UK
        string address
        string type
        string password
        string billing_identification
        string billing_name
        string billing_address
        string billing_phone
        string billing_email
    }

    CONGRESOS {
        int id_congreso PK
        string nombre
        string edicion
        int anio
        date fecha_inicio
        date fecha_fin
        text descripcion_general
        string poster_cover_url
        string lugar_sede
        string modalidad
        int duracion_horas
        string sitio_web_oficial
        int cupo_maximo
        float costo_estudiante_uta
        float costo_profesional
        float costo_ponente
    }

    SPEAKERS {
        int id_speaker PK
        int id_congreso FK
        string nombres_completos
        string titulo_academico
        string institucion
        string pais
        string foto_url
        string tipo_speaker
    }

    SPONSORS {
        int id_sponsor PK
        string nombre
        string logo_url
        string sitio_web
        text descripcion
        boolean activo
    }

    CONGRESO_SPONSORS {
        int id_congreso PK,FK
        int id_sponsor PK,FK
        string categoria
        decimal aporte
    }

    SESIONES_CRONOGRAMA {
        int id_sesion PK
        int id_congreso FK
        int id_speaker FK
        date fecha
        time hora_inicio
        time hora_fin
        string titulo_sesion
        string jornada
        string lugar
    }

    SESSION_ATTENDANCE {
        int id PK
        int id_enrollment FK
        int id_sesion FK
        datetime check_in_time
        int scanned_by_user_id
        string notes
    }

    CATEGORIES {
        int id PK
        string name UK
        string description
        string svgurl
        string status
        datetime created_at
        datetime updated_at
        int created_by FK
    }

    CONGRESS_CATEGORIES {
        int id PK
        string name UK
        string description
        string svgurl
        string status
        datetime created_at
        datetime updated_at
        int created_by FK
    }

    COURSE {
        int id PK
        string title
        text description
        string place
        string course_image
        string course_image_detail
        int category_id FK
        string status
        text objectives
        text organizers
        text materials
        text target_audience
    }

    COURSEREQUIREMENT {
        int id PK
        int course_id FK,UK
        date start_date_registration
        date end_date_registration
        date start_date_course
        date end_date_course
        text days
        time start_time
        time end_time
        string location
        int min_quota
        int max_quota
        int in_person_hours
        int autonomous_hours
        string modality
        string certification
        text prerequisites
        text prices
    }

    COURSECONTENT {
        int id PK
        int course_id FK
        string unit
        string title
        text topics
    }

    ENROLLMENTS {
        int id PK
        int id_user_platform FK
        int id_course FK
        int id_congress FK
        datetime enrollment_date
        string status
        string payment_order_url
        string tariff_type
        float amount
    }

    SCIENTIFIC_PAPERS {
        int id PK
        int id_congress FK
        int id_user_platform FK
        string title
        text abstract
        string track
        string keywords
        string file_url
        string status
        text reviewer_comments
        datetime submitted_at
        datetime updated_at
    }

    CACES_SURVEYS {
        int id PK
        int id_congress FK
        int id_course FK
        int id_user_platform FK
        int q1_rating
        int q2_rating
        int q3_rating
        text comments
        datetime submitted_at
    }

    POSTS {
        int id PK
        json config
    }
```

---

## 2. Módulo de Autenticación y Cuentas de Usuario

Representa la separación estricta entre el personal administrativo de FISEI (`users`) y los usuarios finales de la plataforma académica (`users_platform`), además de la trazabilidad en la creación de categorías institucionales:

```mermaid
erDiagram
    USERS ||--o{ CATEGORIES : "registra"
    USERS ||--o{ CONGRESS_CATEGORIES : "registra"

    USERS {
        int id PK
        string name
        string last_name
        string email UK
        string password
    }

    CATEGORIES {
        int id PK
        string name UK
        string description
        string svgurl
        string status
        datetime created_at
        datetime updated_at
        int created_by FK
    }

    CONGRESS_CATEGORIES {
        int id PK
        string name UK
        string description
        string svgurl
        string status
        datetime created_at
        datetime updated_at
        int created_by FK
    }

    USERS_PLATFORM {
        int id PK
        string identification
        string first_name
        string second_name
        string first_last_name
        string second_last_name
        string cellphone
        string email UK
        string address
        string type
        string password
        string billing_identification
        string billing_name
        string billing_address
        string billing_phone
        string billing_email
    }
```

---

## 3. Módulo de Congresos, Cronogramas y Auspiciantes

Detalla la arquitectura de gestión de eventos internacionales como CSEI y CISED, coordinando conferencistas magistrales, agenda horaria y alianzas con patrocinadores:

```mermaid
erDiagram
    CONGRESOS ||--o{ SPEAKERS : "convoca"
    CONGRESOS ||--o{ SESIONES_CRONOGRAMA : "agenda"
    CONGRESOS ||--o{ CONGRESO_SPONSORS : "vincula"
    SPONSORS ||--o{ CONGRESO_SPONSORS : "financia"
    SPEAKERS ||--o{ SESIONES_CRONOGRAMA : "diserta"

    CONGRESOS {
        int id_congreso PK
        string nombre
        string edicion
        int anio
        date fecha_inicio
        date fecha_fin
        text descripcion_general
        string poster_cover_url
        string lugar_sede
        string modalidad
        int duracion_horas
        string sitio_web_oficial
        int cupo_maximo
        float costo_estudiante_uta
        float costo_profesional
        float costo_ponente
    }

    SPEAKERS {
        int id_speaker PK
        int id_congreso FK
        string nombres_completos
        string titulo_academico
        string institucion
        string pais
        string foto_url
        string tipo_speaker
    }

    SPONSORS {
        int id_sponsor PK
        string nombre
        string logo_url
        string sitio_web
        text descripcion
        boolean activo
    }

    CONGRESO_SPONSORS {
        int id_congreso PK,FK
        int id_sponsor PK,FK
        string categoria
        decimal aporte
    }

    SESIONES_CRONOGRAMA {
        int id_sesion PK
        int id_congreso FK
        int id_speaker FK
        date fecha
        time hora_inicio
        time hora_fin
        string titulo_sesion
        string jornada
        string lugar
    }
```

---

## 4. Módulo de Cursos de Formación Continua

Modela la oferta de capacitación técnica del CTT, su dependencia de categorías temáticas y su descomposición en requerimientos operativos (1:1) y unidades de contenido modular (1:N):

```mermaid
erDiagram
    CATEGORIES ||--o{ COURSE : "agrupa"
    COURSE ||--|| COURSEREQUIREMENT : "posee_requisitos"
    COURSE ||--o{ COURSECONTENT : "contiene_unidades"

    CATEGORIES {
        int id PK
        string name UK
        string description
        string svgurl
        string status
        int created_by FK
    }

    COURSE {
        int id PK
        string title
        text description
        string place
        string course_image
        string course_image_detail
        int category_id FK
        string status
        text objectives
        text organizers
        text materials
        text target_audience
    }

    COURSEREQUIREMENT {
        int id PK
        int course_id FK,UK
        date start_date_registration
        date end_date_registration
        date start_date_course
        date end_date_course
        text days
        time start_time
        time end_time
        string location
        int min_quota
        int max_quota
        int in_person_hours
        int autonomous_hours
        string modality
        string certification
        text prerequisites
        text prices
    }

    COURSECONTENT {
        int id PK
        int course_id FK
        string unit
        string title
        text topics
    }
```

---

## 5. Módulo de Inscripciones, Asistencia QR y Facturación RIDE

Ilustra la relación polimórfica exclusiva (XOR) en `enrollments`, el control de asistencia presencial por escaneo QR y los datos de facturación electrónica:

```mermaid
erDiagram
    USERS_PLATFORM ||--o{ ENROLLMENTS : "solicita"
    CONGRESOS ||--o{ ENROLLMENTS : "recibe_pago"
    COURSE ||--o{ ENROLLMENTS : "recibe_pago"
    ENROLLMENTS ||--o{ SESSION_ATTENDANCE : "registra_asistencia"
    SESIONES_CRONOGRAMA ||--o{ SESSION_ATTENDANCE : "valida_ingreso"

    USERS_PLATFORM {
        int id PK
        string identification
        string email UK
        string billing_identification
        string billing_name
        string billing_address
        string billing_phone
        string billing_email
    }

    ENROLLMENTS {
        int id PK
        int id_user_platform FK
        int id_course FK
        int id_congress FK
        datetime enrollment_date
        string status
        string payment_order_url
        string tariff_type
        float amount
    }

    SESSION_ATTENDANCE {
        int id PK
        int id_enrollment FK
        int id_sesion FK
        datetime check_in_time
        int scanned_by_user_id
        string notes
    }

    CONGRESOS {
        int id_congreso PK
        string nombre
        int cupo_maximo
    }

    COURSE {
        int id PK
        string title
    }

    SESIONES_CRONOGRAMA {
        int id_sesion PK
        int id_congreso FK
        string titulo_sesion
    }
```

---

## 6. Módulo de Investigación (Call for Papers) y Calidad CACES

Modela el flujo de postulación y revisión de artículos científicos para congresos indexados (Scopus/IEEE), junto con las encuestas de satisfacción académica requeridas para acreditación universitaria:

```mermaid
erDiagram
    CONGRESOS ||--o{ SCIENTIFIC_PAPERS : "recibe_manuscritos"
    USERS_PLATFORM ||--o{ SCIENTIFIC_PAPERS : "postula_articulo"
    
    CONGRESOS ||--o{ CACES_SURVEYS : "audita_calidad"
    COURSE ||--o{ CACES_SURVEYS : "audita_calidad"
    USERS_PLATFORM ||--o{ CACES_SURVEYS : "evalua_servicio"

    CONGRESOS {
        int id_congreso PK
        string nombre
        string edicion
    }

    COURSE {
        int id PK
        string title
    }

    USERS_PLATFORM {
        int id PK
        string identification
        string email UK
    }

    SCIENTIFIC_PAPERS {
        int id PK
        int id_congress FK
        int id_user_platform FK
        string title
        text abstract
        string track
        string keywords
        string file_url
        string status
        text reviewer_comments
        datetime submitted_at
        datetime updated_at
    }

    CACES_SURVEYS {
        int id PK
        int id_congress FK
        int id_course FK
        int id_user_platform FK
        int q1_rating
        int q2_rating
        int q3_rating
        text comments
        datetime submitted_at
    }
```
