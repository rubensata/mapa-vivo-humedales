# Tasks

## 1. Reconciliación de especificaciones

- [x] 1.1 Archivar `add-secondary-spatial-layer` y verificar que `openspec list --specs` incluye `data-charts` y `sinader-layer`; ejecutar `openspec validate --all --strict` y registrar el resultado.
- [x] 1.2 Actualizar `openspec/specs/data-charts/spec.md` para retirar el requisito de SVG nativo y alinear su propósito con la sección de análisis; verificar que `openspec validate --all --strict` sigue pasando.
- [x] 1.3 Registrar en el cambio la decisión de que `cross-filtered-dashboard` es la única dueña del estado compartido y de las interacciones; verificar que no queda ningún requisito de filtrado duplicado en `data-charts`.

## 2. Estado único y selectores

- [x] 2.1 Implementar el store observable con `getState`, `dispatch`, `subscribe`, `revision` y `lastChange`, y verificar con una prueba de consola que una acción produce una sola notificación.
- [x] 2.2 Implementar la normalización de filtros con las reglas región-comuna, valores vacíos, catálogos y tratamiento por presencia de tonelaje; verificar los casos vacío, inválido, región con comuna incompatible y tratamiento ausente.
- [x] 2.3 Declarar la tabla única de definiciones de filtro y derivar desde ella los predicados JavaScript y las expresiones MapLibre; verificar que mapa, contadores y gráficos devuelven los mismos conteos e identificadores.
- [x] 2.4 Migrar `filterState` y `onFilterChange` al store, conservando los IDs de los selectores y la cascada de comunas; verificar que los cinco selectores siguen comportándose igual y que `Limpiar filtros` restablece todo.
- [x] 2.5 Construir el snapshot por revisión con humedales, SINADER, conteos y agregaciones; adaptar contadores, exportación y snapshot del buffer para consumirlo; verificar que no se vuelve a filtrar por consumidor.
- [x] 2.6 Coalescer las notificaciones en `requestAnimationFrame` y añadir el guard de reingreso; verificar que una ráfaga de cambios renderiza una sola vez y que el renderizado no dispara nuevas acciones.

## 3. Chart.js y gráficos

- [x] 3.1 Implementar el cargador de Chart.js 4.5.1 con SRI, `crossorigin`, timeout y un solo intento; verificar que se solicita una vez y que un fallo simulado no afecta mapa, filtros, exportación ni buffer.
- [x] 3.2 Reemplazar los contenedores de la sección 07 por las tarjetas de los tres gráficos aprobados, conservando `chart-humedales` y `chart-sinader-month` y añadiendo `chart-sinader-rubro`; verificar el orden y la presencia de canvas, resumen y tabla accesible.
- [x] 3.3 Implementar el gráfico de superficie de humedales por región con suma de `ha`, conteo y orden descendente; verificar sus valores contra una agregación independiente del dataset y el estado vacío.
- [x] 3.4 Implementar el gráfico de tonelaje por rubro apilado por tratamiento con top 10; verificar que los segmentos suman el tonelaje de los rubros mostrados, que los colores coinciden con `SINADER_SYMBOLOGY` y que se indica el alcance top 10 de 19.
- [x] 3.5 Implementar el gráfico mensual apilado por tratamiento con los doce meses; verificar que la suma de las doce barras coincide con el tonelaje SINADER filtrado y que el estado vacío aparece sin datos.
- [x] 3.6 Actualizar los gráficos en sitio desde el snapshot, con `update('none')` en ráfagas o movimiento reducido, y destruir instancias solo al reemplazarlas; verificar que no se recrean en cada filtro ni dejan listeners acumulados.

## 4. Interacciones cruzadas

- [x] 4.1 Implementar `applyChartSelection` con metadatos de filtro y despacho atómico; verificar clic en región, clic en rubro, clic en segmento de tratamiento y clic en área vacía.
- [x] 4.2 Implementar leyendas HTML y botones accesibles que reutilicen `applyChartSelection`; verificar que teclado y clic producen el mismo estado y que la leyenda nativa de Chart.js no oculta series.
- [x] 4.3 Implementar el ajuste de cámara por foco (`humedales`, `sinader`, `shared`) y el aviso sin resultados; verificar que una región enfoca humedales, rubro/tratamiento enfoca SINADER y un resultado vacío conserva la vista.
- [x] 4.4 Implementar la alternancia del filtro activo y la sincronización de selectores, contadores y exportación tras cada acción; verificar que repetir la categoría la quita, que la cascada de comunas se respeta y que `Limpiar filtros` devuelve los totales.

## 5. Accesibilidad y composición

- [x] 5.1 Aplicar estilos Swiss compactos a las tres tarjetas y limitar su altura; verificar ausencia de overflow y que el mapa conserva la mayor parte del área interactiva a 1440, 768 y 375 px.
- [x] 5.2 Añadir nombre y resumen accesible al canvas, tabla de datos con botones de filtro, foco rojo visible, áreas táctiles de 44 px y respeto a `prefers-reduced-motion`; verificar navegación completa por teclado y lectura de la alternativa tabular.
- [x] 5.3 Implementar el fallback cuando Chart.js no carga; verificar que muestra las mismas agregaciones en tabla, permite filtrar y mantiene operativos mapa, selectores, contadores, exportación y buffer.

## 6. Integración y verificación

- [x] 6.1 Ampliar la suite CDP para cubrir selector→gráficos y gráfico→mapa en los tres gráficos, filtros compartidos y propios, toggle, limpiar, capa oculta, cambio de mapa base, conteos, exportación y buffer; verificar que no hay errores de consola.
- [x] 6.2 Verificar las solicitudes de red: un solo pedido de Chart.js con SRI, ninguna petición de datasets y ninguna entidad enviada a servidores; simular el fallo del CDN y comprobar el fallback.
- [x] 6.3 Medir el tiempo de filtrado y actualización con 8.906 establecimientos y 107 humedales, registrar resultados y comprobar que no hay crecimiento de listeners ni instancias tras veinte cambios de filtro.
- [x] 6.4 Ejecutar `openspec validate --all --strict`, comprobar que cada requisito de `cross-filtered-dashboard` tiene escenarios y registrar el resultado.
- [x] 6.5 Revisar visualmente escritorio, tablet y móvil, incluyendo panel colapsado, movimiento reducido y fallback; documentar las desviaciones antes de archivar.

## Workflow follow-up

- Archivar el cambio cuando la implementación, la reconciliación de `data-charts` y las verificaciones estén completas.
- Verificar el spec archivado y que `cross-filtered-dashboard` conserva su propósito.
