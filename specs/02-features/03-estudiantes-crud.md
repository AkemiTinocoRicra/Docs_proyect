# [FEAT-003] CRUD de Estudiantes

## Historia de Usuario

**Como** usuario autenticado en Booki,  
**Quiero** listar, crear, editar y desactivar estudiantes,  
**Para** gestionar la información de los estudiantes de la app.

---

## Criterios de Aceptación (Formato Gherkin)

- **Escenario 1: Lista de estudiantes**
  - **Dado que** el usuario inició sesión y tiene un JWT válido
  - **Cuando** navega a "Gestionar estudiantes"
  - **Entonces** el backend responde `200` con la lista de la tabla `estudiante` (sin contraseñas)
  - **Y** la app muestra cada estudiante con DNI, nombre, grado y chip de estado (Activo/Inactivo)

- **Escenario 2: Crear estudiante**
  - **Dado que** el usuario está en la lista de estudiantes y pulsa "+"
  - **Cuando** completa DNI, nombres, apellidos, fecha de nacimiento, grado y contraseña, y pulsa "Guardar"
  - **Entonces** el backend crea el registro con `estado = 'AC'` y `cantidad_copas = 0`
  - **Y** la app vuelve a la lista recargada

- **Escenario 3: Crear con DNI duplicado**
  - **Cuando** se intenta crear un estudiante con un DNI ya registrado
  - **Entonces** el backend responde `409` y la app muestra "El DNI ya está registrado"

- **Escenario 4: Editar estudiante**
  - **Dado que** el usuario toca un estudiante en la lista
  - **Cuando** modifica nombres, apellidos, fecha de nacimiento, grado, contraseña u estado y guarda
  - **Entonces** el backend aplica solo los campos enviados y responde `200`
  - **Y** el DNI permanece bloqueado (es el identificador único)

- **Escenario 5: Estudiante inexistente**
  - **Cuando** se pide editar o eliminar un id inexistente
  - **Entonces** el backend responde `404` "Estudiante no encontrado"

- **Escenario 6: Eliminar (lógico)**
  - **Dado que** el usuario confirma la eliminación en el diálogo
  - **Cuando** se envía `DELETE /api/estudiantes/{id}`
  - **Entonces** el backend pone `estado = 'IN'` (el estudiante no podrá iniciar sesión; sus datos se conservan)
  - **Y** la app muestra al estudiante como Inactivo

- **Escenario 7: Reactivar**
  - **Cuando** en la edición de un estudiante inactivo se cambia el estado a Activo
  - **Entonces** el backend pone `estado = 'AC'` y vuelve a poder iniciar sesión

- **Escenario 8: Sin JWT válido**
  - **Cuando** se llama a cualquier endpoint del CRUD sin token, con token inválido o expirado
  - **Entonces** el backend responde `401`

- **Escenario 9: Validación local**
  - **Cuando** se escribe un DNI ≠ 8 dígitos o una contraseña ≠ 4 dígitos en el formulario
  - **Entonces** la app muestra el mensaje de validación y **no** envía la petición

---

## Reglas de Negocio

- Todos los endpoints del CRUD exigen **JWT** (`Authorization: Bearer …` verificado con `JWT_SECRET_KEY`).
- **DNI**: 8 dígitos, único (`UNIQUE`); no editable en `PUT`.
- **Contraseña**: 4 dígitos numéricos; vacía al editar = no modificar; se guarda en claro (deuda técnica ya documentada en FEAT-001).
- **Borrado lógico**: `DELETE` pone `estado = 'IN'`; reactivación vía `PUT` con `estado = 'AC'`.
- Campos que rellena el backend al crear: `cantidad_copas = 0` y `estado = 'AC'`.
- **Grado**: el formulario (crear y editar) obtiene el catálogo de la tabla `grado` con `GET /api/grados` (público, solo activos); **no** hardcodea 1°–6°. El backend valida que `id_grado` exista (`422` "Grado no válido").
- `EstudianteOut` **nunca** incluye `contrasena`.
- `PUT` solo actualiza los campos enviados y no nulos (whitelist: nombres, apellidos, fecha_nacimiento, contrasena, id_grado, estado).

## Estados de Pantalla

> ViewModel (`uiState`): `idle` / `loading` / `success` / `error` + `errorMessage`.

| Pantalla | Acciones |
| -------- | -------- |
| Lista (`/estudiantes`) | `load()` al entrar y al volver del formulario; `eliminar(id)` con confirmación |
| Formulario (`/estudiantes/form`) | `cargar(id)` al editar; `cargar()` del catálogo de grados al abrir; `guardar(...)` con validación local previa |

## Navegación (rutas Flutter)

```
'/home' ──"Gestionar estudiantes"──> '/estudiantes'
'/estudiantes' ──FAB "+"────────────> '/estudiantes/form'        (crear)
'/estudiantes' ──tap fila───────────> '/estudiantes/form' (id)   (editar)
'/estudiantes/form' ──exito─────────> '/estudiantes'             (lista recargada)
```

## Endpoints

| Método | Ruta                       | Body           | Respuesta    | Errores           |
| ------ | -------------------------- | -------------- | ------------ | ----------------- |
| `GET`  | `/api/estudiantes`         | - (query `?estado=AC\|IN`) | `200` lista | 401, 500 |
| `GET`  | `/api/estudiantes/{id}`    | -              | `200` objeto | 401, 404, 500 |
| `POST` | `/api/estudiantes`         | `RegistroIn`   | `201` objeto | 401, 409, 422 |
| `PUT`  | `/api/estudiantes/{id}`    | `EstudianteUpdateIn` | `200` objeto | 401, 404, 422 |
| `DELETE` | `/api/estudiantes/{id}`  | -              | `204`        | 401, 404, 500 |

## Capas implementadas

```
backend/src/backend/
├── apis/deps.py               # requiere_auth (verificación JWT)
├── apis/estudiantes.py        # Routers CRUD
├── services/estudiantes.py    # Lógica (NoEncontrado, reutiliza DniYaRegistrado)
├── repositories/estudiantes.py# SQL parametrizado (find_all/find_by_id/update/set_estado)
└── schemas/estudiantes.py     # EstudianteOut, EstudianteUpdateIn

flutter_application_01/lib/
├── models/estudiante.dart
├── services/estudiante_service.dart      # con header Authorization
├── viewmodels/estudiantes_viewmodel.dart # lista
├── viewmodels/estudiante_form_viewmodel.dart
└── views/estudiantes/estudiantes_list_screen.dart
    └── estudiante_form_screen.dart
```

## Specs relacionadas

- **Feature base (auth)**: `docs/specs/02-features/01-auth.md`
- **Técnica (auth)**: `docs/specs/03-technicals/01-auth-tech.md`
- **Patrón MVVM**: `docs/specs/03-technicals/flutter-mvvm.md`
- **Estructura backend**: `docs/specs/03-technicals/backend-structure.md`
