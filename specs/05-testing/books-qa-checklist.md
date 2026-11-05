# Checklist QA: CRUD de Libros (FEAT-002)

> **Versión**: 1.0  
> **Fecha**: [AAAA-MM-DD]  
> **Build**: `flutter run --release` (o `flutter build apk --debug`)  
> **Especificación base**: `docs/specs/02-features/02-books.md`

---

## Instrucciones para QA

1. Backend: `cd backend && uv run uvicorn backend.main:app --reload`
2. Ajustar `flutter_application_01/.env` (`API_BASE_URL`) según el entorno
   (emulador Android: `http://10.0.2.2:8000`)
3. App: `cd flutter_application_01 && flutter run`
4. Ejecutar cada escenario en orden
5. Marcar ✅ PASS / ❌ FAIL / ⚠️ PARTIAL
6. Documentar evidencias (capturas, logs, pasos para reproducir)

---

## HU-01: [TODO: Nombre de la Historia]

| ID      | Escenario Gherkin                                | Pasos de Prueba  | Esperado | Resultado |
| ------- | ------------------------------------------------ | ---------------- | -------- | --------- |
| HU01-01 | **Dado que** ... **Cuando** ... **Entonces** ... | 1. ...<br>2. ... | ...      | ☐         |
| HU01-02 | ...                                              | ...              | ...      | ☐         |

---

## Validación Visual (UI Spec `02-books-ui.md`)

| Check | Descripción                    | Verificación | Resultado |
| ----- | ------------------------------ | ------------ | --------- |
| V-01  | [TODO] lista muestra los libros| ...          | ☐         |
| V-02  | [TODO] estados loading/empty/error | ...       | ☐         |

---

## Estados y Manejo de Errores

| Estado  | Cómo provocarlo                         | Comportamiento esperado          | Resultado |
| ------- | --------------------------------------- | -------------------------------- | --------- |
| Loading | backend caído / red lenta               | indicador en `uiState=loading`   | ☐         |
| Empty   | tabla `Book` sin filas                  | mensaje de estado vacío          | ☐         |
| Error   | `API_BASE_URL` mal escrita o API en 500 | `uiState=error` + botón reintentar | ☐       |

---

## Comunicación Backend ↔ Flutter (`.env`)

| Check  | Descripción | Resultado |
| ------ | ----------- | --------- |
| ENV-01 | La app usa la URL de `flutter_application_01/.env` (cambiarla y reconstruir cambia el destino) | ☐ |
| ENV-02 | El backend carga `DB_*` de `backend/.env` (arranca sin credenciales hardcodeadas) | ☐ |
| ENV-03 | `.env` **no** está commiteado en ninguno de los dos repos; `.env.example` sí | ☐ |
| ENV-04 | Sin secretos en el `.env` de Flutter (va empaquetado en el APK) | ☐ |

---

## Automatización

Flutter (`desde flutter_application_01/`):

```bash
flutter analyze
flutter test
dart format --output=none --set-exit-if-changed lib/ test/
```

Backend (`desde backend/`):

```bash
uv run pytest
```

| Check                | Comando                                  | Resultado |
| -------------------- | ---------------------------------------- | --------- |
| Tests nuevos pasan   | `flutter test --plain-name "Book"`       | ☐         |
| Suite completa pasa  | `flutter test`                           | ☐         |
| Analyzer limpio      | `flutter analyze`                        | ☐         |
| Backend: pytest pasa | `uv run pytest`                          | ☐         |

---

## Regresiones (Funcionalidades Existentes)

| Funcionalidad | Verificación rápida | Resultado |
| ------------- | ------------------- | --------- |
| [TODO]        | ...                 | ☐         |

---

## Resumen de Ejecución

| Métrica          | Valor |
| ---------------- | ----- |
| Total Escenarios |       |
| ✅ PASS          |       |
| ❌ FAIL          |       |
| ⚠️ PARTIAL       |       |
| 🚫 NOT RUN       |       |
| **Cobertura**    | **%** |

---

## Defectos Encontrados

| ID      | Severidad | Descripción | Pasos Repro | HU Afectada |
| ------- | --------- | ----------- | ----------- | ----------- |
| DEF-001 |           |             |             |             |

---

## Firmas

| Rol           | Nombre | Fecha | Firma |
| ------------- | ------ | ----- | ----- |
| QA Tester     |        |       |       |
| Product Owner |        |       |       |
| Tech Lead     |        |       |       |

---

> **Nota**: Este checklist debe ejecutarse **antes de considerar la feature terminada**. Cualquier FAIL bloquea.
