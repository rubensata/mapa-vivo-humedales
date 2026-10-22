# Proposal

## Why

El visor no renderiza ninguna capa GeoJSON: los filtros de estilo usan valores `$type` con variantes `Multi*` que MapLibre GL JS 4.7.1 rechaza, y al cambiar de mapa base las capas cargadas desaparecen porque el evento `style.load` no se dispara en los cambios de estilo por diff. Esto deja inutilizable la funcionalidad de datos y bloquea las tareas de regresión del cambio `instanciando-base-map`.

## What Changes

- Corregir los filtros de las capas para que acepten `Point`, `LineString`, `Polygon` y sus variantes `Multi*` sin errores de validación.
- Reinyectar las capas cargadas cuando cambie el estilo del mapa base, de modo que la capa demo y los datos del usuario persistan.
- Verificar que los popups de atributos y la alternancia de visibilidad sigan funcionando tras los cambios.
- Mantener intactos el mapa base oscuro, la cámara inicial en Chile, el panel lateral y los demás componentes.
- No se introducen cambios de comportamiento en la carga de archivos ni en la interfaz.

## Capabilities

### New Capabilities
- `layer-rendering`: renderizado de capas GeoJSON de todos los tipos de geometría y persistencia de esas capas ante cambios de estilo del mapa base.

### Modified Capabilities

Ninguna. La spec principal está vacía; se crea una capacidad específica de renderizado para no duplicar `layer-management`, que existe como delta no sincronizado en `zero-server-gis-viewer`.

## Impact

- Código: `index.html`, en los filtros de las capas de `reapplyAllLayers` y en la estrategia de reinyección posterior a `setStyle`.
- Dependencias: ninguna nueva; se mantiene MapLibre GL JS 4.7.1.
- UI: sin cambios en el panel lateral ni en los componentes existentes.
- OpenSpec: nueva capability `layer-rendering`; desbloquea las tareas 1.3, 3.2 y 3.3 de `instanciando-base-map`.
