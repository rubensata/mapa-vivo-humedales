# Design

## Context

`index.html` es un visor de archivo único que inicializa MapLibre GL JS, registra capas en un arreglo `state.layers` y las reconstruye en `reapplyAllLayers()` tras cargar el estilo o cambiar de mapa base. Cada capa usa hoy un único color para relleno, línea y círculos, y la capa inicial es un GeoJSON de demostración embebido con hitos internacionales. Ver `proposal.md` para la motivación.

El dataset `Humedales_Urbanos.geojson` contiene 107 features `MultiPolygon` en EPSG:4326, sin valores nulos, y la variable `proceso_humedal_urbano` tiene solo dos valores: `Municipal` y `de Oficio`.

## Goals / Non-Goals

**Goals:**

- Cargar los humedales como capa inicial sin depender de un servidor ni de `fetch()` sobre `file://`.
- Mantener la simbología categórica al cambiar de mapa base y al alternar la visibilidad.
- Reutilizar el modelo de capas existente sin romper la carga de archivos locales.

**Non-Goals:**

- No se agregan filtros interactivos, búsqueda ni gráficos por región o comuna.
- No se modifican el panel de mapas base, los controles de navegación ni la cámara inicial de Chile con zoom 4.
- No se simplifican ni reproyectan las geometrías del dataset.

## Decisions

### 1. Incrustar el GeoJSON en un bloque `application/json`

El contenido del archivo se incorpora en `index.html` dentro de `<script type="application/json" id="humedales-data">` y se parsea al iniciar con `JSON.parse`. Se prefiere esto a un objeto JavaScript literal porque evita escapes y mantiene los datos separados de la lógica. Se descarta `fetch('Humedales_Urbanos.geojson')` porque los navegadores bloquean esa lectura bajo `file://`, lo que rompería el modelo Zero-Server. Se descarta un archivo `.js` auxiliar para conservar el despliegue de archivo único.

El dataset se guarda minificado; se conservan todas las propiedades necesarias para popups y simbología. El crecimiento esperado de `index.html` es de aproximadamente 3,7 MB.

### 2. Reemplazar la capa demo y conservar el registro de capas

Se elimina `DEMO_GEOJSON` y su llamada de inicialización. En su lugar se registra la capa `Humedales Urbanos` con el mismo flujo de `addGeoJSONLayer`, pero con metadatos adicionales:

- `builtIn: true` para impedir su eliminación desde el panel.
- `symbology: 'process'` para indicar que el color es data-driven.
- `popupTemplate: 'humedales'` para usar la plantilla específica.

La capa se marca visible y no se ejecuta `fitBounds` al iniciar, porque la vista inicial ya está fijada en Chile con zoom 4.

### 3. Expresiones de estilo almacenadas en el registro de la capa

`reapplyAllLayers()` reconstruye las capas desde `state.layers`, por lo que la simbología no puede vivir solo en la llamada inicial. Cada registro conservará opciones de pintura opcionales (`fillColor`, `fillOpacity`, `lineColor`, `lineWidth`). Para humedales se usarán expresiones de MapLibre:

- `Municipal` → relleno `#06b6d4` con opacidad `0.45`; borde `#0891b2`.
- `de Oficio` → relleno `#f59e0b` con opacidad `0.45`; borde `#b45309`.
- Valor no reconocido → relleno gris neutro `#64748b` con opacidad `0.35`; borde `#475569`.

Ambas expresiones usan `['match', ['get', 'proceso_humedal_urbano'], ...]`. El relleno y el borde comparten la misma categoría, pero con tonos distintos para mantener legibilidad. Las capas cargadas por el usuario conservan la paleta de un solo color existente.

Alternativa considerada: agregar una propiedad derivada `_color` a cada feature. Se descarta porque muta los datos originales y complica el popup y la exportación futura del dataset.

### 4. Leyenda estática en el panel lateral

Se agrega una sección `Simbología` debajo de la lista de capas, con dos muestras de color y las etiquetas `Municipal` y `de Oficio`. La leyenda se construye desde una constante `HUMEDALES_SYMBOLOGY` que también alimenta las expresiones de estilo, para evitar que los colores se desincronicen. Se mantiene visible mientras la capa de humedales exista; si la capa se oculta, la leyenda permanece como referencia.

### 5. Popup específico con enlaces seguros

`showFeaturePopup()` conserva la tabla genérica para capas de usuario y usa una plantilla específica cuando `popupTemplate === 'humedales'`. La plantilla muestra nombre, código, región, comuna, proceso, hectáreas con formato local, resolución y los enlaces `url_res_bcn` y `url_simbio` como anclas con `target="_blank"` y `rel="noopener noreferrer"`.

Los valores de texto pasan por `escapeHtml()`. Antes de crear un enlace se valida que la URL comience con `http://` o `https://`; en caso contrario se muestra como texto plano. La superficie se formatea con `toLocaleString('es-CL')` y hasta dos decimales.

### 6. Persistencia y visibilidad

La reinyección existente (`style.load` y `idle` tras `setStyle`) seguirá cubriendo la capa porque el registro incluye sus opciones de pintura y sus IDs de subcapas. Los listeners de clic y cursor se vuelven a registrar con el mismo mecanismo `off`/`on` para evitar popups duplicados. La alternancia de visibilidad sigue usando `layer.layerIds`.

## Risks / Trade-offs

- **Tamaño del HTML** (~3,8 MB) → se minifica el JSON y se conserva una sola fuente de datos; el parseo inicial sigue siendo aceptable para 107 polígonos.
- **Colores poco legibles sobre satélite** → se usan tonos saturados con borde más oscuro y opacidad de relleno moderada; se verifica sobre los cuatro mapas base.
- **Datos con espacios finales en URLs y nombres** → la plantilla aplica `trim()` a los textos y valida el esquema de las URLs antes de crear enlaces.
- **Categorías futuras distintas de las dos actuales** → la expresión `match` incluye un color neutro de respaldo para no dejar geometrías sin estilo.
- **Pérdida de la capa demo** → es un reemplazo intencional acordado; la carga de archivos locales sigue disponible para cualquier GeoJSON externo.

## Migration Plan

1. Incorporar el bloque de datos y las constantes de simbología.
2. Sustituir la inicialización de la capa demo por la capa de humedales.
3. Extender `reapplyAllLayers()` y `addGeoJSONLayer()` para aceptar opciones de pintura y metadatos de popup/leyenda.
4. Agregar la leyenda al panel y la plantilla de popup.
5. Verificar el cambio de mapas base, la visibilidad, los popups y la carga de `Humedales_Urbanos.geojson` como archivo local.

Rollback: restaurar `DEMO_GEOJSON` y su llamada de inicialización, y retirar el bloque de datos, la leyenda y la plantilla específica. No hay migración de datos ni cambios de dependencias.

## Open Questions

Ninguna pendiente que afecte el alcance o las specs.
