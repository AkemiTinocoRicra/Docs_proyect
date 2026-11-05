# Archivos `.env` (Backend ↔ BD y Backend ↔ Flutter)

> **Estado:** [TODO: definido / en revisión]
> **Última actualización:** [TODO: AAAA-MM-DD]
> Reglas globales en [architecture-blueprint.md](./architecture-blueprint.md).

Se definen **dos archivos `.env`**, uno por cada flujo de comunicación.

## 1. Visión general

| # | Archivo | Ubicación | Comunica | Quién lo lee |
|---|---------|-----------|----------|--------------|
| 1 | `.env` | `backend/.env` | **Backend → Base de datos** | `pydantic-settings` → `src/backend/config.py` (`settings`) |
| 2 | `.env` | `flutter_application_01/.env` | **Flutter → Backend** | `flutter_dotenv` → `lib/core/env.dart` (`Env.apiBaseUrl`) |

## 2. `.env` del backend (Backend ↔ BD)

### `backend/.env`

```env
# --- Servidor ---
BACKEND_HOST=127.0.0.1
BACKEND_PORT=8000
CORS_ORIGINS=http://localhost:8080

# --- Base de datos (sin ORM, conexión directa con driver) ---
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=[TODO]
DB_USER=[TODO]
DB_PASSWORD=[TODO]
```

### `backend/.env.example`

```env
BACKEND_HOST=127.0.0.1
BACKEND_PORT=8000
CORS_ORIGINS=http://localhost:8080
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=
DB_USER=
DB_PASSWORD=
```

### Uso en el código

Se lee **una sola vez** con `pydantic-settings` (`uv add pydantic-settings`); el resto del
código importa `settings`, nunca `os.getenv` directo:

```python
# src/backend/config.py
from pydantic_settings import BaseSettings, SettingsConfigDict
from pathlib import Path

ENV_PATH = Path(__file__).resolve().parents[2] / ".env"   # backend/.env

class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=ENV_PATH)

    backend_host: str = "127.0.0.1"
    backend_port: int = 8000
    cors_origins: str = "http://localhost:8080"

    db_host: str = "127.0.0.1"
    db_port: int = 3306
    db_name: str = ""
    db_user: str = ""
    db_password: str = ""

    @property
    def cors_origins_list(self) -> list[str]:
        return [o.strip() for o in self.cors_origins.split(",") if o.strip()]

settings = Settings()
```

Conexión a la BD (única puerta a MySQL, en `db.py`):

```python
# src/backend/db.py
import pymysql
from backend.config import settings

def get_connection():
    return pymysql.connect(
        host=settings.db_host,
        port=settings.db_port,
        database=settings.db_name,
        user=settings.db_user,
        password=settings.db_password,
        cursorclass=pymysql.cursors.DictCursor,
        autocommit=False,
    )
```

## 3. `.env` de Flutter (Backend ↔ Flutter)

### `flutter_application_01/.env`

```env
# URL base del backend consumido por la app
API_BASE_URL=http://10.0.2.2:8000
API_TIMEOUT=30000
```

### `flutter_application_01/.env.example`

```env
API_BASE_URL=
API_TIMEOUT=30000
```

### Uso en el código

**Método elegido para este proyecto: `flutter_dotenv`** (el `.env` viaja como asset):

```bash
cd flutter_application_01
flutter pub add flutter_dotenv
```

```yaml
# pubspec.yaml
dependencies:
  flutter_dotenv: ^5.x

flutter:
  uses-material-design: true
  assets:
    - .env
```

```dart
// lib/main.dart
Future<void> main() async {
  await dotenv.load();
  runApp(const MainApp());
}

// lib/core/env.dart — único punto de acceso; el resto no toca dotenv directo
import 'package:flutter_dotenv/flutter_dotenv.dart';

class Env {
  static String get apiBaseUrl => dotenv.env['API_BASE_URL'] ?? 'http://localhost:8000';
  static int get apiTimeout => int.tryParse(dotenv.env['API_TIMEOUT'] ?? '') ?? 30000;
}
```

> Alternativa (sin asset, útil en CI/producción): inyectar en tiempo de compilación con
> `--dart-define=API_BASE_URL=...` y leer con `String.fromEnvironment('API_BASE_URL')`.

## 4. Reglas de seguridad

- [ ] `.env` está en `.gitignore` (solo se versiona `.env.example`).
- [ ] No hay credenciales ni URLs de producción en el código fuente.
- [ ] Cada entorno (local, pruebas, producción) tiene su propio `.env`.
- [ ] El `.env.example` documenta **todas** las variables requeridas sin valores sensibles.
- [ ] **Nunca secretos en el `.env` de Flutter**: va empaquetado en el APK; solo URLs y
      valores no sensibles.

## 5. `.gitignore` mínimo

```gitignore
# Backend
backend/.env

# Flutter
flutter_application_01/.env
```

## 6. Valores de `API_BASE_URL` por entorno

| Entorno                        | Valor                      |
| ------------------------------ | -------------------------- |
| Emulador Android               | `http://10.0.2.2:8000`     |
| Emulador/simulador iOS         | `http://localhost:8000`    |
| Dispositivo físico (misma red) | `http://192.168.x.x:8000`  |

> Cambiar el `.env` de Flutter obliga a **rebuild** (`flutter run` de nuevo); el hot reload no alcanza.

## 7. Verificación

| Acción                    | Comando |
| ------------------------- | ------- |
| Backend carga su `.env`   | `cd backend && uv run python -c "from backend.config import settings; print(settings.db_name)"` |
| Flutter asset registrado  | `flutter run` sin error `Unable to load asset: .env` |
