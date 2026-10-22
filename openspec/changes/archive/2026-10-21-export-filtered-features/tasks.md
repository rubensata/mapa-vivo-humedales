# Tasks

## 1. Sección de exportación y estados

- [x] 1.1 Insertar la sección `08 Exportar` después de `#analytics-section`, con `#export-section`, título, conteo `#export-count`, botones `#export-geojson` y `#export-shapefile`, y estado `#export-status` con `role="status"` y `aria-live="polite"`. Verificar en el DOM que la sección queda dentro de `#panel-content`, que los botones son `type="button"` y que el panel conserva las secciones 01–07.
- [x] 1.2 Añadir estilos Swiss para la sección, los botones y el estado, incluyendo apilado a 375 px, foco visible y respeto por `prefers-reduced-motion`. Verificar a 375, 768 y escritorio que no hay desbordamiento horizontal y que el foco de teclado es visible.
- [x] 1.3 Incorporar una nota visible en la sección que explique que se exportan los filtros activos (no la visibilidad), que las capas locales no se incluyen y que el Shapefile normaliza tildes mientras el GeoJSON conserva el texto exacto. Verificar que la nota es legible y queda asociada a la sección mediante `aria-describedby`.
- [x] 1.4 Implementar `refreshExportState()` con los conteos de humedales y SINADER, la disponibilidad de SINADER y los estados `idle`, `sin-resultados` y `generando`; invocarlo desde `applyFilters()`, al inicializar la interfaz y después de cargar SINADER. Verificar que `#export-count` coincide con `#filter-count` tras cambiar cada filtro, limpiar filtros y cargar SINADER.
- [x] 1.5 Implementar la habilitación y deshabilitación de ambos botones según el total y el estado ocupado, con `aria-busy="true"` durante la generación. Verificar con 0 resultados, con resultados solo de humedales, con resultados solo de SINADER y durante una generación simulada que los botones no aceptan clics.

## 2. Snapshot filtrado y descarga GeoJSON

- [x] 2.1 Implementar la captura de un snapshot único con `getFilteredHumedalesFeatures()` y `getFilteredSinaderFeatures()`, incluyendo conteos y marca de tiempo. Verificar que el archivo exportado coincide con el contador del panel, que una capa oculta sigue incluida y que una capa local cargada por el usuario queda excluida.
- [x] 2.2 Verificar que el snapshot no cambia si los filtros se modifican durante la generación: iniciar una exportación, cambiar un filtro antes de que termine y comprobar que las entidades y conteos corresponden al clic inicial.
- [x] 2.3 Implementar un helper de descarga con `Blob`, `URL.createObjectURL` y un enlace temporal, con nombre `humedales_sinader_YYYY-MM-DD_HHmm.geojson` y revocación posterior del URL. Verificar que el navegador descarga un archivo con el patrón de nombre esperado y que no queda un enlace visible en el DOM.
- [x] 2.4 Construir la `FeatureCollection` GeoJSON combinada conservando geometrías y propiedades originales, sin copiar `crs`, y descargarla como `application/geo+json;charset=utf-8`. Verificar que el archivo parsea, que la suma de features coincide con el contador, que `meses` sigue siendo un arreglo y que los textos con tildes y eñes se conservan.
- [x] 2.5 Conectar el botón GeoJSON con el estado de progreso, la descarga y una notificación de éxito con los conteos; restaurar botones y estado en `finally`. Verificar el mensaje "Generando GeoJSON…" visible antes de la descarga, la notificación de éxito y la inexistencia de descargas parciales si la generación falla.

## 3. Adaptación DBF y descarga Shapefile

- [x] 3.1 Implementar el loader diferido de `https://unpkg.com/@mapbox/shp-write@0.4.3/shpwrite.js` con `integrity="sha384-OZN0ldIk+CMjlOe7C8oGHiN/KaZtlkin35SPz2uuvB0g5cY6RoecVzw2zuoFebQB"`, `crossorigin="anonymous"` y promesa cacheada. Verificar que el script no se solicita al cargar la página, que se solicita una sola vez al exportar Shapefile y que el atributo `integrity` está presente.
- [x] 3.2 Implementar los mapeos explícitos de campos DBF para humedales y SINADER, con nombres únicos de hasta 10 caracteres, textos ASCII sin diacríticos, recorte a 254 caracteres, números finitos y `meses` serializado como JSON. Verificar con un script que no hay nombres duplicados, que `meses` es texto válido y que los valores con `ñ`/tildes quedan en ASCII sin corromper el registro.
- [x] 3.3 Implementar la generación del ZIP con `shpwrite.zip(...)`, `outputType: "blob"` y `compression: "DEFLATE"`, usando `humedales_urbanos` para polígonos y `sinader_2024` para puntos. Verificar que se incluyen ambos grupos cuando hay resultados mixtos y solo el grupo con datos cuando falta uno.
- [x] 3.4 Verificar el contenido del ZIP descargado: entradas `.shp`, `.shx`, `.dbf` y `.prj` por grupo, `.prj` con WGS84, cantidad de registros DBF igual al conteo de cada capa y ausencia de archivos vacíos de la capa sin resultados.
- [x] 3.5 Conectar el botón Shapefile con el loader, el estado de progreso, la descarga `.zip` y las notificaciones de éxito/error; restaurar botones y estado en `finally`. Verificar que el mensaje "Generando Shapefile…" se pinta antes del trabajo pesado y que un fallo del loader no descarga un ZIP incompleto.

## 4. Integración y verificación final

- [x] 4.1 Ejecutar una prueba CDP de extremo a extremo abriendo `index.html` desde `file://`: aplicar filtros compartidos y propios, descargar GeoJSON y Shapefile, y comprobar conteos, contenido y nombres de archivo. Verificar que no hay solicitudes con geometrías o atributos hacia servidores.
- [x] 4.2 Verificar la accesibilidad y la respuesta visual: recorrido con teclado, anuncio de `aria-live`, `aria-busy` durante la generación, foco visible y ausencia de desbordamiento a 375, 768 y escritorio. Verificar también que `prefers-reduced-motion` no impide completar la exportación.
- [x] 4.3 Simular el fallo de carga del CDN de Shapefile y comprobar que la descarga Shapefile muestra un error comprensible, restaura los botones y que la descarga GeoJSON sigue funcionando. Verificar que la interfaz no queda bloqueada.
- [x] 4.4 Medir en navegador la exportación con el dataset completo sin filtros y con un filtro amplio; registrar tiempo, tamaño del ZIP y momento en que aparece el estado de progreso. Verificar que el mensaje se muestra antes de que termine la generación y que el ZIP abre correctamente en una herramienta SIG o biblioteca de lectura.
- [x] 4.5 Ejecutar `openspec validate --all --strict` y registrar el resultado; comprobar que las tareas completadas quedan marcadas y que no hay requisitos de exportación sin cubrir.

## Workflow follow-up

- Revisar manualmente la sección en escritorio, tablet y móvil antes de archivar.
- Archivar el cambio cuando la implementación y las verificaciones estén completas.
- Verificar el spec archivado y que la capacidad `data-export` conserva su propósito.
