# Tasks

## 1. Filtros de geometrías

- [x] 1.1 Corregir los filtros de relleno, línea y círculo para usar solo `$type` con valores `Polygon`, `LineString` y `Point`. Verificación: cargar el visor y comprobar con CDP/consola que la capa demo agrega sus tres subcapas y no aparecen errores `expected one of [Point, LineString, Polygon]`.
- [x] 1.2 Verificar el renderizado de `MultiPolygon` cargando `Humedales_Urbanos.geojson`. Verificación: comprobar que se agregan las capas de relleno y línea, que hay features renderizadas y que no hay errores de validación.
- [x] 1.3 Verificar que los puntos de la capa demo se dibujan y son consultables. Verificación: localizar un punto con `queryRenderedFeatures` o clic y confirmar que responde.

## 2. Reinyección tras cambio de mapa base

- [x] 2.1 Extraer el manejo de estilo a una función compartida y registrar `map.once('idle', handleStyleReady)` después de `map.setStyle` en `setBasemap`. Verificación: cambiar de OSM Dark a OSM Bright y confirmar que las fuentes y capas demo siguen presentes.
- [x] 2.2 Verificar que las capas del usuario también persisten al cambiar de mapa base. Verificación: cargar Humedales, cambiar a Bright y a Satelital, y confirmar que sus capas siguen visibles.
- [x] 2.3 Verificar el retorno a OSM Dark. Verificación: comprobar que el fondo vuelve a `#020617` y que todas las capas siguen visibles.

## 3. Regresión e integración

- [x] 3.1 Verificar que los toggles de visibilidad siguen ocultando y mostrando capas después de cambiar el mapa base. Verificación: alternar la capa demo y las capas cargadas.
- [x] 3.2 Verificar popups antes y después de cambiar el mapa base. Verificación: hacer clic en un punto demo y en un polígono de Humedales; en ambos casos se abre un único popup con atributos.
- [x] 3.3 Verificar que no se duplican manejadores tras varios cambios de mapa base. Verificación: cambiar de estilo varias veces y confirmar que un clic abre un solo popup.
- [x] 3.4 Verificar que la cámara sigue en Chile con zoom 4, que el panel lateral y el layout están intactos y que la consola no muestra errores de inicialización ni de red. Verificación: abrir el visor, inspeccionar estado del mapa y DevTools.

## Workflow follow-up

- Reanudar `instanciando-base-map` y completar sus tareas pendientes 1.3, 3.2 y 3.3 una vez corregidas las capas.
- Archivar ambos cambios cuando la implementación y verificación estén completas.
