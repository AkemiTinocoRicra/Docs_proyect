# Plano de Arquitectura Técnica

> **[TODO: fecha de verificación]** — verificar contra el código y actualizar este documento
> si cambia.
> **Arquitectura objetivo**: se documenta a continuación.
> **Estado real**: [TODO: qué existe hoy en `backend/` y `flutter_application_01/`].

## Decisiones Confirmadas

| Área | Decisión |
| ---- | -------- |
| Requisitos | **Historias de usuario** (formato Como/Quiero/Para + criterios Gherkin) en `docs/specs/02-features/` |
| App | **Flutter** con patrón **MVVM** (View ← ViewModel ← Model/Service) |
| Backend | **FastAPI** en `backend/`: `main.py` → `apis/` → `services/` → `repositories/` (+ `schemas/`) |
| Persistencia | **Sin ORM**: SQL a mano con driver, sobre **MySQL** (`DB_NAME=[TODO]`) |
| BD ↔ Backend | `backend/.env` (leído con `pydantic-settings`) |
| Backend ↔ Flutter | `flutter_application_01/.env` (leído con `flutter_dotenv`) |

## Estructura General

El repo contiene **dos proyectos independientes** (cada uno con su propio git repo, sin manifest en la raíz):

```
Booki_01/
├── backend/                  # Backend FastAPI (uv, Python [TODO])
├── flutter_application_01/   # App Flutter (MVVM)
└── docs/                     # Este directorio (no es un proyecto)
```

**Comunicación**: [TODO: existe / pendiente]. Las APIs viven en **`backend/` (FastAPI)**.

## Arquitectura Objetivo (por capas)

### Backend: `main.py` → apis → services → repositories (+ esquemas)

```
backend/src/backend/
├── main.py            # App factory: crea FastAPI, registra CORS desde settings, monta routers, arranque uvicorn
├── config.py          # pydantic-settings: lee backend/.env → settings (DB_*, BACKEND_HOST/PORT, CORS_ORIGINS)
├── db.py              # Conexión MySQL (PyMySQL): get_connection() / context manager (commit/rollback)
├── apis/              # Capa API (routers): recibe la petición, valida con schemas y llama a services
│   └── <recurso>.py   #   Decoradores @router.get/post/... — sin lógica de negocio
├── services/          # Capa de servicio: lógica de negocio, reglas, orquestación
│   └── <recurso>.py   #   No conoce HTTP ni el detalle del almacenamiento
├── repositories/      # Capa de repositorio: SQL a mano (sin ORM) contra MySQL
│   └── <recurso>.py   #   Consultas/CRUD con parámetros — único punto que toca la BD
└── schemas/           # Modelos Pydantic: DTOs de request/response (contrato de la API)
    └── <recurso>.py   #   También aquí los modelos de dominio que devuelven los repositories
```

Flujo de una petición:

```
HTTP → apis/ → services/ → repositories/ → MySQL
                 ↑               ↑
            schemas/ (valida la entrada y modela la salida)
```

Reglas:

- `apis/` solo valida (`schemas/`) y delega en `services/`; nunca accede a datos directamente.
- `services/` nunca importa `FastAPI` ni `starlette`; se puede probar sin levantar el servidor.
- `repositories/` es el **único** que toca la base de datos, y siempre con **SQL parametrizado**.
- Todo payload que entra o sale pasa por `schemas/` (Pydantic) — es el contrato del API.

#### Sin ORM: acceso a datos con SQL a mano

- Driver: **PyMySQL** (`uv add pymysql`) — **no** SQLAlchemy, Tortoise, peewee ni ningún ORM.
- Las consultas viven en `repositories/`; **nunca** se concatena usuario en el SQL: siempre
  `%s` con la tupla de parámetros de pymysql (protege contra SQL injection).
- Conexión centralizada en `db.py`: lee `settings.db_*` (del `.env`), `DictCursor`,
  `autocommit=False` y se hace `commit()` en operaciones de escritura / `rollback()` en error.
- El esquema de tablas (DDL) **no** se gestiona desde el código: se documenta en las specs
  y se aplica a mano en MySQL. No hay migraciones automáticas (consecuencia de no usar ORM).

```python
# repositories/<recurso>.py — patrón de referencia
def get_by_id(conn, item_id: str) -> dict | None:
    with conn.cursor() as cur:
        cur.execute("SELECT ... FROM ... WHERE id = %s", (item_id,))
        return cur.fetchone()
```

### Flutter: View → ViewModel → Model (MVVM)

```
flutter_application_01/lib/
├── main.dart              # await dotenv.load(); runApp(...)
├── app/
│   ├── app.dart           # MaterialApp (tema, rutas)
│   └── router.dart        # rutas nombradas de la app
├── core/
│   ├── env.dart           # Env.apiBaseUrl (envuelve flutter_dotenv) — única fuente de la URL base
│   ├── api_client.dart    # Cliente HTTP (http): base URL, headers, timeouts, manejo de errores
│   └── errors.dart        # AppException (network / server / notFound / validation)
├── models/                # DTOs Dart: espejo de backend/schemas/ (mismos nombres y campos)
│   └── <recurso>.dart
├── services/              # Capa de datos: llamadas HTTP por recurso (única que habla con el backend)
│   └── <recurso>_service.dart
├── viewmodels/            # Estado de cada feature: ChangeNotifier + lógica de UI
│   └── <recurso>_viewmodel.dart
└── views/                 # UI: páginas y widgets — solo pintan, no hacen HTTP ni parsean JSON
    └── <feature>/
        ├── <pantalla>_screen.dart
        └── widgets/       #   widgets propios de la pantalla
```

Flujo de datos:

```
views/<pantalla>_screen.dart
        │  (escucha: context.watch / ListenableBuilder)
        ▼
viewmodels/<recurso>_viewmodel.dart   ← estado (UiState: idle/loading/success/error)
        │  (llama a)
        ▼
services/<recurso>_service.dart  ──HTTP──►  backend/apis/  (usa Env.apiBaseUrl del .env)
        ▼
models/<recurso>.dart            ◄── JSON ── backend/schemas/ (mismos campos)
```

Reglas de MVVM:

- La **View** (`views/`) solo renderiza estado y envía eventos al ViewModel. **Nunca** hace
  `http`, **nunca** parsea JSON, **nunca** conoce endpoints.
- El **ViewModel** (`viewmodels/`) expone el estado (campos `bool isLoading`, `String? error`,
  `List<X> items` o un `enum UiState`) y los métodos que la View invoca (`load()`, `save()`).
  **No** importa widgets de Material más allá de lo imprescindible; se puede testear sin UI.
- El **Model** es `models/` (DTOs) + `services/` (acceso a datos). Los DTOs deben reflejar los
  `schemas/` del backend (mismos nombres/campos) para que el analyzer falle si el contrato cambia.
- Registro de ViewModels: `ChangeNotifierProvider` en el arranque de la feature (raíz del árbol
  de esa pantalla), de modo que al navegar se cree/destruya con la ruta.

Estado mínimo que expone cada ViewModel:

| Propiedad       | Tipo            | Descripción                         |
| --------------- | --------------- | ----------------------------------- |
| `uiState`       | `enum UiState`  | `idle / loading / success / error`  |
| `errorMessage`  | `String?`       | Mensaje mostrable (null si no hay)  |
| `items` / modelo| dominio         | Datos a pintar                      |

## Configuración: archivos `.env`

**Nada se hardcodea.** Cada proyecto tiene **su propio** `.env` (no hay `.env` en la raíz:
los proyectos son independientes). Flutter no escribe la URL del backend a mano, y el backend
no escribe credenciales de BD a mano.

| Proyecto     | Archivo                        | Quién lo lee                                        | Variables                                                                                             |
| ------------ | ------------------------------ | --------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Backend      | `backend/.env`                 | `pydantic-settings` → `src/backend/config.py` (`settings`) | `BACKEND_HOST`, `BACKEND_PORT`, `CORS_ORIGINS`, `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` |
| Flutter      | `flutter_application_01/.env`  | `flutter_dotenv` → `lib/core/env.dart` (`Env.apiBaseUrl`)   | `API_BASE_URL`                                                                                 |

En ambos casos: **`.env` NO se commitea** (`.gitignore`) y **sí se commitea `.env.example`**
sin valores reales, para que alguien nuevo sepa qué variables esperar.

### Backend ↔ BD: `backend/.env`

```env
BACKEND_HOST=127.0.0.1
BACKEND_PORT=8000
CORS_ORIGINS=http://localhost:8080
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=[TODO]
DB_PASSWORD=[TODO]
DB_NAME=[TODO]
```

- Se lee con **`pydantic-settings`** (`uv add pydantic-settings`) en `src/backend/config.py`
  → `settings`; el resto del código importa `settings`, nunca `os.environ` directo.
- `main.py` usa `settings.cors_origins` (parseado a lista) para `CORSMiddleware` y
  `settings.backend_port`/`settings.backend_host` para arrancar con uvicorn.
- `repositories/` toma las credenciales **solo** desde `settings.db_*` (vía `db.py`).
- `backend/.gitignore` debe ignorar `.env` (sección `# Secrets`).

### Backend ↔ Flutter: `flutter_application_01/.env`

Flutter **no lee `.env` nativamente**. Se usa **`flutter_dotenv`**, que carga el archivo como
asset en tiempo de ejecución:

```bash
cd flutter_application_01
flutter pub add flutter_dotenv
```

`pubspec.yaml` (resultado + asset obligatorio):

```yaml
dependencies:
  flutter_dotenv: ^5.x   # la versión exacta la fija `flutter pub add`

flutter:
  uses-material-design: true
  assets:
    - .env
```

`lib/main.dart`:

```dart
Future<void> main() async {
  await dotenv.load();
  runApp(const MainApp());
}
```

`lib/core/env.dart` (único punto de acceso; el resto del código no toca `dotenv` directo):

```dart
class Env {
  static final String apiBaseUrl = dotenv.env['API_BASE_URL'] ?? 'http://localhost:8000';
}
```

`flutter_application_01/.env`:

```env
API_BASE_URL=http://10.0.2.2:8000
```

Valor de `API_BASE_URL` según dónde se ejecute la app:

| Entorno                        | Valor                      |
| ------------------------------ | -------------------------- |
| Emulador Android               | `http://10.0.2.2:8000` (el host se ve como 10.0.2.2) |
| Emulador/simulador iOS         | `http://localhost:8000`    |
| Dispositivo físico (misma red) | `http://192.168.x.x:8000`  |

- Alternativa (sin asset): **`--dart-define=API_BASE_URL=...`** + `String.fromEnvironment`.
  Útil para CI/producción; el `.env` + `flutter_dotenv` es el flujo diario de desarrollo.
- **Nunca secretos en el `.env` de Flutter**: todo lo que ahí viaje se empaqueta dentro del
  APK/IPA y es visible para cualquiera. Solo URLs y valores no sensibles.
- Cambiar el `.env` obliga a **rebuild** (`flutter run` de nuevo), no alcanza con hot reload.
- `flutter_application_01/.gitignore` debe ignorar `.env`.

### Verificación

| Acción                     | Comando                                                                                     |
| -------------------------- | ------------------------------------------------------------------------------------------- |
| Backend carga su `.env`    | `cd backend && uv run python -c "from backend.config import settings; print(settings.db_name)"` |
| Flutter lee su `.env`      | log temporal de `Env.apiBaseUrl` en `main.dart` + comprobar que arranca sin error            |
| Flutter asset registrado   | `flutter run` y verificar que no lanza `Unable to load asset: .env`                         |

## Backend: `backend/`

- **Runtime**: Python [TODO] (`.python-version` = [TODO]).
- **Layout**: `src/backend/` (src-layout de `uv_build`).
- **Entry point actual**: [TODO].
- **Entry point objetivo**: `src/backend/main.py` con la app factory; al crearlo, actualizar la
  entrada de `pyproject.toml` (o exponer con `uv run uvicorn backend.main:app --reload`).
- **Dependencias actuales**: `fastapi`, `uvicorn`.
- **Dependencias a añadir**: `uv add pydantic-settings pymysql` · dev: `uv add --dev pytest httpx`.
- **Configuración**: `backend/.env` leído con `pydantic-settings` en `config.py` (ver arriba).

### Comandos (ejecutar desde `backend/`)

| Acción            | Comando                                              |
| ----------------- | ---------------------------------------------------- |
| Instalar deps     | `uv sync`                                            |
| Añadir dependencia| `uv add <paquete>` / `uv add --dev <paquete>`        |
| Levantar API      | `uv run uvicorn backend.main:app --reload`           |
| Verificar config  | `uv run python -c "from backend.config import settings; print(settings)"` |
| Tests             | `uv run pytest`                                      |

## Flutter: `flutter_application_01/`

- **Framework**: Flutter [TODO] stable (Dart SDK [TODO]), `flutter_lints` activo.
- **Arquitectura**: MVVM (ver [Arquitectura Objetivo](#arquitectura-objetivo-por-capas)).
- **Estado**: `ChangeNotifier` + `provider`.
- **HTTP**: paquete `http` envuelto en `core/api_client.dart` (una sola clase con la base URL).
- **Rutas**: `MaterialApp.routes` / `onGenerateRoute` en `app/router.dart`.
- **Tema**: Material 3 (`ColorScheme.fromSeed`) en `app/app.dart`.
- **Estado actual**: [TODO].
- **Tests**: `flutter_test` en `pubspec.yaml`; [TODO: existe carpeta `test/`].

### Comandos (desde `flutter_application_01/`)

| Acción           | Comando                     |
| ---------------- | --------------------------- |
| Instalar deps    | `flutter pub get`           |
| Ejecutar         | `flutter run`               |
| Analizar         | `flutter analyze`           |
| Tests            | `flutter test` (requiere `test/`) |
| Formato          | `dart format lib/`          |

## Restricciones que condicionan el diseño

- **Sin ORM** ⇒ sin migraciones automáticas ni `migrate`: el DDL se aplica a mano en MySQL y
  se documenta en cada spec (tabla, columnas, tipos, UNIQUE/FK).
- **CORS no protege a la app nativa**: las apps móviles no envían `Origin`, así que CORS solo
  importa para `flutter run -d chrome`/web. Las credenciales de BD **jamás** viajan al cliente.
- **`.env` de Flutter va empaquetado**: no usar para secretos.
- Emulador Android no ve `localhost`: usar `10.0.2.2` (tabla arriba).
- [TODO: plataformas generadas — solo `android/` y `web/`?]

## Consideraciones Futuras

- [TODO] Añadir `test/` en Flutter y `pytest` en backend (ver `05-testing/qa-strategy.md`).
- [TODO] Capa de autenticación (JWT con `JWT_SECRET_KEY`/`JWT_EXPIRE_MIN` en `backend/.env`).
- [TODO] CI (flutter analyze + flutter test + pytest).
