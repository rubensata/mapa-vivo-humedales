# Design

## Context

El visor es un `index.html` único que carga MapLibre GL JS 4.7.1 y mantiene dos capas integradas: 107 humedales en `MultiPolygon` (unos 79.000 vértices) y 8.906 establecimientos SINADER en `Point`. Los datos ya están embebidos, los filtros se resuelven en `getFilteredHumedalesFeatures()` y `getFilteredSinaderFeatures()`, y la exportación client-side existente ya resolvió descarga, metadatos, Shapefile y carga diferida de dependencias.

La restricción principal es que todo debe seguir funcionando desde `file://`, sin backend ni build step. El análisis agrega una dependencia de cómputo y una capa derivada, pero no debe alterar el modelo de filtros ni el comportamiento de las capas locales.

La exploración con Turf 7.2 sobre estos datos (Node/V8, no una medición definitiva de cada navegador) arrojó:

| Operación | Tiempo aproximado | Salida |
| --- | ---: | --- |
| Buffer individual, 8.906 puntos, 1 km, steps 8 | 1,85 s | 15,49 MB, 293.898 vértices |
| Buffer individual, 107 polígonos, 1 km, steps 8 | 9,16 s | 0,77 MB, 19.045 vértices |
| Buffer individual combinado | ~11 s | ~16,3 MB |
| Dissolve global de 8.906 puntos, 1 km, steps 8 | fallo de memoria (>2 GB) | sin resultado |
| Dissolve por lotes de 250, 1 km, steps 8 | ~54 s | 3,27 MB, 82.821 vértices, 1.740 features |
| Dissolve por lotes de 250, 1 km, steps 4 | ~48 s | 1,81 MB, 44.710 vértices |

## Goals / Non-Goals

**Goals:**

- Calcular áreas de influencia en metros sobre las capas integradas, con alcance total o filtrado.
- Producir buffers individuales o un dissolve válido, sin solapamientos internos en el modo disuelto.
- Mantener la interfaz respondiendo durante cálculos de decenas de segundos.
- Publicar el resultado como una capa temporal de análisis y permitir su exportación.
- Conservar el comportamiento de filtros, popups, capas locales y exportación existente.

**Non-Goals:**

- Analizar capas GeoJSON cargadas por el usuario.
- Persistir el resultado entre recargas o compartirlo mediante un backend.
- Implementar cancelación real del cálculo una vez iniciado.
- Añadir buffers sobre líneas, puntos de resultado de otros análisis o entradas de distancia en otras unidades.
- Cambiar la cámara automáticamente al terminar; el resultado se muestra en la vista actual y el usuario decide cómo navegar.
- Reemplazar la exportación actual de datos filtrados por la exportación del análisis.

## Decisions

### Turf 7.2 para buffer y dissolve de polígonos

Se usará `@turf/turf@7.2.0` con `buffer(feature, distancia, { units: 'meters', steps: 8 })` y `dissolve(featureCollection)`. Turf proyecta localmente para calcular la distancia y devuelve WGS84, que es el sistema de los datos y el que exige la exportación. `steps: 8` mantiene una silueta aceptable de las áreas y evita introducir una simplificación geométrica que altere el resultado.

Alternativas consideradas:

- Buffer manual con proyección propia: más código y más superficie de error; Turf ya resuelve la proyección local.
- Reducir `steps` para acelerar: no es necesario con el procesamiento por lotes y degrada la forma de las áreas.
- Calcular un buffer exacto sobre el elipsoide: Turf no ofrece ese contrato y el visor no requiere precisión geodésica sub-métrica para áreas de influencia de hasta 50 km.

### Worker inline para todo el cálculo

El análisis se ejecutará en un Web Worker creado a partir de un Blob, con Turf cargado dentro del Worker mediante `importScripts` desde la versión fijada. El hilo principal solo captura la instantánea, transfiere las features seleccionadas, recibe progreso/resultado y actualiza el DOM y MapLibre. Así la interfaz sigue respondiendo durante los ~11 s del buffer individual completo y los ~54 s del dissolve completo.

El Worker se crea la primera vez que se ejecuta un análisis, no al cargar la página. Se reutiliza mientras la página viva y se puede terminar al recargar. Las features se envían por `postMessage`; no se necesitan transferables porque el volumen de entrada es de unos 7,6 MB y el de salida de hasta ~16 MB.

Alternativas consideradas:

- Calcular en el hilo principal con `setTimeout` o `requestAnimationFrame`: no aísla el costo real y puede congelar la interfaz.
- Un Worker por ejecución: añade carga de Turf y complejidad sin beneficio para este flujo.
- Backend o servicio externo: rompe Zero-Server y la compatibilidad con `file://`.

### Dissolve por lotes espaciales y unión final

El modo dissolve no se ejecutará sobre las 8.906 geometrías en una sola llamada. Los buffers se agruparán con una partición espacial tipo quadtree por centroides, en lotes de aproximadamente 250 geometrías. Cada lote se disuelve por separado y el resultado de todos los lotes se une una vez más al final para resolver solapamientos en los bordes de la partición.

La unión es asociativa: la unión de las uniones parciales es la misma área que la unión global, por lo que el resultado conserva el contrato de dissolve. La partición acota la memoria y permite informar progreso por lote. No se usa una partición por región administrativa porque el benchmark mostró que la región Metropolitana concentra 3.731 puntos y seguía siendo un lote demasiado costoso.

El modo individual no se particiona: se generan todas las geometrías y se publican juntas; su costo es menor y no tiene el problema de memoria del dissolve.

### Ejecución explícita, no reactiva

El cálculo solo se inicia con la acción "Ejecutar buffer". No se recalcula al mover un control, cambiar un filtro o modificar la distancia. Esto evita repetir operaciones de 10 a 50 segundos y permite que el usuario prepare los parámetros antes de pagar el costo.

La instantánea se captura en el hilo principal al hacer clic. Un cambio de filtros durante el cálculo no modifica la ejecución en curso. Al terminar, el resultado anterior se reemplaza de forma atómica; si falla, el resultado anterior permanece y se informa el error.

### Modelo de resultado y capa de análisis

El resultado se mantiene en un objeto de estado independiente, no dentro de `state.layers`, para no mezclarlo con las capas de datos ni con la lista de capas locales. La capa usa un `source` y un par de capas MapLibre (`fill` y `line`) propios, se inserta sobre las capas de datos y se puede ocultar o eliminar sin tocar sus filtros.

Propiedades del resultado:

- Modo individual: `buffer_m`, `modo`, `origen`, `fuente_id`, `fuente_nombre`.
- Modo dissolve: `buffer_m`, `modo`, `origen`, `n_fuentes`.

`origen` distingue `humedales`, `sinader` o `mixto`. En el modo individual se conserva solo la trazabilidad mínima; no se copian todos los atributos originales porque la geometría ya es derivada y el Shapefile debe mantener un esquema estable.

El resultado no participa de los filtros, contadores, gráficos ni de la exportación de datos filtrados. Cambiar filtros después de un análisis no lo modifica; una nueva ejecución o la acción de limpiar son las únicas operaciones que lo reemplazan o eliminan.

### Controles, validación y accesibilidad

La sección `09 Análisis de buffer` se coloca después de `08 Exportar`. La distancia se expresa en metros, con valor inicial 1.000, mínimo 1 y máximo 50.000; se rechazan valores vacíos, decimales, no numéricos y fuera de rango. El alcance se implementa con un grupo de opciones accesible (dataset completo / solo filtradas) y el dissolve con una casilla independiente. Los controles de ejecución y exportación se deshabilitan durante el cálculo.

El estado del análisis usa `role="status"` y `aria-live="polite"`, con mensajes de carga, progreso, éxito y error. La sección conserva el patrón visual Swiss existente, etiquetas explícitas, foco visible, apilado a 375 px y respeto por `prefers-reduced-motion`. Si el alcance filtrado no tiene entidades, se informa sin crear una capa vacía.

### Exportación del resultado

La exportación del resultado reutiliza los helpers de descarga existentes. GeoJSON se genera como `FeatureCollection` en WGS84. Shapefile se genera con `shpwrite` y una sola capa de polígonos `areas_influencia`, con archivos `.shp`, `.shx`, `.dbf` y `.prj`; los campos DBF usan los mismos límites de nombre y la misma normalización ASCII que la exportación actual.

El nombre incluye `buffer`, el alcance, la distancia en metros y una marca temporal. Las acciones se habilitan solo cuando existe un resultado y no hay cálculo en curso. No se incluyen las entidades de origen ni una capa de puntos en el ZIP: el resultado del buffer es siempre poligonal.

### Carga diferida de Turf

Turf se carga desde `https://unpkg.com/@turf/turf@7.2.0/turf.min.js` dentro del Worker, solo en la primera ejecución. El bundle tiene unos 563 KB. No se añade al arranque de la página ni se carga para usar filtros, mapas o exportación GeoJSON existente.

`importScripts` no admite SRI, por lo que se fija la versión y se usa el mismo modelo de CDN que MapLibre; si el recurso no está disponible, el Worker informa el error y no publica un resultado parcial. La alternativa de vendorizar el bundle en el repositorio se descarta porque aumentaría el `index.html` y no evita la dependencia de red en el primer uso.

## Optimizaciones necesarias

Solo se incorporan las optimizaciones que el análisis previo justificó:

1. **Worker dedicado:** evita bloquear el hilo principal durante cálculos de 10–54 s y mantiene el mapa y los controles utilizables.
2. **Dissolve por lotes de ~250 geometrías con unión final:** evita el fallo de memoria del dissolve global y permite progreso granular; el resultado geométrico sigue siendo la unión completa.
3. **Ejecución explícita y reutilización del Worker:** evita recalcular buffers por cambios de UI o filtros y evita recargar Turf en cada ejecución.
4. **Carga diferida de Turf:** no penaliza el arranque del visor ni las funciones que no usan análisis.

No se añaden simplificación de polígonos, reducción de `steps`, indexación espacial adicional, rasterización, WebGL ni procesamiento en servidor: el análisis previo no las hace necesarias y introducirían pérdida de precisión o complejidad sin resolver un problema medido. La partición espacial ya forma parte de la estrategia de dissolve, no es una capa adicional de optimización.

## Risks / Trade-offs

- **El dissolve completo sigue tardando decenas de segundos** → se ejecuta en Worker, se advierte antes de iniciar y se muestra progreso; no se promete una duración menor.
- **El resultado individual completo ocupa ~16 MB y añade ~294.000 vértices al mapa** → se usa una sola capa `fill`/`line` sin símbolos por feature y se permite ocultar o limpiar el resultado.
- **El buffer de Turf es una aproximación local, no un cálculo geodésico exacto** → se limita la distancia a 50 km y se documenta que es un área de influencia para análisis, no una medición legal.
- **El CDN de Turf puede fallar o cambiar** → versión fijada, error explícito, restauración de controles y ninguna publicación parcial.
- **Un cambio de filtros durante el cálculo podría confundir el resultado** → la instantánea se captura al hacer clic y la sección muestra alcance, distancia y conteos usados.
- **Un resultado anterior podría perderse si el nuevo cálculo falla** → el resultado anterior se conserva hasta que el nuevo esté completo.
- **La partición del dissolve añade una unión final** → es necesaria para resolver solapamientos entre lotes; se mantiene un solo resultado final y no se expone la partición al usuario.
- **La UI debe seguir funcionando desde `file://`** → Worker Blob + Turf por CDN; no se depende de un archivo Worker externo ni de un servidor.

## Migration Plan

- Cambio aditivo: no hay migración de datos ni cambios en los datasets embebidos.
- La nueva sección, el Worker y la capa de análisis se pueden retirar sin afectar filtros, capas locales ni exportación existente.
- No se modifica el esquema de `openspec/specs/data-export` ni los contratos de `humedales-layer`.
- La primera ejecución descarga Turf; las siguientes reutilizan el Worker cargado.
- Rollback: eliminar la sección `09`, el estado/capa de análisis y el Worker; el resto del visor queda igual.

## Open Questions

Ninguna que cambie el alcance o los requisitos. El tamaño exacto de lote y el umbral de advertencia se ajustarán dentro del rango medido durante la implementación sin alterar el contrato funcional.
