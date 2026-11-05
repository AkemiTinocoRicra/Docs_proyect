# Documentación del Proyecto (docs/)

Este directorio concentra **todas las especificaciones** del proyecto **Booki**. No es un
proyecto ejecutable: es la fuente de verdad de *qué* se construye, *cómo* y *cómo se verifica*.

- **Requisitos**: historias de usuario (Como/Quiero/Para) con criterios Gherkin.
- **Formato**: Markdown, en español, un archivo por funcionalidad.
- **Código**: vive fuera de aquí, en [`../backend/`](../backend/) y
  [`../flutter_application_01/`](../flutter_application_01/).

> Los archivos `TEMPLATE-*.md` **nunca se editan**: se copian para crear una feature nueva.
> Los documentos marcados como *verificado el [TODO: AAAA-MM-DD]* describen el estado real del
> scaffold; si el código cambia, hay que actualizarlos.

---

## Estructura

```
docs/
└── specs/
    ├── 01-product/
    │   └── product-backlog.md          # Visión, roadmap, estado y deuda técnica
    ├── 02-features/                    # QUÉ construir (requisitos)
    │   ├── TEMPLATE-FEATURE.md         #   plantilla: HU + Gherkin + estados + endpoints
    │   └── NN-<slug>.md                #   una por funcionalidad
    ├── 03-technicals/                  # CÓMO construirlo
    │   ├── architecture-blueprint.md   #   reglas globales: capas, MVVM, .env, comandos
    │   ├── TEMPLATE-TECH.md            #   plantilla: esquema SQL, API, lógica, tests
    │   └── NN-<slug>-tech.md           #   una por funcionalidad
    ├── 04-ux-ui/                       # CÓMO SE VE
    │   ├── design-system.md            #   reglas globales: colores, tipografía, layout
    │   ├── TEMPLATE-UI.md              #   plantilla: wireframes, estados, navegación
    │   └── NN-<slug>-ui.md             #   una por funcionalidad
    └── 05-testing/                     # CÓMO SE VERIFICA
        ├── qa-strategy.md              #   umbrales, comandos y Definition of Done
        ├── TEMPLATE-QA-CHECKLIST.md    #   plantilla: checklist manual pre-merge
        └── <slug>-qa-checklist.md      #   uno por funcionalidad
```

### Documentos globales (leer una vez)

| Documento | Contenido |
| --------- | --------- |
| [`01-product/product-backlog.md`](specs/01-product/product-backlog.md) | Visión del producto, decisiones confirmadas, funcionalidades implementadas/especificadas, roadmap y deuda técnica. **Punto de entrada**: actualizarlo siempre que se toque algo. |
| [`03-technicals/architecture-blueprint.md`](specs/03-technicals/architecture-blueprint.md) | Arquitectura objetivo por capas (FastAPI `apis → services → repositories` y Flutter MVVM), archivos `.env`, comandos y restricciones del diseño. |
| [`04-ux-ui/design-system.md`](specs/04-ux-ui/design-system.md) | Paleta, tipografía, espaciado, responsive, accesibilidad y convenciones de código Dart. |
| [`05-testing/qa-strategy.md`](specs/05-testing/qa-strategy.md) | Stack de testing, umbrales de calidad, checklist pre-merge y matriz de pruebas por capa. |

---

## Flujo de trabajo: de la idea a la funcionalidad

Cada funcionalidad se especifica en **4 documentos** que se crean a partir de las
plantillas. El orden importa: primero el *qué*, después el *cómo*.

1. **Copiar las 4 plantillas** con el mismo número y slug:

   ```bash
   cd docs/specs
   cp 02-features/TEMPLATE-FEATURE.md      02-features/01-<slug>.md
   cp 03-technicals/TEMPLATE-TECH.md       03-technicals/01-<slug>-tech.md
   cp 04-ux-ui/TEMPLATE-UI.md              04-ux-ui/01-<slug>-ui.md
   cp 05-testing/TEMPLATE-QA-CHECKLIST.md  05-testing/<slug>-qa-checklist.md
   ```

2. **Completar la historia de usuario** en `02-features/01-<slug>.md`: HU, criterios
   Gherkin, reglas de negocio, estados del ViewModel y endpoints.
3. **Especificar lo técnico** en `03-technicals/01-<slug>-tech.md`: esquema SQL de MySQL
   (se aplica **a mano**, no hay ORM ni migraciones), contrato de la API,
   ViewModel/Services y estrategia de tests.
4. **Diseñar la UI** en `04-ux-ui/01-<slug>-ui.md`: wireframes, estados
   (loading / empty / error), navegación, accesibilidad y responsive.
5. **Registrar la feature** en `01-product/product-backlog.md` (sección
   *Funcionalidades Especificadas*, formato en el comentario del archivo).
6. **Implementar** en `backend/` y `flutter_application_01/` siguiendo el blueprint.
7. **Ejecutar el checklist QA** `05-testing/<slug>-qa-checklist.md` completo; cualquier
   FAIL bloquea. Repetir hasta PASS.
8. **Actualizar el backlog**: mover la feature a *Funcionalidades Implementadas*.

---

## Decisiones de arquitectura (resumen)

Detalle completo en [`03-technicals/architecture-blueprint.md`](specs/03-technicals/architecture-blueprint.md).

| Área | Decisión |
| ---- | -------- |
| Requisitos | Historias de usuario + criterios Gherkin en `02-features/` |
| App | **Flutter** con patrón **MVVM** (View ← ViewModel ← Model/Service) |
| Backend | **FastAPI**: `main.py` → `apis/` → `services/` → `repositories/` (+ `schemas/`) |
| Persistencia | **Sin ORM**: SQL a mano con PyMySQL sobre **MySQL** (`DB_NAME=[TODO]`) |
| BD ↔ Backend | `backend/.env` con `pydantic-settings` (`DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`) |
| Backend ↔ Flutter | `flutter_application_01/.env` con `flutter_dotenv` (`API_BASE_URL`) |
| Comunicación | HTTP/JSON: los `schemas/` (Pydantic) y los `models/` (Dart) son espejo entre sí |

Reglas que no se negocian:

- Solo `repositories/` toca la base de datos, siempre con **SQL parametrizado** (`%s`).
- La View de Flutter **nunca** hace HTTP ni parsea JSON; eso vive en Service/ViewModel.
- **Nada hardcodeado**: configuración por `.env` (`.env` no se commitea, `.env.example` sí).
- El `.env` de Flutter va empaquetado en el APK: **nunca secretos** ahí.

---

## Comandos rápidos

```bash
# Backend (desde backend/)
uv sync                                    # dependencias
uv run uvicorn backend.main:app --reload   # levantar API
uv run pytest                              # tests (tras añadir pytest)

# Flutter (desde flutter_application_01/)
flutter pub get                            # dependencias
flutter run                                # ejecutar
flutter analyze                            # estático → debe dar 0 issues
flutter test                               # tests (requiere crear test/)
dart format lib/                           # formato

# Docs: sin build ni herramientas; solo Markdown en git.
```

---

## Estado actual

- **Fase**: [TODO: especificación / implementación].
- **Funcionalidades especificadas**: [TODO: N] · **implementadas**: [TODO: N].
- `backend/`: [TODO: estado real del scaffold].
- `flutter_application_01/`: [TODO: estado real de la app].
- Comunicación backend ↔ Flutter: [TODO: existe / no existe].

El estado detallado, los pendientes y la deuda técnica se mantienen en
[`01-product/product-backlog.md`](specs/01-product/product-backlog.md) — **consultarlo
antes de empezar a trabajar**.
