# Especificación UI/UX: CRUD de Libros

> Basado en: `docs/specs/02-features/02-books.md` · Reglas globales en `docs/specs/04-ux-ui/design-system.md`

## 1. Paleta de Colores

| Color   | Valor            | Uso en esta feature |
| ------- | ---------------- | ------------------- |
| primary | `colorScheme.*`  | [TODO]              |
| error   | `colorScheme.*`  | validaciones        |

> Consumir con `Theme.of(context).colorScheme.*`, no valores fijos.

## 2. Tipografía

| Estilo      | Tamaño | Peso | Uso            |
| ----------- | ------ | ---- | -------------- |
| titleLarge  | [TODO] | ...  | título pantalla|
| bodyMedium  | ...    | ...  | [TODO]         |

## 3. Espaciado

| Token   | Valor  | Uso                 |
| ------- | ------ | ------------------- |
| [TODO]  | ...    | ...                 |

## 4. Pantallas

### 4.1 Lista de libros (`views/books/books_list_screen.dart`)

```
┌─────────────────────────────────────┐
│  [TODO: wireframe ASCII]            │
└─────────────────────────────────────┘
```

**Estados (pintados desde el `uiState` del ViewModel):**

- **Loading**: `CircularProgressIndicator` centrado / `Skeleton`
- **Error**: mensaje + botón "Reintentar"
- **Empty**: ilustración + texto guía ("No hay libros todavía")

**Interacciones:**

| Acción           | Resultado        |
| ---------------- | ---------------- |
| [TODO: tap, FAB]| [TODO]           |

---

### 4.2 Formulario de libro (`views/books/book_form_screen.dart`)

```
┌─────────────────────────────────────┐
│  [TODO: wireframe ASCII]            │
└─────────────────────────────────────┘
```

---

## 5. Navegación

```
'/books' ──acción──> '/books/new'      (alta)
'/books' ──tap────> '/books/{id}'      (edición)
```

- Ruta registrada en `lib/app/router.dart`.

## 6. Accesibilidad

- Contraste AA mínimo; targets táctiles >= 44-48 dp.
- [TODO: reglas específicas de la feature]

## 7. Responsive

| Ancho      | Comportamiento |
| ---------- | -------------- |
| < 600 px   | lista en una columna        |
| >= 600 px  | [TODO: 2 paneles / grid]    |

## 8. Animaciones / Transiciones

- [TODO]

## 9. Iconografía

| Acción   | Icono Material  |
| -------- | --------------- |
| agregar  | `Icons.add`     |
| editar   | `Icons.edit`    |
| borrar   | `Icons.delete`  |
