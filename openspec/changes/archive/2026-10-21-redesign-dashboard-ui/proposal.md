# Proposal

## Why

La interfaz actual del visor usa glassmorphism oscuro, radios grandes, sombras de neón y una jerarquía tipográfica funcional pero genérica. El sistema Swiss International ofrece una identidad objetiva, tipográfica y estructural que mejora la legibilidad del panel denso de filtros y capas, y da al visor una presencia institucional coherente sin tocar su comportamiento cartográfico.

## What Changes

- Introducir una capa visual Swiss International en `index.html`: blanco, negro, gris `#F2F2F2` y rojo `#FF3000`; tipografía Inter en mayúsculas; bordes negros gruesos; radio cero; sin sombras; texturas de grid, puntos, diagonales y ruido.
- Reestructurar únicamente la UI estática: header superior, panel de control a la izquierda a altura completa en escritorio y a ancho completo en móvil, secciones numeradas y grid visible.
- Reestilizar mediante CSS las superficies generadas por JavaScript (tarjetas de capas, leyenda, popups, toasts y controles de MapLibre), sin modificar las funciones que las generan.
- Conservar todos los `id`, clases de enganche, `data-*`, textos funcionales y comportamiento de filtros, capas, popups, toasts, carga local, arrastre y mapas base.
- Ajustar `base-map` para que la preservación exigida sea funcional y de IDs, no estructural, ya que el rediseño cambia la composición visual del panel.
- No se modifican datos, lógica MapLibre, configuración de capas ni funciones JavaScript.

## Capabilities

### New Capabilities

- `dashboard-ui`: sistema visual Swiss International, composición responsive de la interfaz y presentación de controles, superficies dinámicas y estados de accesibilidad.

### Modified Capabilities

- `base-map`: los requisitos de instanciación y preservación cambian de "estructura y reubicación intactas" a "contenedor, funcionalidad, IDs y comportamiento intactos", permitiendo el rediseño visual.

## Impact

- **Archivo afectado**: solo `index.html`.
  - `<head>`: configuración de Tailwind, carga de Inter, tokens CSS y overrides de MapLibre.
  - Markup estático del `<body>`: header, panel lateral, secciones, overlay de arrastre y contenedor de toasts.
  - CSS adicional para componentes dinámicos; el bloque `<script type="module">` no se modifica.
- **Dependencias**: una hoja de Google Fonts para Inter, con fallback a Helvetica Neue/Arial; no se añaden librerías ni build.
- **Compatibilidad**: se preservan los contratos usados por el JS (`id`, `.basemap-btn`, `.layer-toggle-checkbox`, `.zoom-layer-btn`, `.remove-layer-btn`, `hidden`, `rotate-180`, clases de opacidad de toasts y overlay).
- **Fuera de alcance**: lógica de MapLibre, filtros, simbología, datos embebidos, popups (contenido), arquitectura Zero-Server y cualquier spec funcional distinta de `base-map`.
