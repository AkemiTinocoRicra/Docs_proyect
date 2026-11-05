# Especificación UI/UX: Autenticación (Crear cuenta e iniciar sesión)

> Basado en: `docs/specs/02-features/01-auth.md` · Reglas globales en `docs/specs/04-ux-ui/design-system.md`

## 1. Paleta de Colores

| Color   | Valor                   | Uso en esta feature                     |
| ------- | ----------------------- | --------------------------------------- |
| primary | `colorScheme.primary`   | botones "Crear cuenta" / "Iniciar sesión" |
| error   | `colorScheme.error`     | validaciones inline y errores de API    |
| surface | `colorScheme.surface`   | fondo de las pantallas                  |

> Consumir con `Theme.of(context).colorScheme.*`, no valores fijos.

## 2. Tipografía

| Estilo      | Tamaño | Peso | Uso                          |
| ----------- | ------ | ---- | ---------------------------- |
| titleLarge  | 24     | 600  | Título de la pantalla        |
| bodyMedium  | 14     | 400  | Labels y mensajes de error   |
| bodySmall   | 12     | 400  | Ayuda / contador "4 dígitos" |

## 3. Espaciado

| Token | Valor | Uso                                 |
| ----- | ----- | ----------------------------------- |
| s1    | 8 dp  | Entre label y su campo              |
| s2    | 16 dp | Entre campos del formulario         |
| s3    | 24 dp | Padding horizontal de la pantalla   |
| s4    | 32 dp | Entre formulario y link secundario  |

## 4. Pantallas

### 4.1 Login (`views/auth/login_screen.dart`)

```
┌─────────────────────────────────────┐
│            📖 Booki                 │
│         Iniciar sesión              │
│                                     │
│  DNI                                │
│  ┌─────────────────────────────┐    │
│  │ 8 dígitos                   │    │
│  └─────────────────────────────┘    │
│                                     │
│  Contraseña                         │
│  ┌─────────────────────────────┐    │
│  │ ••••                        │    │
│  └─────────────────────────────┘    │
│  (4 dígitos numéricos)              │
│                                     │
│  ┌─────────────────────────────┐    │
│  │      Iniciar sesión         │    │
│  └─────────────────────────────┘    │
│                                     │
│  ¿No tienes cuenta?  Crear cuenta   │
└─────────────────────────────────────┘
```

**Estados (pintados desde el `uiState` del ViewModel):**

- **Loading**: `CircularProgressIndicator` sobre el botón; campos deshabilitados.
- **Error**: mensaje bajo el botón (`errorMessage`) en `colorScheme.error`.
- **Success**: navegación a `'/home'` y persistencia del JWT.

**Validaciones locales (sin llamada al backend):**

| Campo       | Regla                          | Mensaje                                  |
| ----------- | ------------------------------ | ---------------------------------------- |
| DNI         | exactamente 8 dígitos          | "El DNI debe tener 8 dígitos"            |
| Contraseña  | exactamente 4 dígitos          | "La contraseña debe tener 4 dígitos"     |

**Interacciones:**

| Acción                                 | Resultado                       |
| -------------------------------------- | ------------------------------- |
| Tap "Iniciar sesión"                   | `AuthViewModel.login(...)`      |
| Tap "Crear cuenta"                     | Navega a `'/registro'`          |
| Texto en DNI/Contraseña                | `keyboardType: TextInputType.number` |
| Campo contraseña                       | `obscureText: true` + icono ojo |

---

### 4.2 Registro (`views/auth/registro_screen.dart`)

```
┌─────────────────────────────────────┐
│  ← Volver        Crear cuenta       │
│                                     │
│  DNI *                              │
│  ┌─────────────────────────────┐    │
│  │ 8 dígitos                   │    │
│  └─────────────────────────────┘    │
│                                     │
│  Nombres *        Apellidos *       │
│  ┌──────────────┐ ┌──────────────┐  │
│  │              │ │              │  │
│  └──────────────┘ └──────────────┘  │
│                                     │
│  Fecha de nacimiento                 │
│  ┌─────────────────────────────┐    │
│  │ dd/mm/aaaa  (📅)            │    │
│  └─────────────────────────────┘    │
│                                     │
│  Grado *                            │
│  ┌─────────────────────────────┐    │
│  │ Selecciona tu grado     ▾   │    │
│  └─────────────────────────────┘    │
│                                     │
│  Contraseña *                       │
│  ┌─────────────────────────────┐    │
│  │ ••••                        │    │
│  └─────────────────────────────┘    │
│                                     │
│  Confirmar contraseña *             │
│  ┌─────────────────────────────┐    │
│  │ ••••                        │    │
│  └─────────────────────────────┘    │
│                                     │
│  ┌─────────────────────────────┐    │
│  │       Crear cuenta          │    │
│  └─────────────────────────────┘    │
│                                     │
│  ¿Ya tienes cuenta?  Iniciar sesión │
└─────────────────────────────────────┘
```

**Estados (pintados desde el `uiState` del ViewModel):**

- **Loading**: `CircularProgressIndicator` sobre el botón; campos deshabilitados.
- **Error**: mensaje bajo el botón (incluye `409` → "El DNI ya está registrado").
- **Success**: navegación a `'/home'` con el JWT persistido.

**Validaciones locales:**

| Campo                | Regla                        | Mensaje                                    |
| -------------------- | ---------------------------- | ------------------------------------------ |
| DNI                  | exactamente 8 dígitos        | "El DNI debe tener 8 dígitos"              |
| Nombres / Apellidos  | no vacíos                    | "Este campo es obligatorio"                |
| Contraseña           | exactamente 4 dígitos        | "La contraseña debe tener 4 dígitos"       |
| Confirmar contraseña | == contraseña                | "Las contraseñas no coinciden"            |
| Grado                | seleccionado del catálogo    | "Selecciona tu grado"                      |

**Interacciones:**

| Acción                                   | Resultado                              |
| ---------------------------------------- | -------------------------------------- |
| Tap "Crear cuenta"                       | `AuthViewModel.register(...)`          |
| Tap "Iniciar sesión"                     | Navega a `'/login'`                    |
| Tap en "Fecha de nacimiento"             | Selector de fecha (`showDatePicker`)   |
| Tap en "Grado"                           | Dropdown poblado desde `GET /api/grados` (solo activos) |
| Texto en DNI / contraseñas              | `keyboardType: TextInputType.number`   |
| Campos de contraseña                     | `obscureText: true`                    |

> **El dropdown no está hardcodeado.** Se puebla con el catálogo de la tabla `grado` vía `GET /api/grados` (solo `estado = 'AC'`, ordenado por `id_grado`); cada opción muestra `nivel`/`nombres` y usa `id_grado` como valor.
>
> Estados del catálogo:
>
> - **Loading**: dropdown deshabilitado con "Cargando grados...".
> - **Error**: mensaje en `colorScheme.error` + botón "Reintentar".
> - **Vacío**: "No hay grados disponibles" y el botón "Crear cuenta" deshabilitado.

---

## 5. Navegación

```
'/login' ──"Crear cuenta"──> '/registro'
'/registro' ──exito / back──> '/login'
'/login' ──exito────────────> '/home'
'/registro' ──exito─────────> '/home'
```

- Rutas registradas en `lib/app/router.dart`.
- Arranque de la app: `restoreSession()` → JWT válido a `'/home'`; sin token a `'/login'`.

## 6. Accesibilidad

- Contraste AA mínimo; targets táctiles >= 44-48 dp.
- Labels visibles siempre (no solo `hintText`); error anunciado con `Semantics(liveRegion: true)`.
- Tamaño de fuente del sistema hasta 200% sin cortar contenido.

## 7. Responsive

| Ancho      | Comportamiento                                          |
| ---------- | ------------------------------------------------------- |
| < 600 px   | Una columna (teléfono, por defecto)                      |
| >= 600 px  | Formulario centrado con ancho máximo ~480 dp (tablet)    |

## 8. Animaciones / Transiciones

- Transición estándar de rutas (`MaterialPageRoute`).
- Sin animaciones decorativas en el formulario (contexto de bajo rendimiento / zonas rurales).

## 9. Iconografía

| Acción              | Icono Material                  |
| ------------------- | ------------------------------- |
| Volver (registro)   | `Icons.arrow_back`              |
| Ver contraseña      | `Icons.visibility` / `_off`     |
| Seleccionar fecha   | `Icons.calendar_today`          |
| Seleccionar grado   | `Icons.arrow_drop_down`         |
| Logo Booki          | `Icons.menu_book`               |
