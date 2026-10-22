# Tasks

## 1. Preparación y validación del dataset

- [x] 1.1 Generar el GeoJSON agregado por `id_vu` desde `gi-sinader-2024-ckan.xlsx` con el conversor local, conservando seis decimales en las coordenadas y sin incluir columnas de estilo.
- [x] 1.2 Verificar que el resultado contiene 8.906 establecimientos, 120.767 registros y 5.634.872,476 toneladas, y que las 18 coordenadas inválidas y 58 fuera de Chile quedaron excluidas.
- [x] 1.3 Comprobar que la suma mensual cuadra con el total, que la suma por tratamiento solo difiere por las 22,514 t sin clasificar del establecimiento 72418 y el redondeo, que `toneladas` y `registros` coinciden por establecimiento y que las 220 coordenadas compartidas conservan sus entidades.
- [x] 1.4 Confirmar que el GeoJSON minificado mide alrededor de 4,54 MB, no contiene `</script>` sin escapar y se codifica en UTF-8.
- [x] 1.5 Registrar en la verificación el hash SHA-256 del xlsx y los totales de control para futuras regeneraciones.

## 2. Empaquetado y carga diferida

- [x] 2.1 Insertar el GeoJSON en un bloque `<script type="application/json" id="sinader-data">` después de los datos de humedales, sin convertirlo en JavaScript ejecutable.
- [x] 2.2 Declarar `SINADER_DATA` y la inicialización con guarda de idempotencia, ejecutada en `map.once('load')` o de inmediato si MapLibre ya cargó.
- [x] 2.3 Parsear el bloque únicamente dentro de la inicialización y envolver la carga en `try/catch` para notificar un error sin afectar humedales ni los controles.
- [x] 2.4 Extender `addGeoJSONLayer` con opciones opcionales para `filterKey`, radio/opacidad/borde de círculo, simbología y plantilla de popup, sin cambiar el comportamiento por defecto de las capas existentes.
- [x] 2.5 Registrar la capa SINADER después de humedales, visible por defecto, con 8.906 puntos, tarjeta en el panel y persistencia al cambiar el mapa base.
- [x] 2.6 Actualizar el metadato de fuentes de la cabecera para mencionar humedales y SINADER 2024.

## 3. Simbología, leyenda y popups

- [x] 3.1 Declarar `SINADER_SYMBOLOGY` con las tres categorías, etiquetas y colores, y construir desde allí las expresiones de color del mapa.
- [x] 3.2 Configurar los círculos con radio progresivo por zoom, borde claro y opacidad legible en los cuatro mapas base, sin estilos embebidos en los datos.
- [x] 3.3 Ampliar `renderLegend` para mostrar la leyenda de humedales y el grupo SINADER con las tres categorías, usando las mismas entradas de simbología.
- [x] 3.4 Implementar el popup individual con nombre o razón social, RUT, región, comuna, rubro, tonelaje, registros, tratamiento dominante y capítulos LER, incluyendo marcadores para campos vacíos.
- [x] 3.5 Implementar el popup de coincidencias con `queryRenderedFeatures` cuando la coordenada agrupe varios establecimientos, listando todos con nombre, tonelaje y tratamiento dominante.
- [x] 3.6 Verificar que ocultar y mostrar la capa conserva filtros, popup y simbología, y que la tarjeta muestra el total de 8.906 establecimientos.

## 4. Filtros, contador y cámara

- [x] 4.1 Migrar el estado de filtros a `region`, `comuna`, `proceso`, `rubro` y `tratamiento`, conservando los IDs `filter-region`, `filter-comuna`, `filter-proceso`, `filter-count` y `clear-filters`.
- [x] 4.2 Añadir los selectores `filter-rubro` y `filter-tratamiento` con opción `Todos`, poblados desde SINADER y ordenados alfabéticamente.
- [x] 4.3 Construir región y comuna como unión de ambos datasets, manteniendo la cascada región-comuna y refrescando las opciones al registrarse SINADER sin perder la selección.
- [x] 4.4 Aplicar a cada capa su propio constructor de filtros: humedales por región/comuna/proceso; SINADER por región/comuna/rubro/tratamiento con presencia de tonelaje en la categoría elegida.
- [x] 4.5 Actualizar `filter-count` con el formato `X humedales · Y establecimientos`, formateado en locale chileno, y recalcular ambos contadores en cada cambio.
- [x] 4.6 Ajustar la cámara según el origen del cambio: humedales para filtros compartidos/proceso, SINADER para rubro/tratamiento, vista de Chile con zoom 4 al limpiar y advertencia sin resultados.
- [x] 4.7 Confirmar que `Limpiar filtros` restablece las cinco claves, ambos catálogos y ambas capas, y que los filtros de humedales siguen funcionando igual que antes.

## 5. Sección de análisis y gráficos

- [x] 5.1 Añadir la sección `07 Análisis` después de `06 Interacción`, con tres contenedores SVG, estilos Swiss y estados vacíos, sin alterar las secciones ni los IDs existentes.
- [x] 5.2 Renderizar el gráfico de humedales por proceso con conteos filtrados y los colores de la simbología de humedales.
- [x] 5.3 Renderizar el gráfico SINADER de tonelaje mensual con doce barras y etiquetas numéricas visibles.
- [x] 5.4 Renderizar el gráfico SINADER por tratamiento con las tres categorías y los colores de `SINADER_SYMBOLOGY`.
- [x] 5.5 Enlazar los tres gráficos a la misma actualización de filtros, sin depender de la visibilidad de las capas, y mostrar un estado vacío cuando una capa no tenga resultados.
- [x] 5.6 Añadir `role="img"`, `<title>`, `<desc>`, etiquetas visibles y `viewBox` responsive; respetar `prefers-reduced-motion` y comprobar 375, 768 y 1440 px.
- [x] 5.7 Verificar que no se añade ninguna librería de gráficos ni se realizan solicitudes de red para renderizar la sección.

## 6. Verificación funcional y regresión

- [x] 6.1 Ampliar la suite CDP temporal con pruebas de carga de SINADER, 8.906 puntos, conteo dual, filtros compartidos y propios, cascada de comunas y limpieza.
- [x] 6.2 Probar popup individual, popup de coincidencias con una coordenada de 19 establecimientos, leyenda conjunta, alternancia de visibilidad y persistencia tras cambiar de mapa base.
- [x] 6.3 Probar los tres gráficos sin filtros, con filtros de proceso/rubro/tratamiento, con capa oculta, con resultados vacíos y tras limpiar filtros.
- [x] 6.4 Ejecutar la regresión de humedales (filtros, contador, cámara, limpieza, mapas base y popups) y confirmar que sigue pasando, sin errores de consola nuevos.
- [x] 6.5 Comprobar el arranque con el bloque SINADER ausente o corrupto: humedales, mapas base, carga local y popups deben seguir operativos con una notificación de error.
- [x] 6.6 Medir el tiempo de arranque y de filtrado con el HTML de ~7,8 MB; documentar el resultado y ajustar la carga diferida si el parseo bloquea la interacción.
- [x] 6.7 Ejecutar `openspec.cmd validate add-secondary-spatial-layer --strict` y corregir cualquier incumplimiento de estructura o escenarios.
