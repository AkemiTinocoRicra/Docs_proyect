# Patrón MVVM en Flutter

> **Estado:** [TODO: definido / en revisión]
> **Última actualización:** [TODO: AAAA-MM-DD]
> Reglas globales en [architecture-blueprint.md](./architecture-blueprint.md); contrato por feature en `TEMPLATE-TECH.md`.

## 1. Estructura de carpetas

```
flutter_application_01/
├── lib/
│   ├── main.dart                    # await dotenv.load(); runApp(...)
│   ├── app/
│   │   ├── app.dart                 # MaterialApp (tema, rutas)
│   │   └── router.dart              # rutas nombradas de la app
│   ├── core/
│   │   ├── env.dart                 # Env.apiBaseUrl (flutter_dotenv) — única fuente de la URL
│   │   ├── api_client.dart          # Cliente HTTP (http): base URL, headers, timeouts, errores
│   │   └── errors.dart              # AppException (network / server / notFound / validation)
│   ├── models/                      # DTOs Dart espejo de backend/schemas/
│   │   └── <recurso>.dart
│   ├── services/                    # Capa de datos: llamadas HTTP al backend
│   │   └── <recurso>_service.dart
│   ├── viewmodels/                  # Estado de cada feature (ChangeNotifier)
│   │   └── <recurso>_viewmodel.dart
│   └── views/                       # Pantallas: solo pintan
│       └── <feature>/
│           ├── <pantalla>_screen.dart
│           └── widgets/
├── .env                             # URL base del backend (no se commitea)
├── .env.example
└── pubspec.yaml
```

## 2. Responsabilidades

| Componente | Responsabilidad | NO debe hacer |
|------------|-----------------|---------------|
| `views/` | Dibujar la UI y capturar eventos del usuario | Hacer HTTP, parsear JSON ni conocer endpoints |
| `viewmodels/` | Exponer estado, validar, invocar al *service*, notificar cambios | Construir widgets ni manejar `BuildContext` |
| `models/` | Representar los datos y su serialización JSON (`fromJson`/`toJson`) | Realizar peticiones de red |
| `services/` | Consumir la API con `api_client` y devolver modelos | Gestionar estado de UI |
| `core/env.dart` | Exponer la URL del backend desde `.env` | Contener valores hardcodeados |

## 3. Flujo de datos

```
   View  ──usuario interactúa──▶  ViewModel  ──▶  Service  ──▶  API (backend)
     ▲                              │
     └───── rebuild (notificación) ◀─┘
```

1. El **View** escucha al **ViewModel** (`context.watch` / `ListenableBuilder`).
2. El **ViewModel** llama al **Service** y transforma el resultado en estado
   (`idle`, `loading`, `success`, `error`).
3. El **Service** usa `api_client` con la URL base leída de `Env.apiBaseUrl` (`.env`).
4. El **Model** serializa/deserializa el JSON que llega de `backend/schemas/`.

## 4. Ejemplo mínimo

### `core/env.dart`

```dart
import 'package:flutter_dotenv/flutter_dotenv.dart';

class Env {
  static String get apiBaseUrl => dotenv.env['API_BASE_URL'] ?? 'http://localhost:8000';
}
```

### `services/<recurso>_service.dart`

```dart
import '../core/api_client.dart';
import '../models/<recurso>.dart';

class <Recurso>Service {
  final _client = ApiClient();

  Future<List<<Recurso>>> fetchAll() async {
    final data = await _client.get('/api/<recurso>');
    return (data as List).map(<Recurso>.fromJson).toList();
  }
}
```

### `viewmodels/<recurso>_viewmodel.dart`

```dart
import 'package:flutter/foundation.dart';
import '../services/<recurso>_service.dart';
import '../models/<recurso>.dart';

enum UiState { idle, loading, success, error }

class <Recurso>ViewModel extends ChangeNotifier {
  final <Recurso>Service _service = <Recurso>Service();

  UiState uiState = UiState.idle;
  List<<Recurso>> items = [];
  String? errorMessage;

  Future<void> load() async {
    uiState = UiState.loading;
    notifyListeners();
    try {
      items = await _service.fetchAll();
      uiState = UiState.success;
    } catch (e) {
      errorMessage = 'No se pudo cargar la información';
      uiState = UiState.error;
    }
    notifyListeners();
  }
}
```

### `views/<feature>/<pantalla>_screen.dart`

```dart
class <Recurso>Screen extends StatefulWidget { ... }

class _<Recurso>ScreenState extends State<<Recurso>Screen> {
  final vm = <Recurso>ViewModel();

  @override
  void initState() {
    super.initState();
    vm.load();
  }

  @override
  Widget build(BuildContext context) {
    return ListenableBuilder(
      listenable: vm,
      builder: (context, _) {
        if (vm.uiState == UiState.loading) return const CircularProgressIndicator();
        if (vm.uiState == UiState.error) return Text(vm.errorMessage!);
        return ListView.builder(
          itemCount: vm.items.length,
          itemBuilder: (_, i) => ListTile(title: Text(vm.items[i].title)),
        );
      },
    );
  }
}
```

## 5. Reglas transversales

- **Un ViewModel por pantalla o feature**; las Views nunca importan `services/` directamente,
  siempre pasan por el ViewModel.
- **Ningún valor de conexión hardcodeado:** la URL del backend sale de `.env`
  (ver [environment-config.md](./environment-config.md)).
- **Manejo de errores uniforme:** todo error del *service* se captura en el ViewModel y se
  expone como estado de error en la UI (`UiState.error` + `errorMessage`).
- **Estado observable:** la UI solo se reconstruye por notificación del ViewModel, nunca por
  mutación directa.
- **Estado mínimo de cada ViewModel**: `uiState` (`enum UiState`), `errorMessage` y los datos
  a pintar.
- **Registro**: `ChangeNotifierProvider` en el arranque de la ruta (raíz del árbol de la
  pantalla), para que se cree/destruya con la navegación.

## 6. Relación con las historias de usuario

| HU | View | ViewModel | Service | Model |
|----|------|-----------|---------|-------|
| FEAT-001 Autenticación | ☐ | ☐ | ☐ | ☐ |
| FEAT-002 CRUD de Libros | ☐ | ☐ | ☐ | ☐ |
| [TODO] | ☐ | ☐ | ☐ | ☐ |
