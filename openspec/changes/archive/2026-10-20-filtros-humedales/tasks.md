# Tasks

## 1. Interfaz de filtros

- [x] 1.1 Agregar al panel lateral la sección `Filtros` con los selectores `Región`, `Comuna`, `Proceso`, el contador de resultados y el botón `Limpiar filtros`; verificar en el navegador que la sección aparece entre el mapa base y la carga local, con etiquetas accesibles y sin alterar el resto del panel.
- [x] 1.2 Construir las opciones de región, comuna y proceso desde `HUMEDALES_DATA`, ordenadas con `localeCompare('es')` y sin duplicados; verificar que región ofrece 15 valores, proceso 2 y el total de comunas es 65.
- [x] 1.3 Implementar el encadenamiento región → comuna: al cambiar la región, el selector de comuna se limita a esa región, descarta una comuna incompatible y vuelve a todas las comunas al elegir `Todas`; verificar con regiones como Ñuble, Aysén y O'Higgins.

## 2. Filtrado de capa y cámara

- [x] 2.1 Agregar el estado de filtros y traducirlo a una expresión `all` sobre `region`, `comuna` y `proceso_humedal_urbano`; aplicarla a las subcapas de humedales y componerla con el filtro geométrico existente en `reapplyAllLayers()`; verificar en la consola que la capa recibe el filtro y que una capa cargada por el usuario no lo recibe.
- [x] 2.2 Actualizar el contador con la cantidad de humedales coincidentes; verificar que sin filtros muestra 107 y que coincide con las features visibles para una región y un proceso concretos.
- [x] 2.3 Ajustar la cámara con `fitBounds` a la extensión filtrada cada vez que cambie un filtro; verificar que el centro y zoom cambian hacia la zona seleccionada y que una combinación sin resultados conserva la vista y muestra un toast de advertencia.
- [x] 2.4 Implementar `Limpiar filtros` para restablecer los tres selectores, el contador y la vista inicial de Chile con zoom 4; verificar que el mapa vuelve al centro `[-71, -35]` y que reaparecen los 107 humedales.
- [x] 2.5 Conservar el filtro al cambiar de mapa base y al ocultar/mostrar la capa; verificar que tras alternar OSM Dark, Bright, Liberty y Satelital el subconjunto filtrado y el contador se mantienen, y que al ocultar y mostrar la capa reaparece filtrada.

## 3. Integración y regresión

- [x] 3.1 Verificar que los filtros no alteran la simbología categórica, la leyenda ni el popup de humedales, y que el popup solo responde a las geometrías visibles tras filtrar.
- [x] 3.2 Verificar que la carga local de GeoJSON, el ajuste a su extensión, la paleta de color y el popup genérico siguen funcionando con filtros activos.
- [x] 3.3 Verificar en navegador que no hay errores de consola tras aplicar filtros, cambiar de mapa base, limpiar filtros y cargar un archivo local.

## Workflow follow-up

- Sincronizar las specs principales al archivar el cambio.
- Archivar el cambio una vez completada la verificación en navegador.
