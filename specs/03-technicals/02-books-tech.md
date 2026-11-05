# Especificación Técnica: CRUD de Libros

> Basado en: `docs/specs/02-features/02-books.md` · Reglas globales en `docs/specs/03-technicals/architecture-blueprint.md`

## 1. Arquitectura / Ubicación en el Código

```
backend/src/backend/
├── apis/books.py          # Endpoint (routers FastAPI) — solo valida y delega
├── services/books.py      # Lógica de negocio
├── repositories/books.py  # SQL a mano (sin ORM)
└── schemas/books.py       # Pydantic: DTOs de request/response

flutter_application_01/lib/
├── models/book.dart               # DTO espejo de schemas/books.py
├── services/book_service.dart     # Llamadas HTTP (única capa que habla con el backend)
├── viewmodels/book_viewmodel.dart # Estado de la feature (ChangeNotifier)
└── views/books/…_screen.dart      # UI: solo pinta
```

- **Backend**: FastAPI en `backend/src/backend/`; registrar el router en `main.py`.
- **Flutter**: MVVM; la View no hace HTTP, el ViewModel no importa widgets, el Service no tiene estado.

## 2. Capa de Datos

### 2.1 Esquema / Entidad

```sql
-- Tabla MySQL que toca esta feature — aplicar a mano, no hay ORM ni migraciones
-- [TODO: DB_NAME]
CREATE TABLE Book (
  -- [TODO: columnas, tipos, PK, UNIQUE, filas de prueba]
);
```

```dart
// models/book.dart — espejo de schemas/books.py
class Book {
  final String id;
  // [TODO: campos]
  factory Book.fromJson(Map<String, dynamic> json) => ...;
  Map<String, dynamic> toJson() => ...;
}
```

### 2.2 API

| Método | Ruta                | Body        | Respuesta     | Errores     |
| ------ | ------------------- | ----------- | ------------- | ----------- |
| `GET`  | `/api/books`        | -           | `200` lista   | 500         |
| `GET`  | `/api/books/{id}`   | -           | `200` objeto  | 404, 500    |
| `POST` | `/api/books`        | `BookIn`    | `201` objeto  | 422, 500    |
| `PUT`  | `/api/books/{id}`   | `BookIn`    | `200` objeto  | 404, 422    |
| `DELETE`| `/api/books/{id}`  | -           | `204`         | 404, 500    |

- **Implementación**: `apis/books.py` (routers) + `services/books.py`.
- **Validación de entrada/respuesta**: `schemas/books.py` (Pydantic).
- **`base_url`**: tomada de `Env.apiBaseUrl` (`flutter_application_01/.env` → `API_BASE_URL`).

### 2.3 Persistencia

- **MySQL vía PyMySQL — sin ORM.** SQL parametrizado (`%s`) en `repositories/books.py`;
  conexión desde `db.py` (`settings.db_*` leídos de `backend/.env`).
- Operaciones/consultas:

```sql
-- [TODO: consultas reales]
SELECT ... FROM Book WHERE ... = %s;
```

## 3. Lógica de Negocio / Estado (MVVM)

### ViewModel — estado expuesto

| Propiedad      | Tipo            | Descripción |
| -------------- | --------------- | ----------- |
| `uiState`      | `enum UiState`  | `idle/loading/success/error` |
| `errorMessage` | `String?`       | [TODO]      |

### Métodos/funciones públicas

```dart
class BookViewModel extends ChangeNotifier {
  Future<void> load();
  Future<void> save();
  // [TODO]
}
```

### Services (Flutter)

```dart
class BookService {
  Future<List<Book>> fetchAll();
  // [TODO]
}
```

### Services (Backend)

```python
def obtener_todos() -> list[BookOut]: ...
# [TODO]
```

## 4. Integración con el Resto de la App

- **Ruta Flutter**: `'/...'` registrada en `lib/app/router.dart`.
- **Router/registro de ViewModel**: `ChangeNotifierProvider` en el arranque de la ruta.
- **Punto de llamada API**: `BookService → Env.apiBaseUrl + '/api/books'`.

## 5. Dependencias

```bash
# flutter_application_01/
flutter pub add http provider

# backend/
uv add pydantic-settings pymysql
uv add --dev pytest httpx
```

- Nada nuevo: [dejar vacío si ya están instaladas]

## 6. Presupuesto / Rendimiento

- Esta feature añade ~[TODO] pantallas/peticiones. Vigilar tiempo de arranque
  (`flutter run --profile` + DevTools) y tamaños de listas.

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

- [TODO: paginación, búsqueda, autenticación, caché offline]
