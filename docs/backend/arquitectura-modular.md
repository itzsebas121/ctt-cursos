# Arquitectura Modular y Middlewares de Back-CTT

Este documento describe la arquitectura modular del backend FastAPI (`Back-CTT`), el ciclo de vida de la aplicación, el patrón de inyección de dependencias utilizado para la gestión transaccional de sesiones de base de datos y la configuración de middlewares y logging.

## Inicialización y Ciclo de Vida de la Aplicación

La aplicación principal se instancia en `src/main.py` mediante un gestor de contexto asíncrono `lifespan`:

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    # 1. Asegurar la existencia de directorios para almacenamiento estático
    init_svg_directory()
    init_upload_directory()
    init_pdf_directory()
    
    # 2. Ejecutar semillas iniciales de datos protegidas contra fallos
    _run_seed(seed_users)
    _run_seed(seed_categories)
    _run_seed(seed_congress_categories)
    _run_seed(seed_courses)
    _run_seed(seed_courses_bulk)
    _run_seed(seed_users_platform)
    _run_seed(seed_enrollments)
    yield
```

### Características del Ciclo de Vida

- **Resiliencia en el Arranque**: La función auxiliar `_run_seed` encapsula la ejecución de cada semilla dentro de un bloque `try-except` con registro en el logger. Si una semilla falla por duplicidad de datos en bases de datos preexistentes, la aplicación no aborta su arranque y continúa sirviendo tráfico HTTP normalmente.
- **Gestión de Esquema Desacoplada**: El esquema de base de datos relacional no utiliza `create_all()` en tiempo de ejecución en producción, sino que se gestiona de forma determinista mediante **Alembic** (`alembic upgrade head`), garantizando que las modificaciones estructurales (`ALTER TABLE`) se apliquen de forma controlada antes de que el servidor Uvicorn acepte peticiones.

---

## Inyección de Dependencias y Manejo de Sesiones

Para interactuar con PostgreSQL de forma segura y concurrente, `Back-CTT` implementa el patrón de inyección de dependencias de FastAPI en `src/dependencies/db_session.py`:

```python
from typing import Annotated, Generator
from fastapi import Depends
from sqlmodel import Session
from src.config.db import engine

def get_db() -> Generator[Session, None, None]:
    with Session(engine) as session:
        yield session

SessionDep = Annotated[Session, Depends(get_db)]
```

### Principios del Manejo de Sesiones

1. **Aislamiento por Petición (Request-Scoped)**: Cada endpoint que declara un parámetro de tipo `SessionDep` recibe una instancia de `Session` fresca, abierta exclusivamente para atender esa petición HTTP.
2. **Cierre Automático y Liberación de Conexiones**: El bloque contextual `with Session(engine) as session` garantiza que al finalizar la ejecución del endpoint (incluso si se produce una excepción no controlada), la sesión se cierre y la conexión retorne al pool de conexiones del motor SQLAlchemy.
3. **Soporte para Pruebas Unitarias e Integración**: Al usar `Depends(get_db)`, la suite de pruebas automatizadas con pytest puede sobrescribir fácilmente la dependencia (`app.dependency_overrides[get_db] = override_get_db`) para conectar a una base de datos SQLite en memoria sin alterar el código fuente de los controladores.

---

## Middlewares del Sistema

`Back-CTT` aplica una pila de middlewares estratégicos que interceptan el tráfico antes de ser procesado por los routers:

### 1. Middleware de Manejo y Normalización de Errores

Implementado en `src/middleware/error_handler.py`, captura cualquier excepción surgida durante la ejecución de consultas en SQLAlchemy o errores generales del servidor:

```python
async def error_handler_middleware(request: Request, call_next: Callable):
    try:
        response = await call_next(request)
        return response
    except SQLAlchemyError as e:
        logger.error(f"Database error: {str(e)}", exc_info=True)
        
        if isinstance(e, OperationalError):
            error_message = "Error de conexión con la base de datos. Por favor, intente nuevamente."
        elif isinstance(e, IntegrityError):
            error_message = "Error de integridad de datos. Verifique que los datos sean correctos."
        elif isinstance(e, DatabaseError):
            error_message = "Error al procesar la operación en la base de datos."
        else:
            error_message = "Ha ocurrido un error con la base de datos."
        
        return JSONResponse(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            content={
                "error": "Database Error",
                "message": error_message,
                "detail": "Por favor, contacte al administrador si el problema persiste."
            }
        )
    except Exception as e:
        logger.error(f"Unexpected error: {str(e)}", exc_info=True)
        return JSONResponse(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            content={
                "error": "Internal Server Error",
                "message": "Ha ocurrido un error inesperado.",
                "detail": "Por favor, contacte al administrador si el problema persiste."
            }
        )
```

**Ventajas operativas**:
- **Seguridad en Producción**: Oculta cadenas de conexión, nombres de tablas internas y sintaxis SQL ante ataques de reconocimiento o inyección.
- **Trazabilidad Interna**: Registra el rastro de la pila (`exc_info=True`) exclusivamente en los archivos de log locales para diagnóstico por parte del equipo de infraestructura.

### 2. Middleware de Seguridad CORS (Cross-Origin Resource Sharing)

Configurado en `src/main.py`, restringe el acceso HTTP según el entorno de ejecución:

```python
raw_origins = os.getenv("ALLOWED_ORIGINS", "")
origins = [o.strip() for o in raw_origins.split(",") if o.strip()]
if not origins:
    origins = [
        "http://localhost:3000",
        "http://localhost:3001",
        "http://localhost:3002",
        "http://127.0.0.1:3000",
        "http://127.0.0.1:3001",
        "http://127.0.0.1:3002",
    ]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

- Permite solicitudes de credenciales (`allow_credentials=True`), lo cual es indispensable para el envío de cabeceras de autorización `Authorization: Bearer <token>`.
- Soporta configuración dinámica mediante la variable de entorno `ALLOWED_ORIGINS` para acoplarse a dominios institucionales (`https://fisei.uta.edu.ec`, `https://ctt.uta.edu.ec`).

---

## Sistema de Registro de Sucesos (Logging)

El sistema de registro centralizado se define en `src/config/logging_config.py` y se inicializa antes de cargar cualquier router:

- **Estructura de Directorio**: Los registros se almacenan dentro de `Back-CTT/logs/`.
- **Estrategia de Rotación**: Implementa `RotatingFileHandler` con un límite estricto de **10 MB** por archivo y un máximo de **5 archivos de respaldo histórico**, evitando que el almacenamiento en disco se sature en entornos de producción.
- **Canales Separados**:
  1. `logs/error.log`: Registra eventos clasificados como `ERROR` y `CRITICAL` (por ejemplo, fallos de conexión a PostgreSQL o caídas en hilos secundarios).
  2. `logs/app.log`: Registra eventos informativos `INFO` (notificaciones de seeds ejecutados, transacciones completadas, generación de PDFs).
  3. `StreamHandler (Consola)`: Emite registros formateados a `stdout` en tiempo real, lo que permite su captura directa por parte del recolector de logs de Docker (`docker logs -f ctt-backend`).

Formato estándar de línea de log:
```
%(asctime)s - %(name)s - %(levelname)s - %(message)s (Fecha: YYYY-MM-DD HH:MM:SS)
```
