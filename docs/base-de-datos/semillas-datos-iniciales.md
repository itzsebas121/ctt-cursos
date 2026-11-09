# Semillas y Datos Maestros Iniciales - CTT FISEI

Este documento describe la arquitectura, datos predefinidos y procedimientos de ejecución del subsistema de inicialización de datos (**Seeds**) en `Back-CTT/src/utils/seeds/`. Las semillas permiten poblar la base de datos con información institucional oficial de la Facultad de Ingeniería en Sistemas, Electrónica e Industrial (FISEI) de la Universidad Técnica de Ambato (UTA), facilitando entornos de desarrollo local, pruebas automatizadas y demostraciones operativas.

---

## 1. Arquitectura del Motor de Seeds

El subsistema de inicialización está modularizado en `src/utils/seeds/` y coordinado por el script orquestador `main_seed.py`.

### Dependencia y Orden de Ejecución

Para preservar la integridad referencial de claves foráneas en PostgreSQL, las semillas se ejecutan en estricto orden topológico:

```
Paso 1:  Usuarios del Sistema (Admin)  ──>  seed_users()
Paso 2:  Usuarios de Plataforma       ──>  seed_users_platform()
Paso 3:  Categorías de Cursos          ──>  seed_categories()
Paso 4:  Categorías de Congresos       ──>  seed_congress_categories()
Paso 5:  Cursos y Requerimientos       ──>  seed_courses() / seed_courses_bulk()
Paso 6:  Congresos FISEI               ──>  seed_congresses()
Paso 7:  Sponsors Maestros             ──>  seed_sponsors()
Paso 8:  Speakers de Ingeniería        ──>  seed_speakers()
Paso 9:  Sesiones del Cronograma       ──>  seed_sesion_cronograma()
Paso 10: Asociaciones Congreso-Sponsor ──>  seed_congreso_sponsor()
Paso 11: Inscripciones y Órdenes       ──>  seed_enrollments()
```

### Sincronización de Secuencias en PostgreSQL (`pg_utils.py`)

Cuando se insertan registros con valores de clave primaria explícitos (por ejemplo, `id=1`), el motor PostgreSQL no incrementa de manera automática el puntero de la secuencia `SERIAL`. Esto generaría un error de colisión de clave primaria (`duplicate key value violates unique constraint`) en la primera inserción realizada por la API.

Para evitar esta anomalía, el módulo `src/utils/seeds/pg_utils.py` implementa la función `sync_pk_sequence`:

```python
def sync_pk_sequence(session: Session, table: str, column: str = "id") -> None:
    if session.get_bind().dialect.name != "postgresql":
        return
    session.execute(
        text(
            f"SELECT setval(pg_get_serial_sequence('{table}', '{column}'), "
            f"COALESCE((SELECT MAX({column}) FROM {table}), 1), "
            f"(SELECT COUNT(*) > 0 FROM {table}))"
        )
    )
    session.commit()
```

Esta función se ejecuta al concluir cada proceso de siembra de datos, asegurando que la secuencia quede alineada con el valor máximo actual.

---

## 2. Eventos Científicos Emblemáticos FISEI UTA

La semilla `src/utils/seeds/congress_seed.py` inicializa exclusivamente los congresos institucionales reales de la facultad:

### 2.1 VII CSEI 2026 (Congreso Activo / Vigente)

- **Nombre Oficial**: VII Congreso Internacional de Ciencia de la Computación, Electrónica e Ingeniería Industrial (CSEI 2026).
- **Edición**: `VII-2026`.
- **Año**: 2026.
- **Fechas**: 21 de octubre al 23 de octubre de 2026.
- **Sede**: Universidad Técnica de Ambato - Campus Huachi, FISEI.
- **Modalidad**: Presencial con soporte híbrido.
- **Carga Académica**: 40 horas.
- **Cupo Máximo**: 300 participantes.
- **Portal Oficial**: `https://csei.uta.edu.ec`.
- **Indexación**: Scopus / IEEE Xplore.
- **Estructura Arancelaria**:
  - Estudiante UTA: **$20.00 USD** (tarifa subsidiada institucional).
  - Profesional / Público General: **$50.00 USD**.
  - Ponente / Autor de Paper: **$80.00 USD**.

### 2.2 VI CSEI 2024 (Edición Anterior)

- **Nombre Oficial**: VI Congreso Internacional de Ciencia de la Computación, Electrónica e Ingeniería Industrial (CSEI 2024).
- **Edición**: `VI-2024`.
- **Año**: 2024.
- **Fechas**: 23 de octubre al 25 de octubre de 2024.
- **Cupo**: 250 participantes.
- **Tarifas**: $20 Estudiante UTA, $50 Profesional, $80 Ponente.
- **Memorias**: Publicadas en Springer Advances in Intelligent Systems and Computing.

### 2.3 V CSEI 2023 (Edición Histórica)

- **Nombre Oficial**: V Congreso Internacional de Ciencia de la Computación, Electrónica e Ingeniería Industrial (CSEI 2023).
- **Edición**: `V-2023`.
- **Año**: 2023.
- **Fechas**: 25 de octubre al 27 de octubre de 2023.
- **Cupo**: 200 participantes.
- **Tarifas**: $20 Estudiante UTA, $50 Profesional, $80 Ponente.

### 2.4 CISED 2025 (Jornadas de Investigación y Transferencia)

- **Nombre Oficial**: I Jornadas de Investigación y Transferencia Tecnológica FISEI (CISED 2025).
- **Edición**: `CISED-2025`.
- **Año**: 2025.
- **Fechas**: 12 de noviembre al 14 de noviembre de 2025.
- **Sede**: Auditorio Central FISEI - Campus Huachi.
- **Carga Académica**: 30 horas.
- **Cupo Máximo**: 180 participantes.
- **Tarifas Institucionales**:
  - Estudiante UTA: **$15.00 USD**.
  - Profesional: **$35.00 USD**.
  - Ponente: **$50.00 USD**.

---

## 3. Usuarios de Prueba y Roles de Plataforma

### 3.1 Usuarios Administrativos (`users`)

Inicializado en `src/utils/seeds/user_seed.py`:

| ID | Nombre Completo | Correo Institucional | Contraseña en Claro | Rol Funcional |
|---|---|---|---|---|
| 1 | Daniel Jerez | `daniel.jerez@example.com` | `securepassword` | Administrador General del CTT FISEI |

*Nota*: La contraseña se almacena como hash bcrypt (`$2b$12$...`) y se actualiza automáticamente si detecta una versión en texto plano.

### 3.2 Usuarios de Plataforma (`users_platform`)

Inicializados en `src/utils/seeds/user_platform_seed.py`:

| ID | Identificación | Nombres y Apellidos | Correo Electrónico | Tipo | Contraseña |
|---|---|---|---|---|---|
| 1 | `1234567890` | María Isabel González Pérez | `maria.gonzalez@example.com` | `Estudiante` | `estudiante123` |
| 2 | `9876543210` | Carlos Ramírez Torres | `carlos.ramirez@example.com` | `Externo` | `externo123` |
| 3 | `1726543890` | Ana Lucía Morales | `ana.morales@example.com` | `Administrativo` | `admin123` |
| 4 al 20 | Generadas | Nombres ecuatorianos diversos | `usuario_N@example.com` | Ponderado | Contraseña segura |

---

## 4. Conferencistas y Patrocinadores Iniciales

### 4.1 Speakers Magistrales (`speakers_seed.py`)

- **Dr. Carlos Rodríguez Paredes**: PhD en Inteligencia Artificial y Robótica (Universidad Politécnica de Madrid / FISEI UTA), tipo `keynote`.
- **Dra. Elena Villacís Mora**: PhD en Ciberseguridad y Redes de Alta Velocidad (Georgia Institute of Technology, EE.UU.), tipo `keynote`.
- **Ing. Roberto Silva Benítez, MSc**: Master en Automatización e Industria 4.0 (Siemens Digital Industries, Alemania), tipo `conferencia`.
- **Dra. Mónica Salazar Ortiz**: PhD en Ciencia de Datos y Machine Learning (FISEI UTA), tipo `conferencia`.

### 4.2 Cronograma de Sesiones Inicial (`sesion_cronograma_seed.py`)

1. **Sesión 1**: Conferencia Magistral: Modelos Fundacionales y Edge Computing en la Industria (Día 1, 09:00 - 10:30, Auditorio Central).
2. **Sesión 2**: Track Científico: Ciberseguridad Ofensiva y Detección de Amenazas en Redes de Potencia (Día 1, 11:00 - 12:30, Sala de Grados 1).
3. **Sesión 3**: Taller Especializado: Implementación de Controladores Lógicos Programables en Gemelos Digitales (Día 2, 14:30 - 16:00, Laboratorio de Automatización).

### 4.3 Auspiciantes y Patrocinios (`sponsors_seed.py` y `congreso_sponsor_seed.py`)

Categorías de patrocinio y rangos de aporte financiero modelados:
- **Oro**: Aportes entre $15,000.00 y $50,000.00 USD.
- **Plata**: Aportes entre $8,000.00 y $14,999.00 USD.
- **Bronce**: Aportes entre $3,000.00 y $7,999.00 USD.
- **Colaborador**: Aportes entre $500.00 y $2,999.00 USD.

---

## 5. Simulación de Inscripciones y Estados de Matrícula

La semilla `src/utils/seeds/enrollment_seed.py` genera un conjunto de inscripciones representativo:

### Distribución de Estados de Inscripción
- `INTERESADO` (~30%): Participantes que preseleccionaron el congreso.
- `ORDEN_GENERADA` (~15%): Participantes con orden de pago UTA emitida (`https://payment.cttecuador.com/...`).
- `PAGADO` (~35%): Participantes con pago verificado en tesorería.
- `FACTURADO` (~15%): Participantes con RIDE oficial emitido.
- `ANULADO` (~5%): Inscripciones canceladas.

---

## 6. Comandos de Ejecución de Semillas

### Ejecución Integral de Todos los Seeds

Para ejecutar el proceso completo respetando todas las dependencias:

```bash
# Desde la carpeta Back-CTT:
python -m src.utils.seeds.main_seed
```

### Ejecución Exclusiva de Congresos

Para entornos de desarrollo centrados en eventos internacionales:

```bash
python -m src.utils.seeds.main_seed congress-only
```

### Ejecución de Semillas Individuales

```bash
python -m src.utils.seeds.user_seed
python -m src.utils.seeds.user_platform_seed
python -m src.utils.seeds.categories_seed
python -m src.utils.seeds.congress_categories_seed
python -m src.utils.seeds.congress_seed
python -m src.utils.seeds.speakers_seed
python -m src.utils.seeds.sesion_cronograma_seed
python -m src.utils.seeds.sponsors_seed
python -m src.utils.seeds.congreso_sponsor_seed
python -m src.utils.seeds.enrollment_seed
```
