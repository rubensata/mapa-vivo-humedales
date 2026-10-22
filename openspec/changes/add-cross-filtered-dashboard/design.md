# Design

## Context

Ver `proposal.md` para la motivación. El visor es un único `index.html` Zero-Server con MapLibre GL JS, datos embebidos (107 humedales y 8.906 establecimientos SINADER) y un panel lateral de 370 px que se apila a ancho completo en móvil. La sección 07 ya contiene tres gráficos SVG.

Hoy el estado de filtros vive en un objeto mutable `filterState`; cada selector lo actualiza en `onFilterChange` y llama a `applyFilters`, que aplica expresiones a MapLibre, actualiza contadores y ejecuta `renderCharts`. Los gráficos SVG, la exportación y el snapshot del buffer leen `getFilteredHumedalesFeatures()` y `getFilteredSinaderFeatures()`. No existe interacción gráfico → mapa y la lógica de filtrado se reparte entre predicados JavaScript y constructores de expresiones MapLibre, con riesgo de divergencia.

La capacidad `data-charts` del cambio `add-secondary-spatial-layer` exige SVG nativo y aún no está archivada. Este diseño la sustituye por Chart.js; la reconciliación de esa spec es un prerrequisito de implementación, no una decisión implícita.

## Goals / Non-Goals

**Goals:**

- Un único estado canónico de filtros y un único ciclo de actualización.
- Una sola definición de criterios de filtrado compartida por mapa, gráficos, contadores, exportación y buffer.
- Interacciones bidireccionales sin bucles ni estados duplicados.
- Tres gráficos Chart.js compactos, accesibles y tolerantes al fallo del CDN.
- Mantener el mapa como elemento principal y conservar IDs y comportamientos existentes.

**Non-Goals:**

- No añadir backend, datasets, filtros temporales ni nuevas capas.
- No rediseñar popups, simbología, mapas base ni capas locales.
- No implementar selección de establecimientos individuales desde los gráficos.
- No sustituir MapLibre ni el renderizado del mapa.
- No medicalizar ni normalizar los valores; se muestran toneladas y hectáreas absolutas.

## Decisions

### 1. Store observable mínimo en lugar de una librería de estado

Se implementará un store propio con cierre sobre el estado:

```text
dispatch(patch, meta) -> normalizar -> revision + 1 -> snapshot -> notificar
```

Expone `getState()`, `dispatch()`, `subscribe()` y una `revision` monotónica. El estado contiene solo los filtros y metadatos de la última acción; los datos filtrados y las agregaciones son derivados y no se persisten. Se descarta Redux/MobX por peso y complejidad para un archivo único, y un event bus global porque dificulta rastrear el origen y el orden de las actualizaciones.

### 2. Forma del estado y normalización determinista

```text
filters: { region, comuna, proceso, rubro, tratamiento }
revision: number
lastChange: { source, focus, keys }
```

`source` identifica `selector`, `chart`, `clear` o `restore`; `focus` identifica `humedales`, `sinader` o `shared`. La normalización recorta espacios, descarta claves desconocidas, valida contra los catálogos y aplica estas reglas:

- `region` vacía significa «Todas»; al cambiar de región se limpia `comuna` si no pertenece.
- `comuna` sin región se conserva solo si el catálogo la reconoce; si se conoce su región, se completa `region` de forma determinista.
- `proceso`, `rubro` y `tratamiento` se validan contra sus catálogos; un valor ausente vuelve a «Todos».
- El tratamiento se compara por presencia de tonelaje en su campo (`> 0`), no por el tratamiento dominante.
- Si SINADER aún no está cargado, los filtros propios se aceptan pero no producen datos hasta que el catálogo esté disponible; al cargarse se recalcula el snapshot.

La normalización es pura y se prueba con entradas vacías, valores inválidos y combinaciones región-comuna contradictorias.

### 3. Un solo ciclo de actualización y snapshot compartido

`dispatch` normaliza, incrementa `revision` y calcula una sola vez:

```text
snapshot = {
  state,
  humedales: features filtradas,
  sinader: features filtradas,
  counts: { humedales, sinader },
  charts: { regiones, rubros, mensual }
}
```

Luego notifica en orden fijo: sincronización de selectores, expresiones MapLibre, cámara si corresponde, contadores/exportación y gráficos. Cada consumidor recibe el snapshot en lugar de volver a filtrar. Los renders son funciones de solo lectura: no llaman a `dispatch`.

Para ráfagas de cambios se coalesce la notificación en un `requestAnimationFrame`; si llega otro cambio antes del frame, solo se renderiza el último snapshot. Un guard `isDispatching` impide que un render o una actualización de Chart.js reingrese al store. Chart.js se actualiza con `chart.update('none')` cuando el usuario prefiere movimiento reducido o durante ráfagas.

### 4. Una definición de filtros para todos los consumidores

Se declara una tabla de definiciones con campo, capas afectadas y tipo de coincidencia:

| Filtro | Campo | Capas | Coincidencia |
| --- | --- | --- | --- |
| region | `region` | humedales, SINADER | igualdad recortada |
| comuna | `comuna` | humedales, SINADER | igualdad recortada |
| proceso | `proceso_humedal_urbano` | humedales | igualdad recortada |
| rubro | `rubro` | SINADER | igualdad recortada |
| tratamiento | campo `t_*` de la categoría | SINADER | tonelaje `> 0` |

Los predicados JavaScript y los constructores de expresiones MapLibre se generan desde esta misma tabla. `getFilteredHumedalesFeatures()` y `getFilteredSinaderFeatures()` pasan a devolver el subconjunto del snapshot. Así, contadores, exportación, buffer y gráficos consumen exactamente los mismos datos que el mapa.

Para verificar equivalencia se comparan conteos e identificadores entre el snapshot y `queryRenderedFeatures`/filtros de MapLibre en la suite CDP.

### 5. Interacciones de gráficos como acciones del store

Cada elemento de gráfico declara metadatos de filtro, no lógica de mapa. Una única función `applyChartSelection({ filterKey, value, secondaryKey, secondaryValue })` construye un patch atómico y llama a `dispatch` con `source: 'chart'` y el foco adecuado:

- Región de humedales: `{ region: value }`, limpia `comuna`, foco `humedales`.
- Rubro: `{ rubro: value }`, foco `sinader`.
- Segmento de tratamiento: `{ rubro, tratamiento }` si viene de rubros; `{ tratamiento }` si viene del mensual; foco `sinader`.
- Si el valor ya está activo, la acción lo alterna a vacío («Todos»).
- Un clic en área vacía no despacha nada.

La cámara se ajusta solo cuando `lastChange` pide foco y hay resultados; con cero resultados se conserva la vista y se muestra una advertencia. `Limpiar filtros` despacha un patch con los cinco valores vacíos y foco de vista inicial.

### 6. Gráficos y agregaciones

Los tres gráficos se construyen desde el snapshot, nunca desde el DOM ni desde MapLibre:

1. **Humedales por región**: suma de `ha` y conteo por región, orden descendente; todas las regiones con datos (15). Barra horizontal de una serie, tinta Swiss y rojo para la región activa.
2. **SINADER por rubro y tratamiento**: top 10 de rubros por tonelaje, apilado por los campos `t_eliminacion`, `t_recepcion` y `t_valorizacion`. Se indica «Top 10 de 19 rubros»; no se agrega una barra «Otros» porque no sería accionable. Colores desde `SINADER_SYMBOLOGY`.
3. **SINADER mensual por tratamiento**: los doce valores de `meses[]` sumados por mes y apilados por tratamiento. No se añade un filtro de mes porque el mapa no tiene dimensión temporal; solo los segmentos/leyenda de tratamiento despachan filtros.

Los tooltips y etiquetas usan formato `es-CL`; las barras muestran valores compactos y las tablas accesibles muestran el valor completo.

### 7. Carga de Chart.js y fallback

Se carga `chart.js@4.5.1/dist/chart.umd.min.js` desde `cdn.jsdelivr.net` con `integrity="sha384-jb8JQMbMoBUzgWatfe6COACi2ljcDdZQ2OxczGA3bGNeWe+6DChMTBJemed7ZnvJ"`, `crossorigin="anonymous"` y `referrerpolicy="no-referrer"`. La carga se dispara después del primer render (idle callback con timeout de respaldo), una sola vez, y no bloquea filtros ni mapa.

Si falla o excede 8 s, se registra el fallo, no se reintenta en cada cambio de filtro y se renderiza la alternativa accesible: tablas con las mismas agregaciones y botones de filtro equivalentes. Mapa, selectores, contadores, exportación y buffer no dependen de Chart.js.

Las instancias se guardan en un registro; se actualizan sus datos en sitio y se destruyen solo al reemplazar un gráfico o al descargar la página. Cambiar la visibilidad de una capa no reconstruye los gráficos.

### 8. Accesibilidad y composición

Cada tarjeta incluye título, resumen textual, canvas con `role="img"` referenciado con `aria-labelledby` y `aria-describedby`, y una tabla de datos con botones enfocables que aplican las mismas categorías. Las leyendas son HTML con botones; se desactiva la leyenda interactiva nativa de Chart.js para que ocultar una serie no sustituya al filtro. Se mantienen foco rojo visible, áreas táctiles de 44 px y contraste; no se depende solo del color.

Los gráficos conservan la sección 07 y viven en tarjetas compactas (canvas de 165–200 px más leyenda, resumen y tabla plegable) dentro del panel. En móvil el panel mantiene su altura máxima y su scroll; el mapa sigue visible y utilizable al colapsar el panel. Se respeta `prefers-reduced-motion`.

Se conservan los IDs existentes `chart-humedales` y `chart-sinader-month`; el gráfico de rubros usa `chart-sinader-rubro` y se retira el contenedor `chart-sinader-treatment`, que queda cubierto por los segmentos del gráfico de rubros. Cualquier prueba que dependa del gráfico de tratamiento se actualiza.

### 9. Reconciliación de especificaciones

Antes de implementar se archiva `add-secondary-spatial-layer` y se sincronizan sus specs. Después se retira de `data-charts` el requisito de SVG nativo y se ajusta su propósito para que describa la sección de análisis sin imponer la tecnología de renderizado. La nueva capacidad `cross-filtered-dashboard` es la dueña del estado compartido y de las interacciones; no se duplica esa responsabilidad en `data-charts`. Esta reconciliación se registra en las tareas y se valida antes de archivar este cambio.

## Risks / Trade-offs

- **CDN de Chart.js no disponible** → fallback tabular accesible; mapa y filtros nunca dependen de la librería.
- **Accesibilidad de canvas** → tabla y botones equivalentes, resumen textual, foco visible y pruebas de teclado; no se confía solo en el clic.
- **Bucles de actualización** → flujo unidireccional, snapshot por revisión, guard de reingreso y coalescing; pruebas de una sola actualización por acción.
- **Divergencia entre predicados y expresiones MapLibre** → tabla única de definiciones y verificación de equivalencia por conteos/IDs.
- **Costo de filtrar 8.906 entidades en cada interacción** → una sola pasada por revisión, snapshot compartido y actualización en sitio de Chart.js; medir con dataset completo.
- **Gráficos que desplazan al mapa en móvil** → tarjetas compactas, panel colapsable y altura máxima; verificar 375/768 px.
- **Top 10 de rubros oculta categorías** → etiqueta explícita del alcance; no se ofrece una categoría «Otros» que no pueda filtrarse.
- **Conflicto con `data-charts` no archivada** → prerrequisito de archivo/sincronización y decisión registrada; no se implementa sobre specs contradictorias.
- **Cambios de filtro durante la carga de SINADER** → el store conserva la intención, el snapshot se recalcula al registrarse la capa y los gráficos muestran el estado correspondiente.

## Migration Plan

1. Archivar `add-secondary-spatial-layer` y reconciliar `data-charts` (retirar SVG nativo y actualizar propósito).
2. Introducir el store, la normalización y la tabla de filtros, manteniendo temporalmente los nombres y salidas actuales para no romper selectores, exportación ni buffer.
3. Migrar contadores, exportación y snapshot del buffer al snapshot compartido; ejecutar regresión de filtros existentes.
4. Cargar Chart.js y reemplazar los renderizadores SVG por los tres gráficos aprobados, manteniendo el fallback.
5. Conectar clics, leyendas y controles accesibles al store; verificar que no haya doble actualización.
6. Eliminar helpers y estilos SVG sin uso, y actualizar las pruebas del cambio anterior que consulten el gráfico de tratamiento.
7. Verificar con CDP, medición de rendimiento y revisión visual; el rollback consiste en revertir `index.html` y los artefactos de este cambio, sin migración de datos.

## Open Questions

- Confirmar si en el futuro se desea un filtro temporal por mes; no forma parte de este cambio porque el mapa no representa esa dimensión. Las tres visualizaciones definidas en este diseño son la base del cambio y cualquier sustitución requiere actualizar la spec y las tareas.
