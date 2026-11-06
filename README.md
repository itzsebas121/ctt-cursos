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
| **CTT-WEB** | `ctt-frontend-web` | `80` | `6000` *(o 5000/3000)* | [http://localhost:6000](http://localhost:6000) |
| **CTT-ADMIN** | `ctt-frontend-admin` | `80` | `6001` | [http://localhost:6001](http://localhost:6001) |
| **Back-CTT** | `ctt-backend` | `8000` | `6002` | [http://localhost:6002](http://localhost:6002) (Docs: `/docs`) |
| **PostgreSQL** | `ctt-postgres` | `5432` | `6003` | `localhost:6003` |

> [!WARNING]
> **Aviso sobre el puerto 6000 en navegadores:**
> Google Chrome y Mozilla Firefox bloquean el puerto `6000` por defecto con el error `ERR_UNSAFE_PORT` (por conflicto histórico con servidores X11).
> Si tu navegador no te permite abrir `http://localhost:6000`, puedes cambiar `WEB_PORT=5000` o `WEB_PORT=3000` en el archivo `.env` y volver a levantar.

---

## ⚡ Arranque Rápido

Para levantar todo el stack en contenedores:

```bash
docker compose up -d --build
```

Para ver el estado de los contenedores:
```bash
docker compose ps
```

Para ver los logs en tiempo real:
```bash
docker compose logs -f
```

Para detener todos los servicios:
```bash
docker compose down
```

---

## 📁 Gestión de Repositorios (Git)

Actualmente cada subcarpeta cuenta con su propio repositorio Git con sus remotos:
- `Back-CTT`: [https://github.com/DannyDev18/Back-CTT.git](https://github.com/DannyDev18/Back-CTT.git)
- `CTT-ADMIN`: [https://github.com/itzsebas121/CTT-ADMIN.git](https://github.com/itzsebas121/CTT-ADMIN.git)
- `CTT-WEB`: [https://github.com/itzsebas121/CTT-WEB.git](https://github.com/itzsebas121/CTT-WEB.git)

La raíz actúa como **Monorepo Orquestador**:
- Puedes inicializar un repositorio Git en la raíz (`git init`) para versionar la configuración de Docker Compose, scripts y variables de entorno (`.env.example`).
- Puedes gestionar los 3 proyectos mediante **Git Submodules** o continuar trabajando dentro de cada carpeta (`cd Back-CTT && git pull`, `cd CTT-ADMIN && git push`, etc.) sin perder la sincronización con los repositorios originales de GitHub.

