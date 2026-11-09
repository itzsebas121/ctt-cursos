# Despliegue con Docker y Entornos de Producción

Este documento detalla la arquitectura de contenedores, la configuración multicontenedor de Docker Compose, el análisis de los Dockerfiles y las directrices de seguridad para el despliegue del ecosistema CTT-FISEI.

## Arquitectura de Contenedores

El despliegue con Docker Compose desacopla la aplicación en cuatro servicios independientes conectados a través de una red interna aislada (`ctt-network`):

```
+-----------------------------------------------------------------------------------------+
|                                     Host Docker                                         |
|                                                                                         |
|  Puertos Host:           :3000                 :3001                 :3002      :3003   |
|                            |                     |                     |          |     |
|  +-------------------------|---------------------|---------------------|----------|--+  |
|  | Red: ctt-network        v                     v                     v          v  |  |
|  |                  +------------+        +------------+        +------------+   |  |  |
|  |                  |  ctt-web   |        | ctt-admin  |        |ctt-backend |   |  |  |
|  |                  | (Puerto 80)|        | (Puerto 80)|        |(Puerto 8000|   |  |  |
|  |                  |Nginx Alpine|        |Nginx Alpine|        |   Uvicorn  |   |  |  |
|  |                  +------------+        +------------+        +------+-----+   |  |  |
|  |                                                                     |         |  |  |
|  |                                                                     v         |  |  |
|  |                                                              +------------+   |  |  |
|  |                                                              |  postgres  |   |  |  |
|  |                                                              |(Puerto 5432|   |  |  |
|  |                                                              +------+-----+   |  |  |
|  |                                                                     |         |  |  |
|  +---------------------------------------------------------------------|---------+--+  |
|                                                                        v                |
|  Volumen Persistente:                                          [postgres_data]          |
+-----------------------------------------------------------------------------------------+
```

---

## Análisis de los Dockerfiles

### 1. Backend (`Back-CTT/Dockerfile`)

La imagen del backend implementa un patrón seguro basado en `python:3.12-slim-bookworm`:

- **Principio de Menor Privilegio (Non-Root User)**: Crea el grupo `appgroup` y el usuario del sistema `appuser` (`/usr/sbin/nologin`).
- **Uso de `gosu`**: Instala la utilidad `gosu`. El contenedor inicia inicialmente como `root` para que el script `wait-for-db.sh` pueda inicializar los directorios de datos en los volúmenes montados y corregir sus permisos (`chown -R appuser:appgroup`). Posteriormente, `gosu appuser` transfiere la ejecución de las migraciones de Alembic y del proceso Uvicorn al usuario no privilegiado, garantizando que el servidor HTTP nunca corra como root.
- **Script de Inicialización `wait-for-db.sh`**:
  1. *Fase 1*: Espera activa con reintentos mediante un script Python ligero (`psycopg2`) hasta que PostgreSQL responda afirmativamente.
  2. *Fase 2*: Creación de los directorios de almacenamiento (`static/images/courses`, `static/pdf`, `logs`).
  3. *Fase 3*: Ejecución automatizada de migraciones (`gosu appuser alembic upgrade head`).
  4. *Fase 4*: Lanzamiento del servidor ASGI Uvicorn (`gosu appuser uvicorn src.main:app --host 0.0.0.0 --port 8000`).

### 2. Frontends (`CTT-WEB/Dockerfile` y `CTT-ADMIN/Dockerfile`)

Ambos frontends emplean el patrón de **compilación en múltiples etapas (Multi-Stage Build)** para maximizar la seguridad y optimizar el peso de la imagen final:

- **Etapa 1 (Build)**: Basada en `node:20`. Instala dependencias (`npm ci --include=dev`), inyecta la variable de compilación `ARG VITE_API_URL` y ejecuta `npm run build` para generar los bundles estáticos minificados en `/app/dist`.
- **Etapa 2 (Producción)**: Basada en `nginx:alpine`. Descarta todas las herramientas de desarrollo de Node.js, copia exclusivamente los artefactos compilados hacia `/usr/share/nginx/html` y añade la configuración personalizada de Nginx (`default.conf`) para enrutar todas las peticiones de la SPA hacia `index.html`. El resultado es una imagen ultraligera (<25 MB) y con una superficie de ataque mínima.

---

## Orquestación con Docker Compose

El repositorio dispone de dos archivos de orquestación según el propósito operativo:

### 1. Entorno de Producción (`docker-compose.yml`)

Levanta la arquitectura completa con frontends compilados en Nginx y backend en modo producción:

```bash
# Compilar y arrancar en segundo plano
docker compose up -d --build

# Verificar el estado de los contenedores
docker compose ps

# Inspeccionar logs en vivo
docker compose logs -f ctt-backend
```

### 2. Entorno de Desarrollo con Hot-Reload (`docker-compose.dev.yml`)

Configurado para desarrollo ágil en equipo:
- Monta los directorios locales de código fuente (`./Back-CTT/src:/app/src`, `./CTT-WEB:/app`, `./CTT-ADMIN:/app`) como volúmenes en caliente.
- Ejecuta Uvicorn con `--reload` y los frontends con el servidor de desarrollo de Vite (`vite --host 0.0.0.0 --port 5173`), reflejando instantáneamente cualquier cambio de código sin reconstruir las imágenes Docker.

```bash
docker compose -f docker-compose.dev.yml up -d --build
```

---

## Mapeo de Puertos, Redes y Volúmenes

### Mapeo de Puertos Externos (Host)

| Servicio | Variable en `.env` | Puerto Host | Puerto Interno Contenedor |
|---|---|---|---|
| `ctt-web` | `WEB_PORT` | `3000` | `80` (HTTP Nginx) |
| `ctt-admin` | `ADMIN_PORT` | `3001` | `80` (HTTP Nginx) |
| `ctt-backend` | `API_PORT` | `3002` | `8000` (HTTP Uvicorn) |
| `postgres` | `POSTGRES_EXTERNAL_PORT` | `3003` | `5432` (PostgreSQL) |

### Redes Internas

- **Nombre**: `ctt-network` (driver: `bridge`).
- Todos los servicios se resuelven internamente por su nombre de servicio Docker:
  - El backend se conecta a la base de datos mediante el host `postgres:5432`.
  - Los frontends acceden a la API mediante la URL configurada en el cliente web.

### Volúmenes Persistentes

- `postgres_data`: Volumen administrado por Docker que preserva los datos de PostgreSQL en `/var/lib/postgresql/data` ante reinicios o destrucción de contenedores.
- Montajes de almacenamiento estático en `ctt-backend`:
  - `./Back-CTT/static/pdf:/app/static/pdf`: Almacenamiento persistente de comprobantes RIDE y órdenes emitidas.
  - `./Back-CTT/static/images:/app/static/images`: Banners, fotos de ponentes y materiales de congresos.
  - `./Back-CTT/logs:/app/logs`: Bitácoras históricas generadas por el backend.

---

## Variables de Entorno Críticas para Producción

Para el despliegue en servidores institucionales de la UTA, configure el archivo `.env` en la raíz con valores robustos y seguros:

```env
# ── Puertos Host ─────────────────────────────────────────────────────────────
WEB_PORT=3000
ADMIN_PORT=3001
API_PORT=3002
POSTGRES_EXTERNAL_PORT=3003

# ── Base de Datos PostgreSQL ──────────────────────────────────────────────────
POSTGRES_DB=ctt_db_prod
POSTGRES_USER=ctt_prod_user
# Generar clave aleatoria criptográficamente segura (mínimo 32 caracteres):
POSTGRES_PASSWORD=ClaveUltraSeguraProduccionUTA2026_xyz987!
POSTGRES_HOST=postgres
POSTGRES_PORT=5432

# ── Backend FastAPI ───────────────────────────────────────────────────────────
# Clave privada para firma de tokens JWT (HS256) - NUNCA compartir ni versionar:
JWT_SECRET_KEY=ClaveMaestraJWTProduccionFISEI2026_a8f9e0b1c2d3e4f5!
DEBUG=False
BACKEND_URL=https://api-ctt.fisei.uta.edu.ec
ALLOWED_ORIGINS=https://ctt.fisei.uta.edu.ec,https://admin-ctt.fisei.uta.edu.ec

# ── Frontend API URL (Inyectado en compilación de Vite) ───────────────────────
VITE_API_URL=https://api-ctt.fisei.uta.edu.ec/api/v1
```

---

## Recomendaciones de Seguridad para Producción

1. **Desactivar el Modo de Depuración**: Asegurar `DEBUG=False` en producción para impedir que errores internos expongan metadatos en las respuestas HTTP.
2. **Reverse Proxy TLS/HTTPS**: Colocar un proxy inverso perimetral institucional (como Nginx, Traefik o Caddy) al frente de los puertos `3000`, `3001` y `3002` con certificados SSL/TLS emitidos por una Autoridad de Certificación reconocida.
3. **Cierre de Puertos de Base de Datos**: En entornos de producción expuestos a Internet, comente o elimine la directiva de puertos expuestos en el servicio `postgres` (`POSTGRES_EXTERNAL_PORT`), garantizando que la base de datos sea accesible exclusivamente a través de la red interna `ctt-network`.
4. **Respaldos Automatizados de PostgreSQL**: Programar tareas cron en el servidor anfitrión para la generación periódica de volcados con `pg_dump`:
   ```bash
   docker exec -t ctt-postgres pg_dump -U ctt_prod_user ctt_db_prod | gzip > /backups/ctt_db_$(date +%Y%m%d_%H%M%S).sql.gz
   ```
