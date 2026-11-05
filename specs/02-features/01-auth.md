# [FEAT-001] Autenticación: Crear cuenta e iniciar sesión

## Historia de Usuario

**Como** estudiante,  
**Quiero** crear una cuenta con mi DNI y una contraseña de 4 dígitos, e iniciar sesión con esos datos,  
**Para** acceder a Booki, mis libros descargados y mis copas de lectura.

---

## Criterios de Aceptación (Formato Gherkin)

_Define cuándo la funcionalidad se considera lista para producción (Definition of Done)._

- **Escenario 1: Registro exitoso**
  - **Dado que** el estudiante no tiene cuenta y está en la pantalla de registro
  - **Cuando** completa DNI (8 dígitos), nombres, apellidos, fecha de nacimiento, grado, contraseña (4 dígitos) y confirmación, y pulsa "Crear cuenta"
   - **Entonces** el backend crea el registro en la tabla `estudiante` con `estado = 'AC'` y `cantidad_copas = 0`
  - **Y** la app recibe un JWT, lo persiste localmente y navega al home logueado

- **Escenario 2: Registro con DNI ya existente**
  - **Dado que** existe un estudiante con el DNI `12345678`
  - **Cuando** otro usuario intenta registrarse con el mismo DNI
  - **Entonces** el backend responde `409` y la app muestra "El DNI ya está registrado"

- **Escenario 3: Contraseña inválida (validación local)**
  - **Dado que** el estudiante está en la pantalla de registro o de login
  - **Cuando** escribe una contraseña que no tiene exactamente 4 dígitos numéricos
  - **Entonces** la app muestra un mensaje de validación y **no** envía la petición al backend

- **Escenario 4: Login exitoso**
  - **Dado que** el estudiante tiene una cuenta creada
  - **Cuando** ingresa su DNI y su contraseña de 4 dígitos y pulsa "Iniciar sesión"
  - **Entonces** el backend valida las credenciales y devuelve un JWT
  - **Y** la app persiste el token y navega al home

- **Escenario 5: Credenciales incorrectas**
  - **Dado que** el estudiante está en la pantalla de login
  - **Cuando** ingresa un DNI inexistente o una contraseña incorrecta
  - **Entonces** el backend responde `401` y la app muestra "DNI o contraseña incorrectos"

- **Escenario 6: Sesión persistente**
  - **Dado que** el estudiante inició sesión y la app guardó el JWT localmente
  - **Cuando** cierra y vuelve a abrir la aplicación
  - **Entonces** la app lee el token guardado, lo valida y muestra el home sin pedir credenciales de nuevo

---

## Reglas de Negocio

- **DNI**: exactamente **8 dígitos numéricos**; único en la tabla `estudiante` (`UNIQUE`).
- **Contraseña**: exactamente **4 dígitos numéricos** (solo números); validada en la app **antes** de llamar al backend.
- **Contraseña en claro**: se guarda tal cual en `estudiante.contrasena VARCHAR(4)` (esquema existente). El hash se considera deuda técnica / mejora futura.
- **Registro**: pide **todos los datos del estudiante**: DNI, nombres, apellidos, fecha de nacimiento, grado y contraseña.
- **Campos que rellena el backend** (no los pide el formulario): `cantidad_copas = 0` y `estado = 'AC'`.
- **Grado**: el estudiante lo elige al registrarse (`id_grado`); **el catálogo de grados viene de la base de datos** (tabla `grado`) y la app lo obtiene con `GET /api/grados` — **nunca** está hardcodeado en el cliente.
- **Catálogo de grados**: `GET /api/grados` es **público** (no requiere JWT, porque el registro ocurre antes de tener token); devuelve solo los grados con `estado = 'AC'`, ordenados por `id_grado`.
- **Validación de `id_grado`**: el backend verifica que el grado exista en la tabla `grado`; si no existe responde `422` ("Grado no válido") y no crea la cuenta.
- **Login**: solo envía **DNI + contraseña**.
- **Sesión**: el login devuelve un **JWT** con expiración (`JWT_EXPIRE_MIN`); la app lo persiste en `shared_preferences` y lo reutiliza al arrancar.
- **Seguridad**: el JWT y las credenciales no se loguean; el `.env` de Flutter no lleva secretos.

## Estados de Pantalla

> Estado expuesto por el **ViewModel** (MVVM). La View solo pinta estos estados.

| Estado    | Descripción                                              |
| --------- | -------------------------------------------------------- |
| `idle`    | Formulario listo, sin operación en curso                  |
| `loading` | Petición de registro/login en curso (botón deshabilitado) |
| `success` | Operación exitosa; se navega al home y se guarda el JWT   |
| `error`   | Mensaje mostrable en `errorMessage` (409, 401, red, etc.) |

Además: `isAuthenticated` (bool) para decidir la ruta inicial (login vs home).

## ViewModel

- **Archivo**: `flutter_application_01/lib/viewmodels/auth_viewmodel.dart`
- **Estado expuesto**: `uiState`, `errorMessage`, `isAuthenticated`
- **Acciones**: `register(...)`, `login(dni, contrasena)`, `restoreSession()`, `logout()`

## Navegación (rutas Flutter)

```
'/login' ──¿cuenta?──> '/registro'     (link "Crear cuenta")
'/registro' ──exito──> '/home'         (tras registrar)
'/login' ──exito────> '/home'          (tras iniciar sesión)
'/home' ──logout────> '/login'         (cerrar sesión)
Arranque: con JWT válido → '/home'; sin JWT → '/login'
```

- Rutas registradas en `lib/app/router.dart`.

## Endpoints

| Método | Ruta                  | Capa que lo implementa                                          |
| ------ | --------------------- | --------------------------------------------------------------- |
| `POST` | `/api/auth/registro`  | `apis/auth.py` → `services/auth.py` → `repositories/estudiantes.py` |
| `POST` | `/api/auth/login`     | `apis/auth.py` → `services/auth.py` → `repositories/estudiantes.py` |
| `GET`  | `/api/grados`         | `apis/grados.py` → `services/grados.py` → `repositories/grados.py` (**público**, sin JWT) |

## Specs relacionadas

- **Técnica**: `docs/specs/03-technicals/01-auth-tech.md`
- **UI/UX**: `docs/specs/04-ux-ui/01-auth-ui.md`
- **QA**: `docs/specs/05-testing/auth-qa-checklist.md`
