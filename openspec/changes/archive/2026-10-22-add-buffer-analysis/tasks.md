# Tasks

## 1. Sección de análisis y validación

- [x] 1.1 Insertar la sección `09 Análisis de buffer` después de `#export-section`, con campo de distancia, grupo de alcance, casilla de dissolve, acciones de ejecutar/limpiar, métricas del resultado, estado accesible y acciones de exportación. Verificar en el DOM el orden de secciones, las etiquetas, los `type` correctos y que la sección queda dentro de `#panel-content`.
- [x] 1.2 Añadir estilos Swiss para el formulario, las opciones, las acciones, el estado y la advertencia de dissolve completo, incluyendo disabled, foco visible, apilado a 375 px y `prefers-reduced-motion`. Verificar con capturas y medición de overflow a 375, 768 y escritorio.
- [x] 1.3 Implementar la validación de distancia entera entre 1 y 50.000 metros, valor inicial 1.000 y error en línea sin ejecutar el análisis con valores inválidos. Verificar vacío, texto, decimal, cero, negativo, 1, 50.000 y 50.001.
- [x] 1.4 Sincronizar habilitación de ejecutar, limpiar y exportar con los estados sin resultado, calculando, resultado disponible y error, manteniendo `aria-busy` y mensajes de `aria-live`. Verificar cada transición con resultados y sin resultados.

## 2. Worker, snapshot y cálculo

- [x] 2.1 Crear el estado del análisis y la capa derivada con `source` y capas `fill`/`line` propias, sin insertarla en `state.layers` ni en el estado de capas locales. Verificar que crear, reemplazar y eliminar el resultado no altera las capas de datos, sus filtros ni sus contadores.
- [x] 2.2 Implementar el Worker inline desde Blob y la carga diferida de Turf 7.2 con `importScripts`, reutilizando el Worker en ejecuciones posteriores. Verificar que Turf no se solicita al cargar la página, que se solicita una vez en el primer análisis y que el Worker responde a un mensaje de prueba.
- [x] 2.3 Capturar la instantánea de entidades al ejecutar, distinguiendo dataset completo y solo filtradas, excluyendo capas locales y usando `getFilteredHumedalesFeatures()`/`getFilteredSinaderFeatures()`. Verificar conteos, capa oculta incluida, capa local excluida y filtros modificados durante el cálculo.
- [x] 2.4 Implementar el cálculo de buffers individuales con `buffer(..., { units: 'meters', steps: 8 })`, geometrías poligonales y atributos de trazabilidad `buffer_m`, `modo`, `origen`, `fuente_id`, `fuente_nombre`. Verificar una geometría por entidad, distancias correctas y superposiciones conservadas.
- [x] 2.5 Implementar el dissolve por lotes espaciales de ~250 geometrías y la unión final, con atributos `buffer_m`, `modo`, `origen`, `n_fuentes`. Verificar ausencia de solapamientos internos, varias partes disjuntas, cantidad de fuentes y equivalencia conceptual con la unión completa.
- [x] 2.6 Implementar progreso por fases y por lote, advertencia de dissolve completo, error comprensible, restauración de controles y conservación del resultado anterior ante fallos. Verificar bloqueo del CDN de Turf, error del Worker y que no se publique un resultado parcial.
- [x] 2.7 Verificar que el cálculo no bloquea la interfaz: iniciar un análisis completo, interactuar con el mapa y comprobar que el estado de progreso avanza y que el resultado aparece solo al finalizar.

## 3. Visualización y ciclo de vida

- [x] 3.1 Definir el estilo diferenciado de la capa de análisis, su orden sobre las capas de datos y el control de visibilidad, sin mover la cámara automáticamente. Verificar visualmente y comprobar que las capas de datos conservan su visibilidad y popups.
- [x] 3.2 Verificar el ciclo de vida: cambiar filtros no modifica el resultado existente, una nueva ejecución reemplaza el resultado de forma atómica y limpiar elimina la capa y deshabilita sus exportaciones. Verificar los conteos y metadatos mostrados tras cada operación.

## 4. Exportación del resultado

- [x] 4.1 Implementar la exportación GeoJSON del resultado con `FeatureCollection` WGS84, atributos de análisis y nombre con alcance, distancia y marca temporal. Verificar parseo, conteo, propiedades y deshabilitado sin resultado o durante el cálculo.
- [x] 4.2 Implementar la exportación Shapefile ZIP con una capa `areas_influencia`, archivos `.shp`, `.shx`, `.dbf` y `.prj`, WGS84, nombres DBF únicos de hasta 10 caracteres y normalización ASCII. Verificar entradas del ZIP, registros DBF y geometrías poligonales.
- [x] 4.3 Verificar errores y aislamiento de la exportación: un fallo de `shpwrite` muestra error y restaura botones, GeoJSON sigue operativo y la capa de análisis no se modifica. Verificar que no se descarga un archivo incompleto.

## 5. Integración y verificación final

- [x] 5.1 Ejecutar una prueba CDP desde `file://` que cubra dataset completo y filtrado, modos individual y dissolve, visualización, exportaciones GeoJSON y Shapefile, conteos y nombres. Verificar que no hay solicitudes con geometrías o atributos hacia servidores.
- [x] 5.2 Verificar accesibilidad y respuesta visual: teclado, etiquetas, `aria-live`, `aria-busy`, foco visible, error de distancia, advertencia de dissolve y ausencia de overflow a 375, 768 y escritorio. Verificar que `prefers-reduced-motion` no impide completar el análisis.
- [x] 5.3 Medir en navegador el buffer individual completo y el dissolve completo, registrando tiempo, tamaño de salida, cantidad de features y aparición del progreso; comparar con las mediciones de exploración y documentar desviaciones. Verificar que el ZIP y el GeoJSON abren correctamente.
- [x] 5.4 Ejecutar `openspec validate --all --strict`, comprobar que no hay requisitos de análisis sin cubrir y registrar el resultado.

## Workflow follow-up

- Revisar manualmente la sección y la capa resultante en escritorio, tablet y móvil antes de archivar.
- Archivar el cambio cuando la implementación y las verificaciones estén completas.
- Verificar el spec archivado y que la capacidad `buffer-analysis` conserva su propósito.
