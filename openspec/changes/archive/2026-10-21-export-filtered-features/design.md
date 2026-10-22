# Design

## Context

Ver `proposal.md` para la motivación. El visor es un único `index.html` que ya mantiene el estado de filtros en `filterState` y obtiene los subconjuntos con `getFilteredHumedalesFeatures()` y `getFilteredSinaderFeatures()`. `applyFilters()` actualiza el contador de la sección 02 y los gráficos de la sección 07; la sección 08 de exportación se añadirá después de análisis.

Los datos están embebidos: 107 humedales (`MultiPolygon`) y 8.906 establecimientos SINADER (`Point`). SINADER se inicializa después de que el mapa carga, por lo que la interfaz de exportación debe reaccionar a esa disponibilidad. La aplicación es Zero-Server y debe funcionar al abrirse desde `file://`; ya depende de CDN para MapLibre y Tailwind.

Restricciones relevantes:

- Shapefile no admite polígonos y puntos en un mismo `.shp`.
- DBF limita los nombres de campo a 10 caracteres y no representa arreglos anidados.
- `shp-write` no genera archivo `.cpg`.
- El navegador no ofrece una API nativa para escribir Shapefile.

## Goals / Non-Goals

**Goals:**

- Exportar exactamente el subconjunto filtrado de las capas integradas, con un snapshot tomado al iniciar la descarga.
- Ofrecer GeoJSON sin dependencias y Shapefile con una única dependencia cargada de forma diferida.
- Mantener una experiencia accesible con conteos, estados deshabilitados, progreso, éxito y error.
- No introducir backend, build step ni cambios en el comportamiento de filtros, capas, popups o gráficos.

**Non-Goals:**

- Exportar capas GeoJSON cargadas por el usuario.
- Exportar solo lo visible en el viewport o respetar la visibilidad de capas.
- Exportar CSV, Excel, KML, GeoPackage o Shapefile con múltiples capas en un solo `.shp`.
- Generar archivos en un Web Worker.
- Añadir `.cpg`, codificación distinta de ASCII en DBF o metadatos adicionales dentro del ZIP.
- Editar atributos o geometrías antes de exportar.

## Decisions

### 1. Snapshot del estado filtrado, no de la escena renderizada

La exportación parte de las funciones de filtrado existentes y captura `{ humedales, sinader, total, capturedAt }` en el clic. No usa `queryRenderedFeatures`, porque ese método depende del viewport y de la visibilidad, y produciría archivos distintos a los contadores y gráficos. La visibilidad de capa se ignora de forma deliberada; el alcance es "filtros activos", no "lo que se ve".

La captura se hace una sola vez por descarga. Si el usuario cambia filtros mientras se genera el ZIP, el archivo sigue representando el estado del clic. Las capas locales no participan porque no tienen `filterKey` ni entran en `filterState`; esto mantiene la coherencia con el resto del panel.

Alternativas descartadas:

- **Leer features renderizadas:** excluye capas ocultas y datos fuera de pantalla.
- **Releer filtros al finalizar:** puede mezclar dos estados y romper la correspondencia con el conteo anunciado.
- **Incluir capas locales:** obligaría a definir filtros para datos arbitrarios y amplía el alcance sin una necesidad clara.

### 2. Contrato de salida

**GeoJSON:** una sola `FeatureCollection` con `type: "FeatureCollection"` y `features` concatenando humedales y SINADER. Se conservan las propiedades originales sin transformarlas y las geometrías en WGS84 (EPSG:4326). No se copia el miembro `crs` de la fuente, porque GeoJSON RFC 7946 asume WGS84 y el archivo combinado no necesita esa ambigüedad.

**Shapefile:** un único ZIP con hasta dos shapefiles, uno por tipo de geometría:

| Grupo | Geometría | Archivos | Prefijo |
| --- | --- | --- | --- |
| Humedales urbanos | `MultiPolygon` | `.shp`, `.shx`, `.dbf`, `.prj` | `humedales_urbanos` |
| SINADER 2024 | `Point` | `.shp`, `.shx`, `.dbf`, `.prj` | `sinader_2024` |

Solo se incluye un grupo si tiene al menos una entidad. Un ZIP con una sola capa sigue siendo válido. El nombre del archivo usa la fecha y hora local con formato `humedales_sinader_YYYY-MM-DD_HHmm.geojson` o `.zip`, lo que evita sobrescrituras accidentales y facilita ordenar exportaciones.

Alternativas descartadas:

- **Dos descargas separadas:** duplica la interacción y puede dejar al usuario con la mitad de la exportación.
- **Un `.shp` combinado:** técnicamente inválido para geometrías mixtas.
- **Convertir todo a puntos o polígonos:** perdería la geometría real.
- **ZIP con GeoJSON dentro:** no cumple el requisito de Shapefile utilizable en SIG.

### 3. Dependencia: `@mapbox/shp-write@0.4.3` cargada solo para Shapefile

Se evaluó la opción de menor peso y riesgo que permita generar un ZIP de Shapefile en el navegador:

| Opción | Peso / dependencias | Escritura Shapefile | Veredicto |
| --- | --- | --- | --- |
| APIs nativas (`Blob`, `URL`) | 0 | No aplica a SHP | Se usan para GeoJSON y descarga |
| `@mapbox/shp-write@0.4.3` | 219.881 bytes; incluye `dbf`, `jszip` y `file-saver` en el bundle | Sí, `.shp/.shx/.dbf/.prj` y ZIP | **Elegida** |
| `shpjs` | Decenas de KB | No, solo lectura | Descartada |
| `shapefile` (npm) | Requiere streams y polyfills de Node | Sí, pero orientada a Node | Descartada |
| Escritor SHP/SHX/DBF manual | 0 dependencias, cientos de líneas binarias | Sí, con alto riesgo | Descartada |
| Conversión en servidor | Backend y subida de datos | Sí | Viola Zero-Server |

La versión se fija en `0.4.3`, con licencia BSD-2-Clause, desde:

`https://unpkg.com/@mapbox/shp-write@0.4.3/shpwrite.js`

Integridad SHA-384 del bundle verificado (219.881 bytes):

`sha384-OZN0ldIk+CMjlOe7C8oGHiN/KaZtlkin35SPz2uuvB0g5cY6RoecVzw2zuoFebQB`

El script se inyecta dinámicamente solo al pulsar "Descargar Shapefile", con `integrity` y `crossorigin="anonymous"`. Unpkg responde con `Access-Control-Allow-Origin: *`, por lo que SRI es aplicable. La promesa de carga se cachea para no descargar el bundle dos veces. Si el script falla, solo se rechaza la exportación Shapefile; el GeoJSON sigue operativo.

Se usa `shpwrite.zip(features, { outputType: "blob", compression: "DEFLATE", types: { polygon: "humedales_urbanos", point: "sinader_2024" } })`. La API agrupa por tipo de geometría y genera el `.prj` WGS84 por defecto. El README advierte sobre la compresión; la versión 0.4.3 usa `DEFLATE` por defecto y la prueba local produjo un ZIP válido, por lo que se verificará también en navegador antes de cerrar las tareas.

Rendimiento medido en Node con el dataset completo (107 polígonos + 8.906 puntos):

- `STORE`: 746 ms, 29,8 MB.
- `DEFLATE`: 1.427 ms, 1,84 MB.

Se elige `DEFLATE`: el tiempo adicional es aceptable con feedback y el tamaño es 16 veces menor. Los subconjuntos filtrados suelen ser mucho más pequeños.

### 4. Adaptación explícita de atributos para DBF

No se pasan las propiedades crudas a `shp-write`, porque los nombres superan los 10 caracteres, pueden colisionar y `meses` es un arreglo. Se define un mapeo explícito por capa:

| Humedales | Campo DBF | Tipo |
| --- | --- | --- |
| `objectid` | `objectid` | N |
| `region` | `region` | C |
| `comuna` | `comuna` | C |
| `cod_humedal` | `cod_hum` | C |
| `nombre_humedal` | `nombre` | C |
| `ha` | `ha` | N |
| `proceso_humedal_urbano` | `proceso` | C |
| `resolucion` | `resolucion` | C |
| `url_res_bcn` | `url_bcn` | C |
| `url_simbio` | `url_simbio` | C |
| `srid` | `srid` | C |

| SINADER | Campo DBF | Tipo |
| --- | --- | --- |
| `id` | `id` | C |
| `nombre` | `nombre` | C |
| `razon` | `razon` | C |
| `rut` | `rut` | C |
| `region` | `region` | C |
| `provincia` | `provincia` | C |
| `comuna` | `comuna` | C |
| `rubro` | `rubro` | C |
| `toneladas` | `toneladas` | N |
| `registros` | `registros` | N |
| `tratamiento` | `tratam` | C |
| `meses` | `meses` | C (JSON) |
| `t_eliminacion` | `t_elim` | N |
| `t_recepcion` | `t_recep` | N |
| `t_valorizacion` | `t_valor` | N |
| `ler1` / `ler2` / `ler3` | `ler1` / `ler2` / `ler3` | C |

Reglas de normalización:

- Los textos se transliteran a ASCII con descomposición Unicode y eliminación de diacríticos (`ñ` → `n`, tildes eliminadas). Esto evita bytes ambiguos sin `.cpg` y mantiene los nombres legibles.
- Los textos se recortan a 254 caracteres, límite de un campo carácter DBF.
- `meses` se serializa con `JSON.stringify` en una cadena compacta, por ejemplo `[7.783,8.638,...]`; el GeoJSON conserva el arreglo original.
- Los números se envían como números; los valores no finitos se convierten en `null` para no ensuciar la tabla.
- Los nombres de campo anteriores son únicos y cumplen la longitud máxima de 10.

Los mapeos son deliberados y estables: no dependen del orden de las propiedades ni de heurísticas de truncamiento. La pérdida de tildes queda limitada al Shapefile y se documenta en la interfaz; el GeoJSON es la vía para conservar los valores exactos.

### 5. Interfaz y máquina de estados

La sección `08 Exportar` se inserta después de `#analytics-section` y antes del cierre de `#panel-content`. Incluye:

- `#export-section` con `aria-labelledby="export-title"`.
- Conteo `#export-count`, actualizado con los mismos valores que `#filter-count`.
- Botones `#export-geojson` y `#export-shapefile`, ambos `type="button"`.
- Estado `#export-status` con `role="status"` y `aria-live="polite"`.

Estados:

| Estado | Conteo | Botones | Mensaje |
| --- | --- | --- | --- |
| `idle` con resultados | "X humedales · Y establecimientos" | Habilitados | "Listo para exportar los filtros activos." |
| `idle` sin resultados | "0 humedales · 0 establecimientos" | Deshabilitados | "Sin resultados para exportar." |
| `generating-geojson` | Se mantiene el snapshot | Deshabilitados, `aria-busy="true"` | "Generando GeoJSON…" |
| `generating-shapefile` | Se mantiene el snapshot | Deshabilitados, `aria-busy="true"` | "Generando Shapefile…" |
| `error` | Se mantiene | Habilitados si hay resultados | "No se pudo generar el archivo." |

`refreshExportState()` se invoca al final de `applyFilters()`, después de cargar SINADER y al inicializar la interfaz. Si `SINADER_DATA` aún es `null`, el conteo SINADER es cero y los botones se habilitan si hay humedales; al cargar la capa se refresca el estado. La sección indica que la exportación depende de los filtros, no de la visibilidad, y que las capas locales no se incluyen.

El estado ocupado se anuncia con `aria-busy` en la sección y un texto `aria-live`. Las notificaciones reutilizan `showToast` con conteos y formato. Al terminar o fallar se restaura el estado en `finally`, evitando botones bloqueados.

### 6. Descarga y manejo de errores

GeoJSON:

1. Capturar el snapshot.
2. Construir `JSON.stringify(collection)`.
3. Crear `Blob` con tipo `application/geo+json;charset=utf-8`.
4. Crear un object URL, un `<a download>` oculto, hacer clic y revocar el URL después de un segundo.

Shapefile:

1. Capturar el snapshot.
2. Cargar el bundle con el loader cacheado; si falla, lanzar un error específico.
3. Adaptar los atributos al mapeo DBF.
4. Esperar `shpwrite.zip(...)` con `outputType: "blob"` y `compression: "DEFLATE"`.
5. Descargar el Blob con el mismo mecanismo y un nombre `.zip`.

Antes de la generación pesada se cede un frame (`requestAnimationFrame` o `setTimeout(0)`) para que el mensaje "Generando…" se pinte. No se usa Web Worker: el dataset completo tarda ~1,4 s en Node y agregaría complejidad de serialización; el feedback y los botones deshabilitados cubren el caso. Si en pruebas de navegador el bloqueo resulta inaceptable, se podrá evaluar un worker en otro cambio sin alterar el contrato externo.

No se descarga nada si la generación falla. Los blobs se revocan para liberar memoria y no se conservan referencias globales.

### 7. Alcance de pruebas

Las pruebas de aceptación cubren:

- Conteos y archivos con filtros compartidos y propios.
- Capa oculta incluida; capa local excluida.
- Conteo cero y deshabilitado; restauración al limpiar filtros.
- GeoJSON parseable, cantidad y propiedades originales.
- ZIP con entradas esperadas por geometría, `.prj` WGS84 y registros DBF.
- Nombres DBF únicos, `meses` serializado y ASCII.
- Estado de progreso, éxito, error y bloqueo durante generación.
- Fallo simulado de la dependencia sin afectar GeoJSON.
- Apertura desde `file://` y ausencia de subida de datos.

## Risks / Trade-offs

- **Pérdida de tildes y eñes en DBF** → Se normaliza a ASCII para evitar corrupción sin `.cpg`; el GeoJSON conserva el texto exacto y la interfaz lo advierte.
- **Dependencia antigua y advertencia de compresión** → Versión fijada, SRI, prueba de ZIP válido y carga diferida; un fallo solo afecta Shapefile.
- **CDN no disponible** → La aplicación ya requiere CDN para el mapa; el GeoJSON no necesita esta dependencia y el error de Shapefile es explícito.
- **Bloqueo del hilo principal durante DEFLATE** → Mensaje previo, botones deshabilitados y snapshots pequeños en la mayoría de los filtros; se medirá en navegador.
- **Límite de 10 caracteres y colisiones** → Mapeo explícito y verificación de unicidad en pruebas.
- **SRI desactualizado si cambia el CDN** → La URL incluye una versión inmutable; el fallo es visible y no silencioso.
- **Confusión entre visibilidad y filtros** → Copia de la sección y mensaje de estado explican que se exportan los filtros activos, incluidas capas ocultas.
- **Memoria de blobs** → Revocación de object URLs y ausencia de retención global.

## Migration Plan

Es un cambio estático sobre `index.html`: no hay migración de datos, esquema ni almacenamiento persistente. Se despliega reemplazando el archivo y se revierte eliminando la sección, el módulo de exportación y el loader de la dependencia. No requiere coordinación de backend ni ventana de mantenimiento.
