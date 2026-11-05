# [FEAT-00N] Nombre de la Funcionalidad

## Historia de Usuario

**Como** [tipo de usuario o rol]  
**Quiero** [realizar una acción o interactuar con un elemento]  
**Para** [obtener un valor de negocio o beneficio claro]

---

## Criterios de Aceptación (Formato Gherkin)

_Define cuándo la funcionalidad se considera lista para producción (Definition of Done)._

- **Escenario 1: Flujo Exitoso Principal**
  - **Dado que** [estado inicial de la app]
  - **Cuando** [el usuario realiza la acción clave]
  - **Entonces** [el sistema reacciona de esta manera]

- **Escenario 2: Flujo Alternativo o de Error**
  - **Dado que** ...
  - **Cuando** ...
  - **Entonces** ...

---

## Reglas de Negocio

- [Reglas que el código debe respetar, con valores exactos.]

## Estados de Pantalla

> Estado expuesto por el **ViewModel** (MVVM). La View solo pinta estos estados.

| Estado    | Descripción |
| --------- | ----------- |
| `loading` | ...         |
| `loaded`  | ...         |
| `error`   | ...         |

## ViewModel

- **Archivo**: `flutter_application_01/lib/viewmodels/<recurso>_viewmodel.dart`
- **Estado expuesto**: `UiState`, `items`, `errorMessage`, ...
- **Acciones**: `load()`, `save()`, `remove()`, ...

## Navegación (rutas Flutter)

```
'/ruta-origen' ──acción──> '/ruta-destino'
```

- Ruta registrada en `lib/app/router.dart`.

## Endpoints

| Método | Ruta                  | Capa que lo implementa                                         |
| ------ | --------------------- | -------------------------------------------------------------- |
| `GET`  | `/api/<recurso>`      | `apis/<recurso>.py` → `services/` → `repositories/`            |

## Specs relacionadas

- **Técnica**: `docs/specs/03-technicals/NN-slug-tech.md`
- **UI/UX**: `docs/specs/04-ux-ui/NN-slug-ui.md`
- **QA**: `docs/specs/05-testing/slug-qa-checklist.md`
