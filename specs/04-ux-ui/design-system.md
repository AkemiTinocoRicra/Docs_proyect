# Sistema de Diseño

> **[TODO: fecha de verificación]** — basado en la configuración real del repo
> (`pubspec.yaml`, `analysis_options.yaml`).
> Estado: [TODO: sólo scaffold / con features implementadas].

## Enlaces de Diseño

- **Figma / mockups**: [TODO — insertar link cuando exista]

## Paleta de Colores

> [TODO — definir con el equipo].

| Token      | Valor       | Uso |
| ---------- | ----------- | --- |
| primary    | [TODO]      | ... |
| background | [TODO]      | ... |
| error      | [TODO]      | ... |

Implementación: `ColorScheme.fromSeed(seedColor: ...)` en `lib/app/app.dart`
(`ThemeData(useMaterial3: true)`), los tokens se consumen con
`Theme.of(context).colorScheme.*` — **nunca** colores hardcodeados en widgets.

## Tipografía y Layout

> [TODO — definir con el equipo]. Hasta entonces: defaults de Material 3
> (`Theme.of(context).textTheme.*`).

| Estilo  | Tamaño | Peso | Uso |
| ------- | ------ | ---- | --- |
| [TODO]  | ...    | ...  | ... |

## Espaciado

| Token | Valor | Uso |
| ----- | ----- | --- |
| [TODO]| ...   | ... |

## Formato de Código (obligatorio)

- **`dart format lib/`** antes de commitear (estándar del SDK: 80 columnas).
- **`flutter analyze`** debe terminar en `No issues found!` (usa `flutter_lints` vía
  `analysis_options.yaml`: `include: package:flutter_lints/flutter.yaml`).
- Indentación: 2 espacios, UTF-8, newline final (defaults de Dart).
- Regla guía: `const` donde sea posible (lo exige `prefer_const_constructors`).

## Convenciones de Widgets y Archivos

- **Archivos**: `snake_case.dart`; clases `PascalCase`; widgets privados con `_` dentro del archivo.
- **Organización por feature**: `lib/views/<feature>/<pantalla>_screen.dart` +
  `lib/views/<feature>/widgets/`; estado en `lib/viewmodels/`, datos en `lib/services/`,
  DTOs en `lib/models/` (ver `03-technicals/architecture-blueprint.md`).
- **Sin lógica en la View**: si un widget necesita HTTP o parsear JSON, ese código va al
  ViewModel/Service.
- Estados de pantalla (loading / empty / error): **obligatorios** en toda pantalla que pida
  datos; se pintan desde el `uiState` del ViewModel.

## Responsive

| Ancho    | Comportamiento                          |
| -------- | --------------------------------------- |
| < 600 px | una columna (teléfono, por defecto)      |
| >= 600  | `LayoutBuilder` / `Row` con 2 paneles (tablet) |

- Soportar orientación vertical/horizontal; evitar tamaños fijos: usar `Expanded`,
  `Flexible`, `SingleChildScrollView`.
- **Targets táctiles >= 44-48 dp** y contraste AA (WCAG) mínimo.

## Accesibilidad

- `Semantics` en controles personalizados; texto alternativo en imágenes.
- Tamaño de fuente del sistema hasta 200% sin cortar contenido.
- `flutter analyze` + revisión manual con TalkBack/VoiceOver por feature.

## Iconografía

| Acción | Icono Material |
| ------ | -------------- |
| [TODO] | `Icons....`    |
