# Especificación Técnica: Autenticación (Crear cuenta e iniciar sesión)

> Basado en: `docs/specs/02-features/01-auth.md` · Reglas globales en `docs/specs/03-technicals/architecture-blueprint.md`

## 1. Arquitectura / Ubicación en el Código

```
backend/src/backend/
├── apis/auth.py               # Routers: POST /registro, POST /login — solo valida y delega
├── apis/grados.py             # Router público: GET /grados (catálogo del registro)
├── services/auth.py           # Lógica de negocio: registrar, autenticar, emitir JWT
├── services/grados.py         # Lógica del catálogo de grados
├── repositories/estudiantes.py# SQL a mano (sin ORM) sobre la tabla estudiante
├── repositories/grados.py     # SQL a mano (sin ORM) sobre la tabla grado
├── schemas/auth.py            # Pydantic: DTOs de request/response
└── schemas/grados.py          # Pydantic: GradoOut

flutter_application_01/lib/
├── models/auth_models.dart         # DTOs: RegistroRequest, LoginRequest, AuthResponse
├── models/grado.dart               # DTO del catálogo de grados
├── services/auth_service.dart      # Llamadas HTTP (única capa que habla con el backend)
├── services/grado_service.dart     # GET /api/grados (público)
├── services/session_service.dart   # Persistencia del JWT en shared_preferences
├── viewmodels/auth_viewmodel.dart  # Estado de la feature (ChangeNotifier)
├── viewmodels/grados_viewmodel.dart# Catálogo de grados para el formulario
└── views/auth/
    ├── login_screen.dart
    └── registro_screen.dart
```

- **Backend**: FastAPI en `backend/src/backend/`; registrar el router `auth` en `main.py`.
- **Flutter**: MVVM; la View no hace HTTP, el ViewModel no importa widgets, el Service no tiene estado.

## 2. Capa de Datos

### 2.1 Esquema / Entidad (tabla existente — **sin DDL**)

La tabla **ya existe** en MySQL (`DB_NAME=Booki`): **`estudiante`**. Esta spec **no crea ni modifica** tablas; solo documenta las columnas con las que trabaja esta feature y las consultas que la capa `repositories/` ejecutará.

| Columna           | Tipo           | Null  | Notas                                                     |
| ----------------- | -------------- | ----- | --------------------------------------------------------- |
| `id_estudiante`   | `INT` AUTO_INCREMENT | NO | PK                                                       |
| `dni`             | `CHAR(8)`      | NO    | **UNIQUE** — 8 dígitos numéricos                          |
| `nombres`         | `VARCHAR(60)`  | NO    | Pedido en el registro                                     |
| `apellidos`       | `VARCHAR(60)`  | NO    | Pedido en el registro                                     |
| `fecha_nacimiento`| `DATE`         | SÍ    | Pedido en el registro (puede quedar NULL)                 |
| `contrasena`      | `VARCHAR(4)`   | SÍ    | 4 dígitos numéricos **en claro** (esquema existente)      |
| `cantidad_copas`  | `INT`          | NO    | Default `0` — lo rellena el backend, no el formulario     |
| `estado`          | `CHAR(2)`      | SÍ    | `'AC'` = activo, `'IN'` = inactivo; el registro pone `'AC'` |
| `id_docente`      | `INT`          | SÍ   | Opcional / Asociado posteriormente                        |
| `id_grado`        | `INT`          | NO    | Elegido por el estudiante al registrarse                  |

> El catálogo de `id_grado` **no** se hardcodea: se lee de la tabla `grado` con `GET /api/grados` (solo `estado = 'AC'`, ordenado por `id_grado`).

### 2.1.1 Catálogo de grados (tabla existente — **sin DDL**)

La tabla **`grado`** ya existe y solo se consulta; esta feature no la modifica.

| Columna     | Tipo          | Null | Notas                                          |
| ----------- | ------------- | ---- | ---------------------------------------------- |
| `id_grado`  | `INT`         | NO   | PK                                             |
| `nombres`   | `VARCHAR`     | NO   | Nombre del grado                               |
| `nivel`     | `VARCHAR`     | NO   | Nivel educativo (p. ej. "Primero")             |
| `estado`    | `CHAR(2)`     | SÍ   | `'AC'` = activo, `'IN'` = inactivo              |

- `GET /api/grados` devuelve **solo los activos**, ordenados por `id_grado`.
- Es **público** (sin JWT): el estudiante aún no tiene token cuando se registra.
- El registro valida que `id_grado` exista; si no, `422` "Grado no válido".

```dart
// models/auth_models.dart — espejo de schemas/auth.py
class RegistroRequest {
  final String dni;            // 8 dígitos
  final String nombres;
  final String apellidos;
  final String? fechaNacimiento; // ISO yyyy-MM-dd o null
  final String contrasena;     // 4 dígitos
  final int idGrado;
  factory RegistroRequest.fromJson(Map<String, dynamic> json) => ...;
  Map<String, dynamic> toJson() => ...;
}

class LoginRequest {
  final String dni;
  final String contrasena;
  factory LoginRequest.fromJson(Map<String, dynamic> json) => ...;
  Map<String, dynamic> toJson() => ...;
}

class AuthResponse {
  final String token;   // JWT
  final String dni;
  final String role;    // 'estudiante'
  factory AuthResponse.fromJson(Map<String, dynamic> json) => ...;
  Map<String, dynamic> toJson() => ...;
}
```

### 2.2 API

| Método | Ruta                 | Body            | Respuesta        | Errores           |
| ------ | -------------------- | --------------- | ---------------- | ----------------- |
| `POST` | `/api/auth/registro` | `RegistroIn`    | `201` + JWT      | 409, 422, 500     |
| `POST` | `/api/auth/login`    | `LoginIn`       | `200` + JWT      | 401, 422, 500     |
| `GET`  | `/api/grados`        | -               | `200` lista de grados | 500          |

- `GET /api/grados` es **público** (sin `Authorization`); devuelve `[{"id_grado", "nombres", "nivel"}]` con los grados `estado = 'AC'` ordenados por `id_grado`.

- **Implementación**: `apis/auth.py` (routers) + `services/auth.py`.
- **Validación de entrada/respuesta**: `schemas/auth.py` (Pydantic).
  - `dni`: patrón `^\d{8}$`
  - `contrasena`: patrón `^\d{4}$`
- **`base_url`**: tomada de `Env.apiBaseUrl` (`flutter_application_01/.env` → `API_BASE_URL`).

Contratos de salida:

- `201` registro: `{"token": "<jwt>", "dni": "12345678", "role": "estudiante"}`
- `200` login: mismo cuerpo que el registro.
- `409` registro: `{"detail": "El DNI ya está registrado"}`
- `401` login: `{"detail": "DNI o contraseña incorrectos"}`
- `422` registro: `{"detail": "Grado no válido"}` (el `id_grado` no existe)
- `200` grados: `[{"id_grado": 1, "nombres": "Primero", "nivel": "Primaria"}]`

### 2.3 Persistencia

- **MySQL vía PyMySQL — sin ORM.** SQL parametrizado (`%s`) en `repositories/estudiantes.py`; conexión desde `db.py` (`settings.db_*` leídos de `backend/.env`).
- Operaciones/consultas:

```sql
-- ¿Existe el DNI? (login y unicidad de registro)
SELECT id_estudiante, dni, contrasena, estado
FROM estudiante
WHERE dni = %s;

-- Alta de cuenta (campos que rellena el backend entre paréntesis)
INSERT INTO estudiante
  (dni, nombres, apellidos, fecha_nacimiento, contrasena,
   cantidad_copas, estado, id_grado)
VALUES (%s, %s, %s, %s, %s, 0, 'AC', %s);

-- Catálogo de grados para el registro (público, solo activos)
SELECT id_grado, nombres, nivel
FROM grado
WHERE estado = 'AC'
ORDER BY id_grado;

-- Validación del id_grado enviado en el registro
SELECT 1 FROM grado WHERE id_grado = %s;
```

> El catálogo de `id_grado` vive en `repositories/grados.py` y nunca se hardcodea en el cliente.

## 3. Lógica de Negocio / Estado (MVVM)

### ViewModel — estado expuesto

| Propiedad       | Tipo            | Descripción                                        |
| --------------- | --------------- | -------------------------------------------------- |
| `uiState`       | `enum UiState`  | `idle/loading/success/error`                       |
| `errorMessage`  | `String?`       | Mensaje mostrable (null si no hay error)           |
| `isAuthenticated` | `bool`        | true si hay JWT válido (memoria o persistido)      |

### Métodos/funciones públicas

```dart
class AuthViewModel extends ChangeNotifier {
  Future<void> register({
    required String dni,
    required String nombres,
    required String apellidos,
    String? fechaNacimiento,
    required String contrasena,
    required String confirmarContrasena,
    required int idGrado,
  });
  Future<void> login({required String dni, required String contrasena});
  Future<void> restoreSession(); // al arrancar: lee shared_preferences
  Future<void> logout();         // borra token, isAuthenticated = false
}
```

Validaciones en el ViewModel (antes del Service):

- DNI: 8 dígitos numéricos.
- Contraseña: exactamente 4 dígitos numéricos.
- Registro: contrasena == confirmarContrasena; nombres y apellidos no vacíos.

### ViewModel — catálogo de grados

| Propiedad      | Tipo              | Descripción                                       |
| -------------- | ----------------- | ------------------------------------------------- |
| `grados`       | `List<Grado>`     | Grados activos traídos de `GET /api/grados`       |
| `uiState`      | `GradosUiState`   | `idle/loading/success/error`                      |
| `errorMessage` | `String?`         | Error al cargar el catálogo                       |

```dart
class GradosViewModel extends ChangeNotifier {
  Future<void> cargar();               // GET /api/grados
  Grado? buscar(int idGrado);          // mapea id_grado → etiqueta visible
}
```

- Se carga al abrir el registro (y el formulario de estudiantes), antes de habilitar el dropdown.
- `loading`: dropdown deshabilitado. `error`: mensaje + "Reintentar". `success` vacío: "No hay grados disponibles".
- El dropdown se puebla **solo** con `grados`; no se genera `1..6` en el cliente.

### Services (Flutter)

```dart
class AuthService {
  Future<AuthResponse> registrar(RegistroRequest request); // POST /api/auth/registro
  Future<AuthResponse> login(LoginRequest request);        // POST /api/auth/login
}

class SessionService {
  Future<void> saveToken(String token);   // shared_preferences
  String? readToken();
  Future<void> clearToken();
}

class GradoService {
  Future<List<Grado>> fetchAll();         // GET /api/grados (público, sin token)
}
```

### Services (Backend)

```python
# services/auth.py — sin FastAPI ni starlette
def registrar(datos: RegistroIn) -> AuthOut: ...
def autenticar(dni: str, contrasena: str) -> AuthOut: ...

# services/grados.py
def obtener_activos() -> list[GradoOut]: ...
def existe(id_grado: int) -> bool: ...
```

- `registrar`: verifica unicidad del DNI → 409 si existe; verifica que `id_grado` exista → 422 si no; INSERT con defaults; firma y devuelve JWT.
- `autenticar`: busca por DNI, compara `contrasena`, verifica `estado = 'AC'`; si falla → 401; si OK → firma JWT.
- **JWT**: `PyJWT`; payload con `sub` = `id_estudiante`, `dni`, `role = 'estudiante'`, `exp`; secret y expiración desde `settings` (`JWT_SECRET_KEY`, `JWT_EXPIRE_MIN`).

## 4. Integración con el Resto de la App

- **Rutas Flutter**: `'/login'` y `'/registro'` en `lib/app/router.dart`; el arranque decide `'/home'` vs `'/login'` según `restoreSession()`.
- **Router/registro de ViewModel**: `ChangeNotifierProvider<AuthViewModel>` y `ChangeNotifierProvider<GradosViewModel>` en la raíz de la app (sesión global y catálogo compartido con el formulario de estudiantes).
- **Punto de llamada API**: `AuthService → Env.apiBaseUrl + '/api/auth/...'`; `GradoService → Env.apiBaseUrl + '/api/grados'`.
- **Token**: `SessionService` (shared_preferences) — la app lo reutiliza en features futuras (p. ej. headers Authorization).

## 5. Dependencias

```bash
# flutter_application_01/
flutter pub add http provider flutter_dotenv shared_preferences

# backend/
uv add pydantic-settings pymysql pyjwt
uv add --dev pytest httpx
```

- Nada nuevo si ya están instaladas en una feature previa.

## 6. Presupuesto / Rendimiento

- Esta feature añade **2 pantallas** y **2 peticiones** (registro y login), más la lectura local del token al arrancar. Sin listas grandes ni imágenes pesadas; el costo de arranque es irrelevante.

## 7. Estrategia de Testing (para QA)

### Tests unitarios (Flutter)

- [ ] ViewModel: `login()` pasa `idle → loading → success` con el service mockeado.
- [ ] ViewModel: `login()` con el service lanzando error → `uiState=error` + `errorMessage` poblado.
- [ ] ViewModel: validación local de contraseña ≠ 4 dígitos **sin** llamar al service.
- [ ] Modelo: `AuthResponse.fromJson` con el payload real del backend.
- [ ] `GradosViewModel`: `cargar()` expone la lista mapeada y pasa `idle → loading → success`.
- [ ] `GradosViewModel`: error del service → `uiState=error` + `errorMessage`.

### Tests unitarios (Backend)

- [ ] `services/auth.py`: registrar con DNI libre → JWT; DNI duplicado → excepción 409; login correcto/incorrecto.
- [ ] `services/auth.py`: `id_grado` inexistente → `GradoInvalido` (422) y **no** inserta.
- [ ] `GET /api/grados`: responde 200 sin JWT y solo con grados `estado='AC'`.
- [ ] `repositories/estudiantes.py`: consultas con parámetros (contra BD de prueba o mock del cursor).

### Tests de integración

- [ ] Flujo completo: pantalla → ViewModel → Service → endpoint → JWT persistido y home accesible.

### Comandos

```bash
cd flutter_application_01
flutter analyze
flutter test

cd backend
uv run pytest
```

## 8. Consideraciones Futuras

- **Hash de contraseñas**: la columna `contrasena VARCHAR(4)` guarda en claro; migrar a hash implica ampliar la columna y acordar longitud mínima (deuda técnica).
- **Asociación docente/estudiante**: completar `id_docente` al importar la lista Excel del docente.
- **Expiración/refresh del JWT**: definir refresh token o re-login al expirar.
- **Rate limiting / bloqueo** por intentos fallidos de login.
- **Recuperación de contraseña** (canal del docente, dado que el estudiante no tiene email).
