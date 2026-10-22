# Design

## Context

Ver `proposal.md` para la motivación. El visor es un único `index.html` sin build: Tailwind por CDN, MapLibre GL JS 4.7.1, datos GeoJSON embebidos y un `<script type="module">` que genera tarjetas de capas, leyenda, popups y toasts. La estética actual es glassmorphism oscuro con acentos cyan, radios grandes y sombras de neón.

Restricciones que condicionan el enfoque:

- No modificar funciones JavaScript ni la lógica de MapLibre.
- Conservar todos los `id`, `data-*`, clases de enganche y textos funcionales.
- Mantener el archivo único, sin build ni librerías nuevas.
- El mapa base inicial sigue siendo oscuro (la spec `base-map` lo exige y cambiarlo implicaría tocar JS).
- El panel lateral debe seguir colapsando mediante el botón existente.
- `base-map` exige conservar `#map` como único contenedor; la composición visual puede reorganizarse, pero el contenedor y la instanciación de MapLibre no se recrean.

## Goals / Non-Goals

**Goals:**

- Aplicar la identidad Swiss International a toda la UI visible.
- Reorganizar la composición estática (header, panel izquierdo, secciones numeradas, grid visible).
- Reestilizar también las superficies dinámicas mediante CSS, sin tocar sus plantillas JS.
- Mantener intactos los flujos de filtros, capas, carga local, popups, toasts y mapas base.
- Dejar un sistema de tokens centralizado y mantenible en un solo bloque CSS.

**Non-Goals:**

- Rediseñar el contenido de specs funcionales (`humedales-layer`, `layer-management`, `layer-rendering`, `map-viewer`).
- Cambiar la cámara, la simbología, los datos o el comportamiento de MapLibre.
- Introducir React, componentes, build, PostCSS o una librería de UI.
- Convertir el mapa a claro ni aplicar filtros de color al lienzo.
- Añadir nuevas funciones de producto (búsqueda, exportación, etc.).

## Decisions

### 1. CSS-first sobre el markup existente

**Decisión:** concentrar el sistema visual en un bloque `<style>` con tokens en `:root`, utilidades de patrón y overrides con selectores acotados por `id` o por ganchos estables. Las plantillas JS no se editan.

**Por qué:** cumple la restricción de no modificar funciones y evita duplicar estilos en plantillas que el JS regenera. Además permite revertir el rediseño tocando solo el head y el markup estático.

**Alternativa descartada:** reescribir las plantillas de `renderLayerList`, `renderLegend`, `showToast` y los popups con clases Swiss. Mejoraría el control, pero viola la restricción explícita.

### 2. Tokens centralizados y Tailwind subordinado

**Decisión:** definir la fuente de verdad en variables CSS:

- `--swiss-paper: #FFFFFF`
- `--swiss-ink: #000000`
- `--swiss-muted: #F2F2F2`
- `--swiss-accent: #FF3000`
- `--swiss-border: 2px`
- `--swiss-border-heavy: 4px`
- `--swiss-font: 'Inter', 'Helvetica Neue', Arial, sans-serif`
- `--swiss-gutter`, `--swiss-panel-width`, `--swiss-header-height`

El tema de Tailwind se extiende con colores `swiss` que apuntan a esas variables, de modo que el markup estático pueda usar utilidades sin reintroducir valores. Los overrides neutralizan `rounded-*`, `shadow-*`, `backdrop-blur-*` y los acentos cyan dentro de la UI.

**Por qué:** una sola fuente de tokens, bajo coste de mantenimiento y compatibilidad con el Tailwind CDN ya presente.

**Alternativa descartada:** eliminar Tailwind y reescribir todo el markup con CSS propio. Aumentaría el riesgo y el volumen de cambios sin beneficio funcional.

### 3. Composición estática nueva, contratos intactos

**Decisión:**

- `<header>` pasa a barra superior fija, fondo negro, tipografía Inter en mayúsculas y metadatos en mono.
- `<aside id="control-panel">` pasa a panel izquierdo a altura completa (debajo del header) en escritorio, con borde derecho grueso; en móvil ocupa todo el ancho.
- Cada sección del panel recibe un índice visible (`01`–`06`) y separadores negros.
- `#map` sigue a pantalla completa detrás de la UI; no se mueve ni se recrea.
- El overlay de arrastre se convierte en una composición blanca/negra con borde discontinuo grueso.
- El contenedor de toasts se mantiene abajo a la derecha; solo cambia su presentación.

Se conservan sin excepción: `map`, `drop-overlay`, `control-panel`, `panel-content`, `toggle-panel-btn`, `toggle-icon`, `basemap-selector`, `filters-section`, `filter-region`, `filter-comuna`, `filter-proceso`, `filter-count`, `clear-filters`, `geojson-file-input`, `layer-count`, `layers-list`, `legend-section`, `legend-content`, `toast-container`.

También se conservan `data-basemap`, `data-layer-id`, `.basemap-btn`, `.layer-toggle-checkbox`, `.zoom-layer-btn`, `.remove-layer-btn`, `hidden`, `rotate-180` y las clases de opacidad/traslación de toasts y overlay.

### 4. Estado activo de mapas base sin tocar JS

**Decisión:** `setBasemap()` reemplaza por completo `btn.className` y marca el activo con la clase Tailwind `bg-cyan-500/20`. El CSS Swiss detecta ese hook con un selector escapado y lo pinta como estado activo (fondo negro, texto blanco, barra roja). El botón inicial `dark` conserva esa clase en el markup.

**Por qué:** es el único estado que el JS expone sin modificarlo. Evita depender de `:focus`, `:hover` o posición.

**Riesgo asumido:** si en el futuro cambia esa clase en JS, el estado activo deja de verse. Queda documentado como contrato.

### 5. Superficies dinámicas por selector de contenedor

**Decisión:** estilizar con CSS:

- `#layers-list > div` para tarjetas de capa, incluidos checkbox visual, swatch, textos, botones de zoom y eliminar.
- `#legend-content > div` para filas de leyenda y su muestra de color inline.
- `.maplibregl-popup-content`, `.maplibregl-popup-tip` y `.maplibregl-popup-close-button` para popups.
- `#toast-container > div` y sus clases de tipo para notificaciones.
- `.maplibregl-ctrl-group`, `.maplibregl-ctrl button` y controles de navegación/geolocalización para integrarlos al sistema.

**Por qué:** los contenedores son estables y pertenecen al markup estático; las tarjetas y filas se regeneran pero mantienen su estructura y ganchos.

**Alternativa descartada:** añadir clases semánticas nuevas desde JS. Prohibido por la restricción.

### 6. Tipografía y patrones

**Decisión:** cargar Inter desde Google Fonts con `display=swap` y fallback Helvetica Neue/Arial. Titulares y etiquetas en mayúsculas; cuerpo en 400/500; cifras y coordenadas en mono. Aplicar patrones de grid (24 px), puntos (16 px), diagonales (10 px) y ruido SVG solo a superficies de UI, con opacidades del 2–4 %.

**Por qué:** la tipografía es la estructura principal del estilo; los patrones aportan profundidad sin sombras. El fallback garantiza operación sin red.

### 7. Accesibilidad y movimiento

**Decisión:** foco visible rojo de 2 px con offset; texto normal siempre negro sobre blanco (el rojo se reserva para titulares grandes, bordes, fondos con texto blanco e iconos); áreas táctiles de 44 px en móvil; `@media (prefers-reduced-motion: reduce)` desactiva transiciones decorativas. Se mantienen etiquetas ARIA y el orden semántico.

**Por qué:** el rojo `#FF3000` sobre blanco no alcanza AA para texto pequeño; usarlo masivamente como texto sería un problema de contraste.

### 8. Se conserva el mapa oscuro

**Decisión:** no se cambia el mapa base inicial ni se aplican filtros CSS al canvas.

**Por qué:** `base-map` exige mapa oscuro y zoom 4; alterarlo requeriría tocar JS y rompería la restricción. La UI blanca opaca sobre el mapa oscuro maximiza contraste y expresa el sistema sin comprometer la cartografía.

## Exact File Impact

Solo se modificará `index.html`:

1. **`<head>` (aprox. líneas 1–85)**
   - Enlaces a Google Fonts (preconnect + Inter).
   - Tema Tailwind extendido con `swiss` mapeado a variables CSS.
   - Reemplazo del `<style>` actual por capas: tokens, base, tipografía, patrones, componentes estáticos, overrides dinámicos, MapLibre, responsive, reduced motion.
2. **Markup estático del `<body>` (aprox. líneas 86–250)**
   - `#drop-overlay`: composición Swiss, conservando id y clases de opacidad.
   - `<header>`: barra superior, nueva jerarquía tipográfica y metadatos.
   - `<aside id="control-panel">`: panel izquierdo, cabecera, índices de sección, separadores y clases de layout.
   - Secciones internas: mapa base, filtros, carga local, capas, simbología e interacción; mismos ids, labels y atributos.
   - Botones de mapa base: mismas `data-basemap` y clases de enganche; el botón inicial conserva el hook de estado activo.
   - `#toast-container`: mismo id y posición funcional.
3. **CSS para superficies dinámicas**
   - Tarjetas de `#layers-list`, filas de `#legend-content`, `.maplibregl-popup-*`, `#toast-container`, controles de MapLibre y scrollbars.
4. **Sin cambios**
   - `<script type="module">` completo (constantes, datos, filtros, capas, popups, toasts, drag & drop, toggle, basemap).
   - `Humedales_Urbanos.geojson`, specs existentes y configuración de OpenSpec.

## Risks / Trade-offs

- **Tailwind y clases inline compiten con el CSS Swiss** → usar selectores con `id` y `!important` acotado; verificar hover, foco y estados activos en navegador.
- **`setBasemap()` borra clases personalizadas de los botones** → no depender de clases propias en esos botones; detectar el estado activo por el hook que el JS ya escribe.
- **Las plantillas JS se regeneran** → no seleccionar por clases efímeras de Tailwind salvo el hook de mapa base; preferir estructura del contenedor y atributos `data-*`.
- **Popups y toasts con estilos inline/clases Tailwind** → sobrescribir por contenedor y mantener legibilidad; probar ambos tipos de popup (humedales y genérico).
- **Panel izquierdo tapa parte del mapa en móvil** → el panel es colapsable con el control existente y el contenido hace scroll; verificar que los controles de MapLibre sigan accesibles.
- **Google Fonts no disponible** → fallback Helvetica/Arial; no bloquea la app.
- **Rojo con contraste insuficiente para texto pequeño** → reservarlo a usos grandes, bordes y fondos; texto corriente en negro.
- **Regresión funcional por cambios de estructura** → ejecutar la suite CDP existente y comprobar todos los flujos antes de archivar.

## Migration Plan

1. Implementar tokens y CSS base sin cambiar el markup; verificar que la app sigue funcionando.
2. Reestructurar header, panel y secciones estáticas conservando ids y ganchos.
3. Aplicar overrides a superficies dinámicas y controles de MapLibre.
4. Verificar escritorio y móvil, teclado, foco y movimiento reducido.
5. Ejecutar regresión funcional completa (filtros, capas, popups, toasts, carga local, mapas base, drag & drop, colapso).
6. Rollback: revertir los cambios de `index.html`; no hay migración de datos ni estado persistido.

## Open Questions

Ninguna que bloquee el diseño. La decisión de mantener el mapa oscuro queda fijada por `base-map` y por la restricción de no tocar JS.
