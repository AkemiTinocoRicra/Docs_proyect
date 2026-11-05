# Especificación UI/UX: [Nombre de la Funcionalidad]

> Basado en: `docs/specs/02-features/NN-slug.md` · Reglas globales en `docs/specs/04-ux-ui/design-system.md`

## 1. Paleta de Colores

| Color | Valor | Uso en esta feature |
| ----- | ----- | ------------------- |
| ...   | ...   | ...                 |

> Consumir con `Theme.of(context).colorScheme.*`, no valores fijos.

## 2. Tipografía

| Estilo | Tamaño | Peso | Uso |
| ------ | ------ | ---- | --- |
| ...    | ...    | ...  | ... |

## 3. Espaciado

| Token | Valor | Uso |
| ----- | ----- | --- |
| ...   | ...   | ... |

## 4. Pantallas

### 4.1 [Nombre pantalla] (`views/<feature>/<pantalla>_screen.dart`)

```
┌─────────────────────────────────────┐
│  [wireframe ASCII]                  │
└─────────────────────────────────────┘
```

**Estados (pintados desde el `uiState` del ViewModel):**

- **Loading**: `CircularProgressIndicator` centrado / `Skeleton`
- **Error**: mensaje + botón "Reintentar"
- **Empty**: ilustración + texto guía

**Interacciones:**

| Acción | Resultado |
| ------ | --------- |
| ...    | ...       |

---

### 4.2 [Siguiente pantalla]

---

## 5. Navegación

```
'/ruta-origen' ──acción──> '/ruta-destino'
```

## 6. Accesibilidad

- Contraste AA mínimo; targets táctiles >= 44-48 dp.
- [Reglas específicas de la feature]

## 7. Responsive

| Ancho      | Comportamiento |
| ---------- | -------------- |
| < 600 px   | ...            |
| >= 600 px  | ...            |

## 8. Animaciones / Transiciones

- ...

## 9. Iconografía

| Acción | Icono Material |
| ------ | -------------- |
| ...    | `Icons....`    |
