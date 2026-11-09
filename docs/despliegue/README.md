# Despliegue e Infraestructura del Ecosistema CTT-FISEI

Esta sección documenta los procedimientos, requerimientos técnicos, topología de red y configuraciones necesarias para desplegar el ecosistema CTT-FISEI en entornos de desarrollo local y producción sobre contenedores Docker.

## Visión General del Despliegue

El ecosistema está diseñado bajo el principio de **paridad entre desarrollo y producción (Twelve-Factor App)**. La infraestructura se apoya en contenedores aislados y orquestación con Docker Compose, permitiendo ejecutar todo el ecosistema (PostgreSQL, FastAPI Backend, React Web y React Admin) con un único comando o mediante procesos locales independientes.

```
+-----------------------------------------------------------------------------------+
|                            Topología de Despliegue                                |
+-----------------------------------------------------------------------------------+
|                                                                                   |
|  [Cliente / Navegador Web]                                                        |
|           |                                                                       |
|           +---> :3000 (HTTP) -> CTT-WEB (Nginx Alpine / Vite Dev)                |
|           +---> :3001 (HTTP) -> CTT-ADMIN (Nginx Alpine / Vite Dev)              |
|           +---> :3002 (HTTP) -> Back-CTT (Uvicorn / FastAPI ASGI)                 |
|                                     |                                             |
|                                     +---> :3003 (TCP) -> PostgreSQL 16 Alpine     |
|                                     +---> :587 (SMTP) -> Servidor de Correo UTA   |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

### Serie de Puertos Unificada (3000-3003)

Para evitar colisiones de puertos y esquivar restricciones de seguridad de los navegadores modernos (como el error `ERR_UNSAFE_PORT` en Chromium para puertos por encima de 6000), el monorepo estandariza los puertos externos del host en la serie 3000:

| Servicio | Componente | Puerto Host | Puerto Contenedor | Propósito |
|---|---|---|---|---|
| `ctt-web` | `CTT-WEB` | `3000` | `80` (Prod) / `5173` (Dev) | Portal público y de estudiantes |
| `ctt-admin` | `CTT-ADMIN` | `3001` | `80` (Prod) / `5173` (Dev) | Panel de administración y eventos |
| `ctt-backend` | `Back-CTT` | `3002` | `8000` | API REST FastAPI |
| `postgres` | Base de Datos | `3003` | `5432` | Motor de persistencia PostgreSQL 16 |

---

## Guía de Navegación

Para consultar los procedimientos específicos de configuración y puesta en marcha, revise los siguientes documentos:

1. [Guía de Desarrollo Local](desarrollo-local.md):
   - Requisitos del sistema (Python 3.12, Node.js 20+, pnpm/npm, PostgreSQL 16).
   - Levantamiento manual paso a paso de cada componente sin Docker.
   - Plantillas comentadas de variables de entorno (`.env`) para backend y frontends.
2. [Despliegue con Docker y Producción](docker-produccion.md):
   - Arquitectura multicontenedor con `docker-compose.yml` y `docker-compose.dev.yml`.
   - Explicación de los Dockerfiles multi-stage y el script de arranque `wait-for-db.sh`.
   - Volúmenes persistentes, redes puente internas y buenas prácticas de seguridad.
3. [Volver al Portal Principal](../README.md)
