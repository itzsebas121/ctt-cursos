# Guía de Puesta en Marcha para Desarrollo Local

Esta guía detalla los pasos para levantar el entorno de desarrollo local sin contenedores Docker, ejecutando los servicios directamente sobre el sistema operativo anfitrión.

## Requisitos Previos del Sistema

Asegúrese de contar con las siguientes versiones de software instaladas:

1. **Python**: Versión `3.12.x` (o superior compatible).
2. **Node.js**: Versión `20.x LTS` (o superior).
3. **Gestor de Paquetes JavaScript**: `pnpm` versión `9.x` o superior (recomendado para el monorepo) o `npm` versión `10.x`.
4. **PostgreSQL**: Servidor PostgreSQL versión `16.x` activo y accesible.
5. **Git**: Para el control de versiones y clonación de submódulos.

---

## 1. Configuración de la Base de Datos PostgreSQL

Inicie sesión en el servidor PostgreSQL local y cree la base de datos y el usuario de la aplicación:

```sql
-- Ejecutar en psql como usuario postgres
CREATE USER ctt_app WITH PASSWORD 'ctt_secret_password';
CREATE DATABASE ctt_db OWNER ctt_app;
GRANT ALL PRIVILEGES ON DATABASE ctt_db TO ctt_app;
```

---

## 2. Configuración y Ejecución del Backend (`Back-CTT`)

### 2.1 Preparación del Entorno Virtual Python

Acceda al directorio del backend, cree un entorno virtual aislado e instale las dependencias:

```bash
cd Back-CTT

# Crear entorno virtual
python3.12 -m venv venv

# Activar entorno virtual
# En Linux/macOS:
source venv/bin/activate
# En Windows:
# venv\Scripts\activate

# Actualizar pip e instalar dependencias
pip install --upgrade pip
pip install -r requirements.txt
```

### 2.2 Variables de Entorno del Backend (`Back-CTT/.env`)

Cree el archivo `.env` dentro de `Back-CTT/` basándose en la siguiente plantilla:

```env
# ── Conexión a Base de Datos PostgreSQL ──────────────────────────────────────
# Si PostgreSQL corre localmente en el host:
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=ctt_db
POSTGRES_USER=ctt_app
POSTGRES_PASSWORD=ctt_secret_password

# ── Criptografía y Seguridad JWT ─────────────────────────────────────────────
JWT_SECRET_KEY=clave_secreta_jwt_ctt_super_segura_2026_dev

# ── Depuración y URLs de Servicio ────────────────────────────────────────────
DEBUG=True
BACKEND_URL=http://localhost:3002

# ── Orígenes Permitidos (CORS) ───────────────────────────────────────────────
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:3001,http://localhost:3002,http://127.0.0.1:3000,http://127.0.0.1:3001,http://127.0.0.1:3002

# ── Servidor SMTP Transaccional (Opcional - Dev Fallback Activo) ──────────────
SMTP_HOST=
SMTP_PORT=587
SMTP_USER=
SMTP_PASSWORD=
SMTP_FROM=ctt.fisei@uta.edu.ec
SMTP_USE_TLS=true
SMTP_USE_SSL=false
```

### 2.3 Ejecución de Migraciones de Base de Datos (Alembic)

Antes de iniciar la API por primera vez, aplique las migraciones históricas:

```bash
# Con el venv activado dentro de Back-CTT:
alembic upgrade head
```

### 2.4 Inicio del Servidor de Desarrollo Uvicorn

Inicie Uvicorn configurando el puerto en `3002` con recarga automática (*hot-reload*):

```bash
uvicorn src.main:app --host 0.0.0.0 --port 3002 --reload
```

Una vez levantado, puede verificar el estado y explorar la documentación interactiva en:
- **Swagger UI**: [http://localhost:3002/docs](http://localhost:3002/docs)
- **ReDoc**: [http://localhost:3002/redoc](http://localhost:3002/redoc)

---

## 3. Configuración y Ejecución de `CTT-WEB` (Portal Público)

En una nueva terminal, acceda al directorio de `CTT-WEB`:

```bash
cd CTT-WEB

# Instalar dependencias con pnpm o npm
npm install
# O si usa pnpm: pnpm install
```

### 3.1 Variables de Entorno (`CTT-WEB/.env`)

Cree o verifique el archivo `CTT-WEB/.env`:

```env
# URL base de consumo para la API REST del backend
VITE_API_URL=http://localhost:3002/api/v1
```

### 3.2 Iniciar Servidor de Desarrollo Vite (Puerto 3000)

Inicie el servidor Vite indicando el puerto `3000`:

```bash
npm run dev -- --port 3000 --host
```

Abra el navegador en [http://localhost:3000](http://localhost:3000).

---

## 4. Configuración y Ejecución de `CTT-ADMIN` (Panel Administrativo)

En una tercera terminal, acceda al directorio de `CTT-ADMIN`:

```bash
cd CTT-ADMIN

# Instalar dependencias con pnpm o npm
npm install
# O si usa pnpm: pnpm install
```

### 4.1 Variables de Entorno (`CTT-ADMIN/.env`)

Cree o verifique el archivo `CTT-ADMIN/.env`:

```env
# URL base de consumo para la API REST del backend
VITE_API_URL=http://localhost:3002/api/v1
```

### 4.2 Iniciar Servidor de Desarrollo Vite (Puerto 3001)

Inicie el servidor Vite en el puerto `3001`:

```bash
npm run dev -- --port 3001 --host
```

Abra el navegador en [http://localhost:3001](http://localhost:3001).

---

## 5. Resumen de Puertos y Verificación del Ecosistema

| Componente | Comando de Arranque | URL de Acceso | Credenciales por Defecto (Seed) |
|---|---|---|---|
| **Base de Datos** | `sudo systemctl start postgresql` | `localhost:5432` | Usuario: `ctt_app` |
| **Back-CTT** | `uvicorn src.main:app --port 3002 --reload` | [http://localhost:3002](http://localhost:3002) | N/A (API REST) |
| **CTT-WEB** | `npm run dev -- --port 3000` | [http://localhost:3000](http://localhost:3000) | Estudiante: `carlos.mora@example.com` / `Secret123!` |
| **CTT-ADMIN** | `npm run dev -- --port 3001` | [http://localhost:3001](http://localhost:3001) | Admin: `admin@uta.edu.ec` / `admin123` |
