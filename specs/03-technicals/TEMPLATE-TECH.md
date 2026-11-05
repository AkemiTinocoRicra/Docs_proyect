# Especificación Técnica: [Nombre de la Funcionalidad]

> Basado en: `docs/specs/02-features/NN-slug.md` · Reglas globales en `docs/specs/03-technicals/architecture-blueprint.md`

## 1. Arquitectura / Ubicación en el Código

```
backend/src/backend/
├── apis/<recurso>.py        # Endpoint (routers FastAPI) — solo valida y delega
├── services/<recurso>.py    # Lógica de negocio
├── repositories/<recurso>.py# SQL a mano (sin ORM)
└── schemas/<recurso>.py     # Pydantic: DTOs de request/response

flutter_application_01/lib/
├── models/<recurso>.dart        # DTO espejo de schemas/
├── services/<recurso>_service.dart  # Llamadas HTTP (una sola capa que habla con el backend)
├── viewmodels/<recurso>_viewmodel.dart  # Estado de la feature (ChangeNotifier)
└── views/<feature>/<pantalla>_screen.dart  # UI: solo pinta
```

- **Backend**: FastAPI en `backend/src/backend/`; registrar router en `main.py`.
- **Flutter**: MVVM; la View no hace HTTP, el ViewModel no importa widgets, el Service no tiene estado.

## 2. Capa de Datos

### 2.1 Esquema / Entidad

```sql
-- Tablas MySQL que toca esta feature — aplicar a mano, no hay ORM ni migraciones
CREATE TABLE ... ( ... );
```

```dart
// models/<recurso>.dart — espejo de schemas/<recurso>.py
class Recurso {
  final String id;
  ...
  factory Recurso.fromJson(Map<String, dynamic> json) => ...;
  Map<String, dynamic> toJson() => ...;
}
```

### 2.2 API

| Método | Ruta             | Body | Respuesta | Errores  |
| ------ | ---------------- | ---- | --------- | -------- |
| `GET`  | `/api/<recurso>` | -    | `200` lista | 404, 500 |

- **Implementación**: `apis/<recurso>.py` (routers) + `services/<recurso>.py`.
- **Validación de entrada/respuesta**: `schemas/<recurso>.py` (Pydantic).
- **`base_url`**: tomada de `Env.apiBaseUrl` (`flutter_application_01/.env` → `API_BASE_URL`).

### 2.3 Persistencia

- **MySQL vía PyMySQL — sin ORM.** SQL parametrizado (`%s`) en `repositories/<recurso>.py`;
  conexión desde `db.py` (`settings.db_*` leídos de `backend/.env`).
- Operaciones/consultas:

```sql
SELECT ... FROM ... WHERE ... = %s;
```

## 3. Lógica de Negocio / Estado (MVVM)

### ViewModel — estado expuesto

| Propiedad      | Tipo       | Descripción |
| -------------- | ---------- | ----------- |
| `uiState`      | `enum UiState` | `idle/loading/success/error` |
| `errorMessage` | `String?`  | ...         |

### Métodos/funciones públicas

```dart
class [Recurso]ViewModel extends ChangeNotifier {
  Future<void> load();
  Future<void> save();
}
```

### Services (Flutter)

```dart
class [Recurso]Service {
  Future<List<[Recurso]>> fetchAll();
}
```

### Services (Backend)

```python
def obtener_todos() -> list[RecursoOut]: ...
```

## 4. Integración con el Resto de la App

- **Ruta Flutter**: `'/...'` registrada en `lib/app/router.dart`.
- **Router/registro de ViewModel**: `ChangeNotifierProvider` en el arranque de la ruta.
- **Punto de llamada API**: `[Service] → Env.apiBaseUrl + '/api/<recurso>'`.

## 5. Dependencias

```bash
# flutter_application_01/
flutter pub add http provider   # solo si la feature los necesita

# backend/
uv add pydantic-settings pymysql
uv add --dev pytest httpx
```

- Nada nuevo: [dejar vacío]

## 6. Presupuesto / Rendimiento

- Esta feature añade ~[X] pantallas/peticiones. Sin presupuesto de bundle como en web:
  vigilar tiempo de arranque (`flutter run --profile` + DevTools) y tamaños de listas.

## 7. Estrategia de Testing (para QA)

### Tests unitarios (Flutter)

- [ ] ViewModel: `load()` pasa `loading → success` y `loading → error` con el service mockeado.
- [ ] Modelo: `fromJson`/`toJson` con el payload real del backend.

### Tests unitarios (Backend)

- [ ] `services/`: reglas de negocio sin levantar el servidor.
- [ ] `repositories/`: consultas contra BD de prueba (o mock del cursor).

### Tests de integración

- [ ] Flujo completo: pantalla → ViewModel → Service → endpoint → respuesta pintada.

### Comandos

```bash
cd flutter_application_01
flutter analyze
flutter test

cd backend
uv run pytest
```

## 8. Consideraciones Futuras

- [Autenticación, paginación, caché offline, accesibilidad, escalabilidad]
