# [FEAT-002] CRUD de Libros

## Historia de Usuario

**Como** [TODO: tipo de usuario o rol]  
**Quiero** [TODO: realizar una acción sobre libros]  
**Para** [TODO: obtener un valor de negocio o beneficio claro]

---

## Criterios de Aceptación (Formato Gherkin)

_Define cuándo la funcionalidad se considera lista para producción (Definition of Done)._

- **Escenario 1: Flujo Exitoso Principal**
  - **Dado que** [TODO: estado inicial de la app]
  - **Cuando** [TODO: el usuario realiza la acción clave]
  - **Entonces** [TODO: el sistema reacciona de esta manera]

- **Escenario 2: Flujo Alternativo o de Error**
  - **Dado que** [TODO]
  - **Cuando** [TODO]
  - **Entonces** [TODO]

---

## Reglas de Negocio

- [TODO: reglas que el código debe respetar, con valores exactos]

## Estados de Pantalla

> Estado expuesto por el **ViewModel** (MVVM). La View solo pinta estos estados.

| Estado    | Descripción            |
| --------- | ---------------------- |
| `loading` | [TODO]                 |
| `loaded`  | [TODO]                 |
| `error`   | [TODO]                 |

## ViewModel

- **Archivo**: `flutter_application_01/lib/viewmodels/book_viewmodel.dart`
- **Estado expuesto**: `UiState`, `items`, `errorMessage`, [TODO]
- **Acciones**: `load()`, `save()`, `remove()`, [TODO]

## Navegación (rutas Flutter)

```
'/ruta-origen' ──acción──> '/ruta-destino'
```

- Ruta registrada en `lib/app/router.dart`.

## Endpoints

| Método | Ruta                        | Capa que lo implementa                                  |
| ------ | --------------------------- | ------------------------------------------------------- |
| `GET`  | `/api/books`                | `apis/books.py` → `services/` → `repositories/`         |
| `GET`  | `/api/books/{id}`           | `apis/books.py` → `services/` → `repositories/`         |
| `POST` | `/api/books`                | `apis/books.py` → `services/` → `repositories/`         |
| `PUT`  | `/api/books/{id}`           | `apis/books.py` → `services/` → `repositories/`         |
| `DELETE` | `/api/books/{id}`         | `apis/books.py` → `services/` → `repositories/`         |

## Specs relacionadas

- **Técnica**: `docs/specs/03-technicals/02-books-tech.md`
- **UI/UX**: `docs/specs/04-ux-ui/02-books-ui.md`
- **QA**: `docs/specs/05-testing/books-qa-checklist.md`
