# Proposal

## Why

El visor ya filtra humedales y establecimientos SINADER, pero sus tres gráficos SVG solo leen el resultado de los selectores: no permiten explorar los datos desde el gráfico ni garantizan que mapa, contadores, exportación y gráficos compartan una única fuente de verdad. El usuario debe volver a los selectores para cada ajuste y la lógica de filtrado se repite entre la vista y los consumidores. Un dashboard cruzado con Chart.js resuelve esa fricción sin backend y convierte el análisis en una herramienta de exploración, manteniendo el mapa como elemento principal.

## What Changes

- Sustituir los tres gráficos SVG actuales por tres gráficos Chart.js aprobados:
  - **Superficie de humedales por región**: suma de `ha` y conteo de humedales por región; barras horizontales; clic en una región aplica el filtro compartido de región y acerca el mapa a los humedales.
  - **Tonelaje SINADER por rubro y tratamiento**: suma de toneladas por rubro, apilada por tratamiento (top 10 rubros); clic en una barra aplica rubro y clic en un segmento aplica rubro + tratamiento.
  - **Perfil mensual SINADER por tratamiento**: suma de los doce valores `meses[]` por mes, apilada por tratamiento; clic en una categoría o su leyenda aplica el filtro de tratamiento y acerca el mapa a SINADER.
- Crear un estado único de filtros para `region`, `comuna`, `proceso`, `rubro` y `tratamiento`. Los selectores, los gráficos, el mapa, los contadores, la exportación y el snapshot del buffer derivarán del mismo estado y de los mismos selectores de datos.
- Hacer bidireccional la interacción: los filtros del visor actualizan los gráficos y las acciones de los gráficos actualizan los selectores, el mapa y los demás consumidores.
- Evitar la lógica duplicada: una sola función de coincidencia por capa, un solo ciclo de actualización y una sola derivación de datos filtrados por revisión de estado.
- Cargar Chart.js 4.5.1 desde CDN con integridad SRI y carga no bloqueante. Si la librería no está disponible, mostrar una alternativa accesible con tabla y controles, sin afectar mapa, filtros, exportación ni buffer.
- **BREAKING** a nivel de especificación: se retira el requisito de gráficos SVG nativos de la capacidad `data-charts` del cambio aún no archivado `add-secondary-spatial-layer`. Esa capacidad debe reconciliarse antes de archivar este cambio.
- Mantener el modelo Zero-Server: sin backend, sin envío de datos y sin duplicar datasets. El mapa sigue siendo el elemento visual principal y los gráficos permanecen compactos dentro del panel.

## Capabilities

### New Capabilities

- `cross-filtered-dashboard`: estado único de filtros, gráficos Chart.js, interacciones gráfico-mapa y sincronización reactiva entre selectores, mapa, contadores, exportación y análisis de buffer.

### Modified Capabilities

<!-- Sin deltas sobre capacidades de `openspec/specs`: el cambio de renderizado de gráficos pertenece a `data-charts`, que aún vive en `add-secondary-spatial-layer` y debe archivarse/sincronizarse antes de aplicar este cambio. -->

## Impact

- `index.html`: sección 07 de análisis, estilos de gráficos, carga de Chart.js, estado de filtros, selectores, mapa, contadores, exportación y snapshot del buffer.
- Dependencia nueva: `chart.js@4.5.1` (UMD minificado, 208.522 bytes) desde `cdn.jsdelivr.net`, con SRI `sha384-jb8JQMbMoBUzgWatfe6COACi2ljcDdZQ2OxczGA3bGNeWe+6DChMTBJemed7ZnvJ`.
- Sin backend ni subida de datos. Solo se solicita el archivo de Chart.js; los datasets siguen embebidos.
- Sin cambios en mapas base, popups, capas locales, simbología, exportación de datos ni análisis de buffer, salvo que todos pasan a consumir los selectores centralizados.
- Prerrequisito: archivar `add-secondary-spatial-layer` y reconciliar `data-charts` antes de implementar este cambio, para que no coexistan los requisitos de SVG nativo y Chart.js.
- Verificación: interacciones en ambos sentidos, teclado, estados vacíos, fallback sin Chart.js, responsive a 375/768/1440 px, ausencia de bucles y solicitudes de red inesperadas.
