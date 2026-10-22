# Design

## Context

Véase `proposal.md` para la motivación. El diagnóstico con Chrome headless y CDP sobre `index.html` confirmó dos defectos preexistentes:

- Los filtros de `reapplyAllLayers` usan `$type` con valores `MultiPolygon`, `MultiLineString` y `MultiPoint`. MapLibre GL JS 4.7.1 solo acepta `Point`, `LineString` y `Polygon` en `$type`; al validar, descarta las capas y emite errores como `expected one of [Point, LineString, Polygon], "MultiPolygon" found`. Por eso no se renderiza ninguna capa, incluida la demo.
- El evento `style.load` solo se dispara en la carga inicial completa. Al cambiar de mapa base, `setStyle` aplica un diff y emite `styledata`, no `style.load`. Como el visor reinyecta capas en `style.load`, al cambiar de estilo se pierden las capas cargadas. En la prueba, `styledata` llegó con `isStyleLoaded() === false` y `idle` con `isStyleLoaded() === true`.

El cambio `instanciando-base-map` quedó en 5/8 tareas: su mapa base, layout y panel están implementados y verificados, pero la regresión de capas depende de este cambio.

## Goals / Non-Goals

**Goals:**

- Corregir los filtros para renderizar geometrías simples y multi.
- Reinyectar las capas después de cada cambio de estilo de mapa base.
- Mantener popups, toggles y el registro de capas existente.
- Dejar verificables las tareas pendientes de `instanciando-base-map`.

**Non-Goals:**

- Rediseñar la gestión de capas o el panel lateral.
- Agregar formatos, proveedores de mapas o dependencias.
- Cambiar la cámara inicial, el estilo base ni la estructura de la interfaz.

## Decisions

### 1. Corregir los filtros manteniendo `$type` con tipos base
- **Decisión**: Reducir los filtros a los valores aceptados por MapLibre: `Polygon` para rellenos, `LineString` o `Polygon` para líneas y `Point` para círculos. Eliminar `MultiPolygon`, `MultiLineString` y `MultiPoint`.
- **Razón**: MapLibre normaliza las geometrías multi a su tipo base en `$type`; se verificó que un `MultiPolygon` renderiza con filtro `Polygon`, un `MultiLineString` con `LineString` y un `MultiPoint` con `Point`. Es el cambio mínimo y elimina los errores de validación.
- **Alternativas descartadas**: Usar la expresión `['geometry-type']` (equivalente, pero implica reescribir los tres filtros) o partir de filtros sin `$type` (más complejo y menos legible).

### 2. Reinyectar capas en `idle` después de cambiar el estilo
- **Decisión**: Extraer el manejo de estilo a una función compartida y, en `setBasemap`, registrar `map.once('idle', handleStyleReady)` inmediatamente después de `map.setStyle(...)`. Mantener `style.load` para la carga inicial.
- **Razón**: `idle` se dispara cuando el nuevo estilo y sus fuentes ya están cargados, con `isStyleLoaded() === true`, a diferencia de `styledata`, que en la prueba llegó antes de que el estilo estuviera listo. `once` evita ejecuciones repetidas por cada ciclo de renderizado.
- **Alternativas descartadas**: Escuchar `styledata` sin esperar (falla porque el estilo aún no está cargado), usar un `setTimeout` arbitrario (frágil), o agregar un `idle` global permanente (trabajo innecesario en cada render).

### 3. Reutilizar la reinyección idempotente existente
- **Decisión**: No cambiar el registro de capas ni la lógica de `reapplyAllLayers`; solo corregir sus filtros y llamarla desde el nuevo punto de disparo.
- **Razón**: `reapplyAllLayers` ya comprueba la existencia de fuentes y capas, y re-registra los manejadores de clic con `off`/`on`. Mantenerla reduce el riesgo de duplicados y de regresiones en popups o toggles.

### 4. Preservar la configuración del mapa base
- **Decisión**: Mantener el mapa oscuro, la cámara inicial en Chile con zoom 4 y el fondo `#020617`; el manejador compartido también reaplica el fondo oscuro al volver a OSM Dark.
- **Razón**: El cambio es correctivo de capas y no debe alterar el comportamiento ya verificado de `instanciando-base-map`.

## Risks / Trade-offs

- **[`idle` puede tardar si la red de mosaicos es lenta]** → *Mitigación*: las capas reaparecen en cuanto el estilo termina de renderizar; no se pierden datos y el usuario puede seguir interactuando.
- **[Cambios rápidos de mapa base acumulan manejadores `once('idle')`]** → *Mitigación*: `reapplyAllLayers` es idempotente y no duplica fuentes, capas ni listeners.
- **[Dependencia de la semántica legacy de `$type`]** → *Mitigación*: se documenta la equivalencia con tipos base; si MapLibre la deprecara, la migración a `geometry-type` queda acotada a tres filtros.
- **[Los popups podrían duplicar manejadores tras reinyectar]** → *Mitigación*: se conserva el patrón `map.off(...)` antes de `map.on(...)` y se verifica con clics tras cambiar de mapa base.

## Migration Plan

No se requiere migración. Los cambios se aplican directamente en `index.html` y pueden revertirse restaurando los filtros originales y el disparo desde `style.load`.
