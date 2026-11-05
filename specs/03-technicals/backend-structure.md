# Estructura del Backend

> **Estado:** [TODO: definido / en revisión]
> **Última actualización:** [TODO: AAAA-MM-DD]
> Reglas globales en [architecture-blueprint.md](./architecture-blueprint.md); contrato por feature en `TEMPLATE-TECH.md`.

## 1. Estructura de carpetas

```
backend/
├── .env                        # Configuración (DB_* + BACKEND_*) — no se commitea
├── .env.example                # Plantilla de variables requeridas
└── src/backend/                # src-layout de uv_build
    ├── main.py                 # App factory: crea FastAPI, CORS desde settings, monta routers
    ├── config.py               # pydantic-settings: lee backend/.env → settings
    ├── db.py                   # Conexión MySQL (PyMySQL): get_connection() / commit / rollback
    ├── apis/
    │   ├── __init__.py
    │   └── <recurso>.py        # Routers FastAPI: solo valida y delega
    ├── services/
    │   ├── __init__.py
    │   └── <recurso>.py        # Lógica de negocio
    ├── repositories/
    │   ├── __init__.py
    │   └── <recurso>.py        # SQL a mano (sin ORM) — único punto que toca la BD
    └── schemas/
        ├── __init__.py
        └── <recurso>.py        # Pydantic: DTOs de request/response
```

> **Mismo nombre de archivo en las cuatro capas** (`apis/books.py`, `services/books.py`,
> `repositories/books.py`, `schemas/books.py`) para que el rastreo sea inmediato.

## 2. Responsabilidades por capa

| Capa | Responsabilidad | NO debe hacer |
|------|-----------------|---------------|
| `main.py` | Instanciar la app, registrar routers, CORS desde `settings`, arranque con uvicorn | Contener lógica de negocio ni SQL |
| `config.py` | Leer `backend/.env` con `pydantic-settings` y exponer `settings` | Acceder a la BD ni conocer HTTP |
| `db.py` | Abrir/conexión MySQL (`DictCursor`, commit/rollback) usando `settings.db_*` | Ejecutar SQL de negocio |
| `apis/` | Recibir la petición, validar con `schemas/`, delegar al *service*, devolver la respuesta HTTP | Ejecutar SQL directamente |
| `schemas/` | Validar y normalizar los datos de entrada/salida (contrato del API) | Acceder a la base de datos |
| `services/` | Reglas de negocio, orquestación, transacciones | Conocer HTTP ni SQL directo |
| `repositories/` | Ejecutar consultas SQL parametrizadas (`%s`) y devolver resultados | Aplicar reglas de negocio |

## 3. Flujo de una petición

```
Cliente (Flutter)
      │  HTTP + JSON  (flutter_application_01/.env define la URL base)
      ▼
   apis/  ── valida con schemas/ ──▶ services/ ──▶ repositories/ ──▶ MySQL
      ▲                                                     │
      └───────────────── respuesta JSON ◀───────────────────┘
                            (db.py usa settings.db_* de backend/.env)
```

Reglas:

1. La petición **siempre** entra por `apis/`.
2. El payload **siempre** se valida en `schemas/` antes de llegar al *service*.
3. El SQL **solo** vive en `repositories/`, con parámetros **`%s`** para evitar inyección SQL.
4. `main.py` **solo** compone la aplicación; `config.py`/`db.py` son la única puerta al `.env`.

## 4. Ejemplo mínimo

### `src/backend/main.py`

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from backend.config import settings
from backend.apis import books

app = FastAPI(title="API de Booki")

app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origins_list,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(books.router, prefix="/api/v1", tags=["books"])
```

### `src/backend/apis/books.py`

```python
from fastapi import APIRouter, HTTPException
from backend.schemas.books import BookCreate
from backend.services import books as books_service

router = APIRouter()

@router.get("/")
def list_all():
    return books_service.get_all()

@router.post("/", status_code=201)
def create(payload: BookCreate):
    return books_service.create(payload.model_dump())
```

### `src/backend/repositories/books.py`

```python
from backend.db import get_connection

def find_all():
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("SELECT id, title FROM books ORDER BY id")
            return cur.fetchall()
```

## 5. Reglas transversales

- **Sin ORM:** solo **PyMySQL** (`uv add pydantic-settings pymysql`). Ninguna consulta se
  genera con SQLAlchemy, Peewee u otro ORM; el DDL se aplica a mano en MySQL y se documenta
  en cada spec técnica (sin migraciones automáticas).
- **Sin credenciales en el código:** todo sale de `backend/.env` vía `config.py` → `settings`
  (ver [environment-config.md](./environment-config.md)). Nada de `os.getenv` suelto.
- **SQL parametrizado:** siempre `%s` con tupla de parámetros; jamás concatenar entrada del usuario.
- **Errores:** los errores de negocio se traducen a códigos HTTP apropiados en la capa `apis/`.
- **Un archivo por recurso** con el mismo nombre en las cuatro capas:
  `apis/<recurso>.py` → `services/<recurso>.py` → `repositories/<recurso>.py` → `schemas/<recurso>.py`.
- **`services/` no importa FastAPI ni starlette**: se prueba sin levantar el servidor.

## 6. Relación con las historias de usuario

Cada HU que implique backend debe dejar evidencia en las cuatro capas:

| HU | `apis/` | `schemas/` | `services/` | `repositories/` |
|----|---------|-----------|-------------|-----------------|
| FEAT-001 Autenticación | ☐ | ☐ | ☐ | ☐ |
| FEAT-002 CRUD de Libros | ☐ | ☐ | ☐ | ☐ |
| [TODO] | ☐ | ☐ | ☐ | ☐ |
