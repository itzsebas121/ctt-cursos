# 🚀 CTT Cursos - Monorepo Orchestrator

Sistema integral de CTT Cursos compuesto por:
1. **Back-CTT**: API REST construida con FastAPI (Python 3.12), SQLModel/SQLAlchemy y Alembic.
2. **PostgreSQL**: Base de datos relacional (v16).
3. **CTT-WEB**: Frontend público para estudiantes/usuarios (React 19 + Vite + Tailwind CSS v4).
4. **CTT-ADMIN**: Frontend de administración y gestión (React 19 + Vite + Tailwind CSS v4).

---

## 🌐 Distribución de Puertos

| Servicio | Contenedor | Puerto Interno | Puerto Host (Accesible) | URL Local |
| :--- | :--- | :--- | :--- | :--- |
| **CTT-WEB** | `ctt-frontend-web` | `80` (prod) / `5173` (dev) | `3000` | [http://localhost:3000](http://localhost:3000) |
| **CTT-ADMIN** | `ctt-frontend-admin` | `80` (prod) / `5173` (dev) | `3001` | [http://localhost:3001](http://localhost:3001) |
| **Back-CTT** | `ctt-backend` | `8000` | `3002` | [http://localhost:3002](http://localhost:3002) (Docs: `/docs`) |
| **PostgreSQL** | `ctt-postgres` | `5432` | `3003` | `localhost:3003` |

---

## 💻 Modos de Desarrollo

### 1. 🔥 Modo Desarrollo Todo en Docker (Con Cambios en Caliente)
*Recomendado si quieres tener **hot-reload instantáneo** tanto en React como en FastAPI sin instalar dependencias en tu máquina física.*

```bash
docker compose -f docker-compose.dev.yml up -d --build
```
- Modifica cualquier archivo en `Back-CTT/src` y FastAPI se recarga automáticamente.
- Modifica cualquier componente en `CTT-WEB` o `CTT-ADMIN` y Vite actualiza el navegador al instante (HMR).

### 2. ⚡ Modo Híbrido (Solo Postgres en Docker + Tu código en local)
*Para quienes prefieren correr los procesos directamente en terminales locales con sus extensiones de VS Code:*

```bash
# 1. Levantar solo la base de datos:
docker compose up -d postgres

# 2. Terminal Backend:
cd Back-CTT
source venv/bin/activate
uvicorn src.main:app --reload --port 3002

# 3. Terminal Web:
cd CTT-WEB
npm run dev -- --port 3000

# 4. Terminal Admin:
cd CTT-ADMIN
npm run dev -- --port 3001
```

### 3. 📦 Modo Producción (Bundles optimizados con Nginx)
*Para probar exactamente cómo correrá en despliegue final:*

```bash
docker compose up -d --build
```

---

## 📁 Gestión de Repositorios (Git)

Los 3 repositorios se gestionan como **Git Submodules** apuntando a sus ramas `main`:
- `Back-CTT`: [https://github.com/DannyDev18/Back-CTT.git](https://github.com/DannyDev18/Back-CTT.git)
- `CTT-ADMIN`: [https://github.com/itzsebas121/CTT-ADMIN.git](https://github.com/itzsebas121/CTT-ADMIN.git)
- `CTT-WEB`: [https://github.com/itzsebas121/CTT-WEB.git](https://github.com/itzsebas121/CTT-WEB.git)
