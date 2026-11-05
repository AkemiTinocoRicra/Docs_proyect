# Checklist QA: Autenticación — Crear cuenta e iniciar sesión (FEAT-001)

> **Versión**: 1.0  
> **Fecha**: [AAAA-MM-DD]  
> **Build**: `flutter run --release` (o `flutter build apk --debug`)  
> **Especificación base**: `docs/specs/02-features/01-auth.md`

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

## HU-01: Crear cuenta e iniciar sesión con DNI y contraseña de 4 dígitos

| ID      | Escenario Gherkin                                                                                                                                      | Pasos de Prueba                                                                                                                                 | Esperado                                                                     | Resultado |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | --------- |
| HU01-01 | **Dado que** el estudiante no tiene cuenta y está en la pantalla de registro<br>**Cuando** completa DNI, nombres, apellidos, fecha, grado, contraseña y confirmación, y pulsa "Crear cuenta"<br>**Entonces** se crea la cuenta, se persiste el JWT y se navega al home | 1. Abrir la app (sin sesión)<br>2. Ir a "Crear cuenta"<br>3. DNI: 8 dígitos nuevos<br>4. Nombres, apellidos, fecha, grado<br>5. Contraseña: 4 dígitos + confirmación<br>6. Tap "Crear cuenta" | 201; fila en `estudiante` con `estado='AC'`, `cantidad_copas=0`; JWT guardado; home visible | ☐         |
| HU01-02 | **Dado que** existe un estudiante con DNI `12345678`<br>**Cuando** otro usuario se registra con ese DNI<br>**Entonces** el backend responde 409 y la app muestra "El DNI ya está registrado" | 1. Registrarse con DNI libre (ok)<br>2. Logout / cerrar app<br>3. Registrarse de nuevo con el mismo DNI<br>4. Observar el mensaje                                | 409; mensaje "El DNI ya está registrado"; no se crea fila duplicada            | ☐         |
| HU01-03 | **Dado que** el estudiante está en registro o login<br>**Cuando** escribe una contraseña ≠ 4 dígitos numéricos<br>**Entonces** la app valida en local y no llama al backend | 1. En registro, escribir contraseña `123`, `12345`, `abcd`<br>2. Observar el mensaje bajo el campo<br>3. Repetir en login                                          | Mensaje "La contraseña debe tener 4 dígitos"; no hay petición HTTP             | ☐         |
| HU01-04 | **Dado que** el estudiante tiene cuenta creada<br>**Cuando** ingresa DNI y contraseña correctos y pulsa "Iniciar sesión"<br>**Entonces** recibe JWT, se persiste y navega al home | 1. En login, DNI correcto<br>2. Contraseña correcta (4 dígitos)<br>3. Tap "Iniciar sesión"                                                                          | 200; JWT guardado; home visible                                               | ☐         |
| HU01-05 | **Dado que** el estudiante está en login<br>**Cuando** ingresa DNI inexistente o contraseña incorrecta<br>**Entonces** el backend responde 401 y la app muestra "DNI o contraseña incorrectos" | 1. Login con DNI no registrado<br>2. Observar mensaje<br>3. Login con DNI válido y contraseña mala<br>4. Observar mensaje                                        | 401; mensaje "DNI o contraseña incorrectos"; sin token guardado               | ☐         |
| HU01-06 | **Dado que** el estudiante inició sesión y la app guardó el JWT<br>**Cuando** cierra y vuelve a abrir la app<br>**Entonces** permanece logueado sin pedir credenciales | 1. Login exitoso<br>2. Cerrar la app por completo<br>3. Reabrirla                                                                              | Home directo (sin pasar por login); token leído de shared_preferences         | ☐         |
| HU01-07 | **Dado que** el estudiante abre el registro<br>**Cuando** toca el dropdown de grado<br>**Entonces** ve solo los grados activos de la tabla `grado` | 1. Ir a "Crear cuenta"<br>2. Tocar "Grado"<br>3. Comparar con `SELECT * FROM grado WHERE estado='AC' ORDER BY id_grado` | Opciones = grados activos de la BD; sin listas hardcodeadas (1°–6°) | ☐         |
| HU01-08 | **Dado que** se envía un registro con `id_grado` inexistente<br>**Cuando** el backend valida el cuerpo<br>**Entonces** responde 422 "Grado no válido" y no crea la cuenta | 1. `POST /api/auth/registro` con `id_grado` que no exista (p. ej. 999)<br>2. Verificar respuesta y tabla | 422; sin fila nueva en `estudiante` | ☐         |

---

## Validación Visual (UI Spec `01-auth-ui.md`)

| Check | Descripción                                                                 | Verificación                                                            | Resultado |
| ----- | --------------------------------------------------------------------------- | ----------------------------------------------------------------------- | --------- |
| V-01  | Login: DNI y contraseña con teclado numérico; contraseña `obscureText`      | Abrir login; tocar cada campo                                           | ☐         |
| V-02  | Registro: todos los campos (DNI, nombres, apellidos, fecha, grado, contraseña, confirmación) | Abrir registro                                                         | ☐         |
| V-03  | Registro: dropdown de grado poblado desde `GET /api/grados` (solo activos) y selector de fecha | Tocar "Grado" y comparar con la tabla `grado`; tocar "Fecha de nacimiento" | ☐         |
| V-04  | Estados loading / error pintados desde `uiState` (botón deshabilitado)      | Provocar latencia o error                                               | ☐         |
| V-05  | Validación inline en DNI y contraseñas                                      | Escribir valores inválidos y observar mensajes                          | ☐         |
| V-06  | Links "Crear cuenta" / "Iniciar sesión" navegan correctamente               | Tocar cada link                                                         | ☐         |

---

## Estados y Manejo de Errores

| Estado  | Cómo provocarlo                                          | Comportamiento esperado                                  | Resultado |
| ------- | -------------------------------------------------------- | -------------------------------------------------------- | --------- |
| Loading | backend caído / red lenta durante login                  | indicador en `uiState=loading`; botón deshabilitado      | ☐         |
| Empty   | n/a (pantallas de formulario, sin listas)                | —                                                        | ☐         |
| Error   | `API_BASE_URL` mal escrita o API en 500                  | `uiState=error` + mensaje bajo el botón                  | ☐         |
| 409     | registrar con DNI ya existente                           | "El DNI ya está registrado"                              | ☐         |
| 401     | login con credenciales inválidas                         | "DNI o contraseña incorrectos"                           | ☐         |
| 422     | registrar con `id_grado` inexistente                     | "Grado no válido"; no se crea la cuenta                  | ☐         |
| Catálogo | `GET /api/grados` caído o sin grados activos            | Dropdown con error/reintentar o "No hay grados disponibles" | ☐         |

---

## Comunicación Backend ↔ Flutter (`.env`)

| Check  | Descripción                                                                                       | Resultado |
| ------ | ------------------------------------------------------------------------------------------------- | --------- |
| ENV-01 | La app usa la URL de `flutter_application_01/.env` (cambiarla y reconstruir cambia el destino)     | ☐         |
| ENV-02 | El backend carga `DB_*` de `backend/.env` (arranca sin credenciales hardcodeadas)                  | ☐         |
| ENV-03 | `.env` **no** está commiteado en ninguno de los dos repos; `.env.example` sí                       | ☐         |
| ENV-04 | Sin secretos en el `.env` de Flutter (va empaquetado en el APK)                                    | ☐         |
| ENV-05 | `JWT_SECRET_KEY` y `JWT_EXPIRE_MIN` en `backend/.env` (no hardcodeados)                            | ☐         |

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

| Check                    | Comando                                    | Resultado |
| ------------------------ | ------------------------------------------ | --------- |
| Tests nuevos pasan       | `flutter test --plain-name "Auth"`         | ☐         |
| Suite completa pasa      | `flutter test`                             | ☐         |
| Analyzer limpio          | `flutter analyze`                          | ☐         |
| Backend: pytest pasa     | `uv run pytest`                            | ☐         |

---

## Regresiones (Funcionalidades Existentes)

| Funcionalidad       | Verificación rápida                                   | Resultado |
| ------------------- | ----------------------------------------------------- | --------- |
| [Feature previa]    | (ninguna implementada aún; rellenar al avanzar)        | ☐         |

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
