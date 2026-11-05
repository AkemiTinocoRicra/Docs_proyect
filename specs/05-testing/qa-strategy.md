# Estrategia de Control de Calidad (QA)

> **[TODO: fecha de verificación]** — comandos verificados contra el scaffold.

## Stack de Testing

- **Flutter**: `flutter_test` (en `pubspec.yaml`) + `flutter analyze` (`flutter_lints`).
  [TODO: existe carpeta `test/` — si no, `flutter test` falla con `Test directory "test" not found`;
  crear `test/` con la primera feature.]
- **Backend**: [TODO: framework de tests] → `uv add --dev pytest httpx`
  (`httpx` para `fastapi.testclient`).
- **Sin e2e**: [TODO: hay `integration_test` configurado o no].

## Comandos

```bash
# Flutter  (desde flutter_application_01/)
flutter analyze                 # estático — objetivo: 0 issues
flutter test                    # unit + widget tests (requiere test/)
flutter test --coverage         # genera coverage/lcov.info
dart format lib/ test/          # formato
flutter run                     # verificación manual en dispositivo/emulador

# Backend  (desde backend/)
uv sync                         # dependencias
uv run pytest                   # tests (tras añadir pytest)
uv run uvicorn backend.main:app --reload   # verificar endpoints a mano
```

## Umbrales

- `flutter analyze`: **0 issues**.
- Tests Flutter: **100% pasando** en `flutter test`.
- Tests backend: **100% pasando** en `uv run pytest`.
- Formato: `dart format` sin cambios pendientes en los archivos tocados.
- Cobertura de la feature nueva: ViewModel + modelos cubiertos (objetivo inicial: sin % fijo).

## Lista de Verificación Pre-Merge (Definition of Done)

1. [ ] `flutter analyze` → `No issues found!`.
2. [ ] `flutter test` → todos los tests pasan.
3. [ ] `uv run pytest` → todos los tests pasan (si toca backend).
4. [ ] Tests nuevos escritos para la funcionalidad (ViewModel + service con mocks;
       `services/` del backend).
5. [ ] Archivos formateados (`dart format lib/ test/`).
6. [ ] Especificación manual con el checklist de la feature (abajo) en **PASS**.
7. [ ] Specs actualizadas (`02-features/`, `03-technicals/`, `04-ux-ui/`).
8. [ ] Backlog actualizado en `docs/specs/01-product/product-backlog.md`.
9. [ ] Si toca configuración: `.env.example` actualizado y `.env` sigue sin commitearse.

## Checklist por Feature

Usar `05-testing/TEMPLATE-QA-CHECKLIST.md` por funcionalidad y ejecutarlo manualmente
**antes de considerar la feature terminada**. Cualquier FAIL bloquea.

## Matriz de Pruebas por Capa

| Capa                | Qué se prueba                                | Herramienta        |
| ------------------- | -------------------------------------------- | ------------------ |
| ViewModel (Flutter) | transiciones de `uiState`, errores           | `flutter test`     |
| Model (Flutter)     | `fromJson` / `toJson`                        | `flutter test`     |
| Service (Flutter)   | URL armada desde `Env.apiBaseUrl`, parsing   | `flutter test` (mock del `http.Client`) |
| View (Flutter)      | render por estado, taps                      | `flutter test` (widget test) |
| services/ (API)     | reglas de negocio sin servidor               | `pytest`           |
| apis/               | status codes y validación (TestClient)       | `pytest` + `httpx` |
| repositories/       | SQL contra BD de prueba                      | `pytest` (BD dedicada) |
| Extremo a extremo   | pantalla → endpoint → respuesta              | manual (checklist) |
