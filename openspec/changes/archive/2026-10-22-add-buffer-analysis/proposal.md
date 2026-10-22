# Proposal

## Why

El visor permite filtrar y exportar humedales y establecimientos SINADER, pero no responde una pregunta espacial básica: qué territorio queda dentro de una distancia determinada de esos elementos. Sin un análisis de áreas de influencia, el usuario debe descargar los datos y repetir el trabajo en un SIG de escritorio. La funcionalidad debe ejecutarse en el navegador para conservar el modelo Zero-Server y el estado de filtros actual.

La exploración previa con Turf 7.2 sobre los datos reales mostró que el buffer individual es viable, pero que el dataset completo y el dissolve tienen un costo que obliga a diseñar la ejecución de forma explícita. Los 107 humedales aportan unos 79.000 vértices y los 8.906 puntos SINADER unos 8.906; en mediciones exploratorias el buffer individual completo tardó del orden de 1–2 s para puntos y 5–9 s para polígonos. El `dissolve` global de los 8.906 puntos superó 2 GB de memoria y falló; la misma operación por lotes espaciales completó en aproximadamente 50 s con una salida de 3,3 MB. Por eso el análisis se ejecutará en un Worker, se activará con una acción explícita y el dissolve se procesará por lotes.

## What Changes

- Añadir una sección `09 Análisis de buffer` al panel lateral, después de `08 Exportar`, con distancia en metros, alcance del dataset, opción de dissolve, acción de ejecución, estado accesible y acciones para limpiar o exportar el resultado.
- Permitir distancias enteras de 1 a 50.000 metros, con 1.000 metros como valor inicial. La validación impedirá ejecutar con valores vacíos, no numéricos o fuera de rango.
- Ejecutar el buffer sobre las capas integradas (`humedales` y `SINADER`) usando el dataset completo o el subconjunto que cumple los filtros activos al momento de la ejecución. Las capas locales del usuario no participan, igual que en la exportación existente.
- Ofrecer dos modos de resultado:
  - Buffers individuales: una geometría de área por cada entidad de origen, conservando la trazabilidad mínima de capa, identificador y nombre.
  - Dissolve: unión de todas las áreas generadas, sin solapamientos internos. El resultado puede contener varios polígonos disjuntos.
- Visualizar el resultado como una capa de análisis independiente, con control de visibilidad y acción de limpieza. La capa no se mezcla con las capas de datos ni con las capas locales.
- Exportar el resultado a GeoJSON y a Shapefile ZIP con una capa de polígonos y atributos genéricos de análisis: distancia, modo, origen y cantidad de fuentes. La exportación reutiliza el patrón client-side ya existente y no envía entidades a ningún servidor.
- Cargar Turf 7.2 de forma diferida, únicamente al ejecutar el primer análisis. El procesamiento pesado ocurrirá en un Web Worker creado desde un Blob para mantener la interfaz respondiendo y conservar la compatibilidad con `file://`.
- Mantener el resultado visible hasta que el usuario lo limpie o ejecute un nuevo análisis; cambiar los filtros no recalcula ni modifica el resultado existente.

## Capabilities

### New Capabilities

- `buffer-analysis`: creación, visualización y exportación de áreas de influencia configurables sobre las capas integradas, con alcance total o filtrado, modo individual o dissolve, progreso y manejo de errores.

### Modified Capabilities

<!-- Ninguna capacidad existente cambia sus requisitos. data-export conserva su comportamiento para los datos filtrados; el análisis tiene sus propias acciones de exportación. -->

## Impact

- `index.html`: nueva sección del panel, controles, estilos Swiss, estado del análisis, capa derivada en MapLibre, Worker inline y lógica de exportación del resultado.
- Dependencia nueva: `@turf/turf@7.2.0`, bundle de aproximadamente 563 KB, cargado desde CDN solo cuando se ejecuta el primer buffer. No se carga al inicio.
- Sin backend ni subida de datos. El Worker recibe únicamente las entidades seleccionadas y devuelve las geometrías calculadas en el navegador.
- Sin cambios en filtros, gráficos, capas locales, popups de datos, mapas base ni en la exportación actual. Se reutilizan los helpers de descarga y la carga diferida de `shpwrite` cuando corresponda.
- Nuevas pruebas de validación de distancia, alcance, modos, snapshot de filtros, visualización, exportación, progreso, fallo de Turf y rendimiento del dataset completo.
