# Proposal

## Why

El visor solo ofrece una capa de demostración con hitos internacionales que no representa el contexto territorial chileno. Se necesita una capa predefinida de humedales urbanos que aproveche el dataset disponible y comunique de inmediato la distinción entre humedales declarados de oficio y los declarados a solicitud municipal.

## What Changes

- Incorporar `Humedales_Urbanos.geojson` como capa predefinida del visor, incrustando sus 107 features `MultiPolygon` en `index.html` para conservar el modelo Zero-Server y el funcionamiento con doble clic (`file://`).
- Cargar la capa de humedales automáticamente al iniciar el visor y reemplazar la actual capa de demostración de hitos y monumentos.
- Aplicar simbología categórica según `proceso_humedal_urbano`:
  - `Municipal` con un color distintivo.
  - `de Oficio` con un segundo color distintivo.
- Diferenciar relleno y borde de los polígonos para mantener legibilidad sobre los mapas base oscuro, claro y satelital.
- Agregar una leyenda de simbología visible en el panel lateral con ambas categorías y sus colores.
- Mantener los popups de atributos con los campos relevantes del humedal: nombre, código, región, comuna, proceso, superficie en hectáreas, resolución y enlaces de referencia.
- Conservar la alternancia de visibilidad de la capa, su persistencia al cambiar de mapa base, la carga de archivos locales y el ajuste de cámara a la extensión de los datos.
- No se agregan dependencias, servidores, API keys ni procesos de compilación.

## Capabilities

### New Capabilities

- `humedales-layer`: capa integrada de humedales urbanos con dataset embebido, carga automática, simbología categórica por proceso de declaración, leyenda y contenido de popup específico.

### Modified Capabilities

- `layer-management`: se reemplaza la capa de demostración de hitos por la capa de humedales como capa predefinida del sistema, sin alterar la carga de archivos locales ni la inspección de atributos.

## Impact

- Código: `index.html`, dataset embebido, inicialización de la capa predefinida, estilos de relleno y borde, leyenda del panel lateral y plantilla de popup.
- Datos: `Humedales_Urbanos.geojson` (107 features, todos `MultiPolygon`, EPSG:4326); se incorpora su contenido al archivo HTML.
- Dependencias: ninguna nueva; se mantiene MapLibre GL JS 4.7.1 y Tailwind CSS por CDN.
- UI: se agrega la leyenda de simbología; el panel lateral, la lista de capas, el selector de mapas base y la carga local conservan su estructura y comportamiento.
- OpenSpec: nueva capability `humedales-layer`; se modifica `layer-management` para retirar la capa de demostración.
