# Product Backlog General

## Visión del Producto

> **[TODO — completar por el equipo]**
> Booki es una aplicación móvil educativa de lectura, pensada para estudiantes de instituciones educativas de zonas rurales con poca o nula conectividad a internet. El estudiante lee libros descargados en su celular (hasta 10, sin internet), gana copas al leer y al completar cuestionarios, y usa esas copas para jugar. El docente registra a sus estudiantes importando una lista en Excel (también sin internet) y consulta su lista por grado. Problema que resuelve: la falta de lecturas adecuadas a cada edad en zonas rurales y la poca motivación lectora, aun sin conexión a internet.

## Decisiones de Arquitectura (confirmadas)

| Área | Decisión |
| ---- | -------- |
| Requisitos | **Historias de usuario** (Como/Quiero/Para) + criterios Gherkin en `02-features/` |
| App | **Flutter** con patrón **MVVM** |
| Backend | FastAPI: `main.py` → `apis/` → `services/` → `repositories/` (+ `schemas/`) |
| Persistencia | **Sin ORM** (SQL a mano con PyMySQL) sobre **MySQL** (`DB_NAME=[TODO]`) |
| BD ↔ Backend | `backend/.env` con `pydantic-settings` (`DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`) |
| Backend ↔ Flutter | `flutter_application_01/.env` con `flutter_dotenv` (`API_BASE_URL`) |

> Detalle completo en `docs/specs/03-technicals/architecture-blueprint.md`.

## Estado Actual del Repositorio

> **Verificado el 2026-10-26** contra el código.

- `backend/`: FastAPI funcional en `src/backend/main.py` (CORS + routers `auth` y `estudiantes`);
  capas `apis/ services/ repositories/ schemas/` + `config.py` (pydantic-settings) + `db.py` (PyMySQL);
  tests con pytest (28 pasando).
- `flutter_application_01/`: app MVVM con `app/ core/ models/ services/ viewmodels/ views/`;
  login/registro + CRUD de estudiantes; `.env` con `flutter_dotenv`; `flutter analyze` limpio;
  tests (15 pasando).
- **Comunicación**: existente (Flutter → FastAPI → MySQL `Booki`).
- **2 funcionalidades implementadas** · **1 especificada sin implementar** (FEAT-002 Books).

## Funcionalidades Implementadas

### ✅ **[FEAT-001]** Autenticación: Crear cuenta e iniciar sesión

- **Historia de Usuario**: `docs/specs/02-features/01-auth.md`
- **Técnica**: `docs/specs/03-technicals/01-auth-tech.md`
- **UI/UX**: `docs/specs/04-ux-ui/01-auth-ui.md`
- **QA**: `docs/specs/05-testing/auth-qa-checklist.md`
- **Estado**: Implementada y funcional (tests backend y Flutter en verde)
- **Ruta / Endpoint**: `POST /api/auth/registro`, `POST /api/auth/login`, `GET /api/grados` (público)
- **Capa de datos**: tablas `estudiante` y `grado` de MySQL `DB_NAME=Booki` (ya existentes; sin DDL).
  El grado se elige desde el catálogo de la BD, nunca hardcodeado en el cliente.

### ✅ **[FEAT-003]** CRUD de Estudiantes

- **Historia de Usuario**: `docs/specs/02-features/03-estudiantes-crud.md` (HU + reglas + endpoints + capas en un solo documento)
- **Estado**: Implementada y funcional (protegida con JWT; tests backend y Flutter en verde)
- **Ruta / Endpoint**: `GET/POST /api/estudiantes`, `GET/PUT/DELETE /api/estudiantes/{id}`
- **Capa de datos**: tabla `estudiante` (borrado lógico con `estado` 'AC'/'IN')

<!--
Formato para nuevas entradas:
### ✅ **[FEAT-NNN]** Nombre de la Funcionalidad
- **Historia de Usuario**: `docs/specs/02-features/NN-slug.md`
- **Técnica**: `docs/specs/03-technicals/NN-slug-tech.md`
- **UI/UX**: `docs/specs/04-ux-ui/NN-slug-ui.md`
- **QA**: `docs/specs/05-testing/slug-qa-checklist.md`
- **Estado**: Implementada y funcional
- **Ruta / Endpoint**: `/ruta` o `POST /api/ruta`
-->

## Funcionalidades Especificadas (Pendientes de Implementación)

### 📋 **[FEAT-002]** CRUD de Libros

- **Historia de Usuario**: `docs/specs/02-features/02-books.md`
- **Técnica**: `docs/specs/03-technicals/02-books-tech.md`
- **UI/UX**: `docs/specs/04-ux-ui/02-books-ui.md`
- **QA**: `docs/specs/05-testing/books-qa-checklist.md`
- **Estado**: Especificada (esqueleto `[TODO]`, pendiente de completar e implementar en
  `backend/` y `flutter_application_01/`)
- **Ruta / Endpoint**: `/api/books` (`GET`/`POST`), `/api/books/{id}` (`GET`/`PUT`/`DELETE`)
- **Capa de datos**: tabla `Book` de MySQL `DB_NAME=[TODO]` — [TODO: columnas, PK, UNIQUE,
  filas de prueba]. **DDL a mano, sin ORM ni migraciones.**

<!--
Para crear la siguiente feature (número 04):

1. Copiar `02-features/TEMPLATE-FEATURE.md` → `02-features/04-<slug>.md` (HU + Gherkin).
2. Copiar `03-technicals/TEMPLATE-TECH.md` → `03-technicals/04-<slug>-tech.md`.
3. Copiar `04-ux-ui/TEMPLATE-UI.md` → `04-ux-ui/04-<slug>-ui.md`.
4. Copiar `05-testing/TEMPLATE-QA-CHECKLIST.md` → `05-testing/<slug>-qa-checklist.md`.
5. Registrar la feature aquí (mismo formato que arriba) y moverla a "Implementadas"
   cuando pase el checklist QA.
-->

## Próximas Funcionalidades (Roadmap)

- [x] **[FEAT-001]** Autenticación (crear cuenta e iniciar sesión) — implementada.
- [x] **[FEAT-003]** CRUD de Estudiantes — implementada.
- [ ] **[FEAT-002]** CRUD de Libros — esqueleto de specs creado, pendiente de completar e implementar.
- [ ] **[FEAT-003] specs de soporte**: añadir `03-estudiantes-crud-tech.md`, `03-estudiantes-crud-ui.md`
      y `estudiantes-crud-qa-checklist.md` (hoy la HU, reglas y endpoints viven en un solo archivo).
- [ ] [TODO: siguientes features del roadmap]

## Objetivos del Sprint Actual

- [x] Definir la arquitectura objetivo (MVVM + capas backend + `.env`) →
      `03-technicals/architecture-blueprint.md`.
- [x] Crear la estructura de specs (plantillas en `02/03/04/05`).
- [x] Configurar `backend/.env` + `backend/.env.example` (`pydantic-settings`, `DB_*`).
      Añadir `.env` a `backend/.gitignore`.
- [x] Configurar `flutter_application_01/.env` + `.env.example` (`API_BASE_URL`) con
      `flutter_dotenv` (asset `.env` en `pubspec.yaml`) e ignorar `.env` en `.gitignore`.
- [x] Crear `src/backend/main.py` (app factory + CORS) y exponer con uvicorn.
- [x] Añadir dependencias: `uv add pydantic-settings pymysql` · `uv add --dev pytest httpx` ·
      `flutter pub add http provider flutter_dotenv shared_preferences`.
- [x] Completar la primera historia de usuario y sus specs (feature / técnica / UI / QA)
      → FEAT-001 Autenticación.
- [x] Implementar FEAT-001 (Flutter + backend).
- [x] Implementar FEAT-003 CRUD de Estudiantes (Flutter + backend).
- [ ] Completar e implementar FEAT-002 CRUD de Libros.

## Deuda Técnica / Mejoras Pendientes

- [x] ~~Backend sin `.env` ni `config.py`~~ — resuelto (`config.py` + `.env.example`).
- [x] ~~Backend sin app FastAPI real~~ — resuelto (`main.py` + uvicorn).
- [x] ~~Backend sin `apis/ services/ repositories/ schemas/` ni `db.py`~~ — resuelto.
- [x] ~~Backend sin framework de tests~~ — resuelto (pytest + httpx, 28 tests).
- [x] ~~Flutter sin MVVM~~ — resuelto (`models/ services/ viewmodels/ views/ app/ core/`).
- [x] ~~Flutter sin `test/`~~ — resuelto (15 tests).
- [x] ~~Flutter sin `.env`~~ — resuelto (`flutter_dotenv` + asset).
- [ ] **Contraseña en claro**: `estudiante.contrasena VARCHAR(4)` guarda los 4 dígitos sin hash;
      migrar a hash implica ampliar la columna y acordar longitud mínima.
- [ ] **FEAT-003 sin specs técnicas/UI/QA separadas**: la información está en la HU; considerar
      extraer a los 4 documentos estándar.
- [ ] **Nombre del paquete** `flutter_application_01` (y `name:` de `pubspec.yaml`):
      renombrar antes de publicar.
- [ ] **Sin e2e**: no hay `integration_test` configurado.
- [ ] **Sin CI** (ni `.github/workflows/` en ningún proyecto).
- [ ] **Catálogo de grados pendiente de implementar**: el backend aún no expone `GET /api/grados` y
      los formularios de registro y de estudiantes todavía generan `1°–6°` en el cliente. Debe
      reemplazarse por el catálogo de la tabla `grado` (ver `02-features/01-auth.md`).
- [ ] **JWT sin refresh token**: al expirar hay que volver a iniciar sesión.
