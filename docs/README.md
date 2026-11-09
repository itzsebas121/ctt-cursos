# Documentación Técnica del Ecosistema CTT-FISEI

Bienvenido al portal central de documentación técnica de la plataforma integral del **Centro de Transferencia y Desarrollo de Tecnologías (CTT)** de la **Facultad de Ingeniería en Sistemas, Electrónica e Industrial (FISEI)** de la **Universidad Técnica de Ambato (UTA)**.

Este repositorio alberga la especificación arquitectónica, el modelo de datos, la referencia de la API REST, la arquitectura de los frontends cliente y las directrices de despliegue para el ecosistema tecnológico que da soporte a los congresos científicos oficiales (como CSEI y CISED) y a los programas de capacitación continua de la facultad.

---

## Resumen Ejecutivo del Ecosistema

El ecosistema CTT-FISEI está conformado por tres aplicaciones interconectadas bajo una arquitectura de micro-servicios y monorepo:

1. **Back-CTT (Backend API REST)**:
   - Motor central desarrollado en **Python 3.12** con **FastAPI**, **SQLModel / SQLAlchemy** y base de datos relacional **PostgreSQL 16**.
   - Integra autenticación dual con tokens JWT (HS256) y contraseñas hasheadas con bcrypt, control de aforo en tiempo real (`cupo_maximo`), verificación de asistencia a sesiones mediante escaneo de códigos QR con detección de colisiones HTTP 409, generador de comprobantes electrónicos RIDE bajo estándares tributarios del SRI (RUC institucional `1860000880001`), órdenes de pago institucionales UTA, sincronización con aulas virtuales Moodle LMS y encuestas de calidad académica CACES.
   - Puerto de servicio: `3002` (Host) / `8000` (Docker).

2. **CTT-WEB (Portal Público y de Estudiantes)**:
   - Aplicación web progresiva implementada en **React 19**, **Vite 7**, **Tailwind CSS v4** y animaciones fluidas con **GSAP**.
   - Proporciona a los participantes una experiencia mobile-first para el descubrimiento de eventos académicos, autogestión de perfil estudiantil con datos fiscales SRI, pasarela de pagos con generación de órdenes UTA, visualización de pase digital QR para ingreso a auditorios, envío de ponencias científicas (Call for Papers) y evaluación de satisfacción CACES.
   - Puerto de servicio: `3000` (Host) / `80` (Producción Docker).

3. **CTT-ADMIN (Panel de Gestión y Eventos)**:
   - Panel de control administrativo construido en **React 19**, **Vite 7** y **Tailwind CSS v4**.
   - Permite a los coordinadores y comités académicos gestionar el ciclo de vida completo de congresos y cursos: programación de cronogramas y ponentes magistrales, escáner móvil/USB en vivo de credenciales QR en puertas de acceso, auditoría y aprobación de depósitos bancarios, paneles financieros con desglose de recaudación por tarifas ($20, $50, $80), exportación de nóminas a Moodle y dictamen editorial de artículos científicos.
   - Puerto de servicio: `3001` (Host) / `80` (Producción Docker).

---

## Tabla de Navegación Principal

La documentación se organiza en seis áreas temáticas estructuradas. Utilice la siguiente tabla para acceder directamente a cada sección:

| Área Temática | Ruta Relativa | Contenido Destacado |
|---|---|---|
| **Arquitectura Global** | [arquitectura/](arquitectura/README.md) | Visión multicapa, principios de diseño, diagramas de arquitectura en capas, diagramas de secuencia (inscripción, escaneo QR y autenticación dual) y matriz de comunicación entre servicios. |
| **Base de Datos y Modelos** | [base-de-datos/](base-de-datos/README.md) | Diagrama entidad-relación (ER), diccionario exhaustivo de tablas, modelos relacionales SQLModel, migraciones con Alembic y semillas oficiales de eventos FISEI. |
| **Backend (FastAPI)** | [backend/](backend/README.md) | Arquitectura modular de FastAPI, inyección de dependencias `SessionDep`, sistema de autenticación dual JWT, controladores y lógica de aforo, generación de RIDE SRI en PDF y catálogo exhaustivo de endpoints. |
| **Frontend Estudiante (Web)** | [frontend-web/](frontend-web/README.md) | Arquitectura de componentes de `CTT-WEB`, diseño mobile-first, animaciones con GSAP ScrollTrigger, flujo de inscripción, pase digital QR y submódulo de Call for Papers. |
| **Frontend Administrativo (Admin)** | [frontend-admin/](frontend-admin/README.md) | Arquitectura de `CTT-ADMIN`, gestión integral de eventos y cronogramas, escáner de asistencia en vivo, conciliación de pagos, dashboard de KPIs financieros y exportador Moodle. |
| **Guías de Despliegue** | [despliegue/](despliegue/README.md) | Requisitos del sistema, guía paso a paso para desarrollo local sin Docker, configuración multicontenedor con Docker Compose y variables de entorno seguras para producción. |

---

## Normas Institucionales de Diseño y Arquitectura

Todo el ecosistema y su documentación se rigen por las siguientes directrices institucionales establecidas en `AGENTS.md`:

1. **Identidad Visual Corporativa**:
   - Azul institucional primario de la Universidad Técnica de Ambato: `#00345e`.
   - Contraste visual adecuado y coherente en modo claro y modo oscuro, evitando combinaciones de bajo contraste.
   - Fotografías e iconografía auténticas del campus universitario Huachi y sus laboratorios, evitando material genérico.
2. **Prohibición Estricta de Elementos Decorativos Superficiales**:
   - Queda terminantemente prohibido el uso de cápsulas, pastillas flotantes (*pill badges*) o sobrelíneas decorativas (*eyebrows*) colocadas artificialmente sobre los encabezados (`<h1>`, `<h2>`, `<h3>`).
   - La jerarquía visual debe emanar exclusivamente de la escala tipográfica, el contraste armónico y la separación espacial limpia.
3. **Animaciones con GSAP y ScrollTrigger**:
   - Cero desaparición o parpadeo de contenido al desplazar la página; queda prohibido el uso de animaciones de salida que dejen elementos con opacidad cero (`toggleActions: "play reverse play reverse"`).
   - Siempre limpiar propiedades CSS inline al culminar la animación de entrada (`clearProps: "all"` y `once: true`) para garantizar visibilidad total al realizar scroll inverso.
4. **Integridad de Datos y Cumplimiento Normativo**:
   - Los datos fiscales cumplen la ficha técnica del SRI ecuatoriano para comprobantes electrónicos educativos exentos de IVA (Tarifa 0%).
   - Cifrado de credenciales con truncamiento seguro a 72 bytes en `bcrypt`.
