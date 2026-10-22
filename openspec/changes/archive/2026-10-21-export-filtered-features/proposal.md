# Proposal

## Why

El visor ya permite filtrar humedales y establecimientos SINADER, pero no permite llevarse el subconjunto que el usuario está analizando. Hoy la única alternativa es descargar o procesar el dataset completo, lo que obliga a repetir los filtros fuera del visor y aumenta el riesgo de usar datos desactualizados o distintos a los que se ven en pantalla. Se necesita una exportación client-side que respete exactamente el estado filtrado actual.

## What Changes

- Añadir una sección `08 Exportar` en el panel lateral con el conteo del subconjunto filtrado, dos acciones de descarga y un estado de progreso accesible.
- Exportar un `FeatureCollection` GeoJSON con las entidades filtradas de humedales y SINADER, conservando geometrías y propiedades originales.
- Exportar un ZIP con shapefiles separados por tipo de geometría: humedales como polígonos y SINADER como puntos, con sus archivos `.shp`, `.shx`, `.dbf` y `.prj`.
- Deshabilitar ambas descargas cuando no existan entidades filtradas y mientras se esté generando un archivo.
- Mostrar feedback visible y accesible durante la generación, además de notificaciones de éxito o error con los conteos exportados.
- Incorporar una única dependencia externa para Shapefile, cargada de forma diferida y fijada a una versión; la descarga GeoJSON no requiere dependencias.
- Mantener todo el procesamiento y la descarga en el navegador, sin backend, sin subir datos y sin alterar el comportamiento de filtros, capas o mapas base.

## Capabilities

### New Capabilities

- `data-export`: exportación client-side del subconjunto filtrado a GeoJSON y Shapefile comprimido, con conteos, estados deshabilitados, feedback de progreso y manejo de errores.

### Modified Capabilities

<!-- Ninguna capacidad existente cambia sus requisitos. La exportación consume el estado de filtros ya definido por humedales-layer y no modifica su comportamiento. -->

## Impact

- `index.html`: nueva sección del panel, estilos Swiss, controles de exportación, estado de progreso y lógica de generación/descarga.
- Dependencia nueva: `@mapbox/shp-write@0.4.3` cargada desde CDN solo al solicitar Shapefile. El GeoJSON usa `Blob` y `URL.createObjectURL` nativos.
- Reutiliza `filterState`, `getFilteredHumedalesFeatures()` y `getFilteredSinaderFeatures()` sin cambiar filtros, contadores, cámara, gráficos ni capas.
- Sin backend, sin carga del Excel en runtime y sin cambios en los datasets embebidos.
- Pruebas nuevas de exportación, estados vacíos, progreso, deshabilitado, contenido GeoJSON, entradas ZIP y fallo de carga de la dependencia.
