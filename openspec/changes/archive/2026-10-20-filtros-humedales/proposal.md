# Proposal

## Why

La capa de humedales se muestra completa sobre todo Chile, por lo que explorar un territorio específico requiere acercar y revisar manualmente decenas de polígonos. Se necesitan filtros por región, comuna y proceso de declaración que reduzcan la capa y lleven la cámara directamente a las zonas seleccionadas.

## What Changes

- Agregar una sección `Filtros` en el panel lateral con tres selectores: `Región`, `Comuna` y `Proceso`, cada uno con una opción `Todas`/`Todos` para no filtrar.
- Poblar las opciones de región y proceso desde el dataset embebido y ordenarlas alfabéticamente; la lista de comunas se actualiza según la región seleccionada.
- Aplicar los filtros de forma combinada (AND) sobre `region`, `comuna` y `proceso_humedal_urbano` a la capa de humedales, sin modificar los datos originales ni afectar las capas cargadas por el usuario.
- Ajustar la cámara (`fitBounds`) a la extensión de los humedales que cumplen los filtros cada vez que cambie la selección.
- Mostrar el número de humedales que coinciden con los filtros y una notificación cuando la combinación no devuelve resultados, sin mover la cámara en ese caso.
- Agregar una acción `Limpiar filtros` que restaura todas las opciones y la vista inicial de Chile.
- Conservar los filtros aplicados al cambiar de mapa base o alternar la visibilidad de la capa.
- No se agregan dependencias, servidores, API keys ni procesos de compilación.

## Capabilities

### New Capabilities

Ninguna.

### Modified Capabilities

- `humedales-layer`: se agregan requisitos de filtrado por región, comuna y proceso, ajuste de cámara a los resultados y limpieza de filtros; la simbología, la leyenda y el popup existentes no cambian.

## Impact

- Código: `index.html` (nueva sección de filtros en el panel, estado de filtros, aplicación de `setFilter` a las subcapas de humedales, cálculo de extensión y control de cámara).
- Datos: `Humedales_Urbanos.geojson` embebido; se derivan 15 regiones, 65 comunas y 2 procesos sin alterar el dataset.
- UI: nueva sección de filtros y contador de resultados en el panel lateral; el resto de la estructura y los mapas base se mantienen.
- OpenSpec: se modifica la capability `humedales-layer`; no se crean capabilities nuevas.
