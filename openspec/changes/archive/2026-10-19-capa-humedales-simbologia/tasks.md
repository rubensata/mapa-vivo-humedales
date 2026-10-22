# Tasks

## 1. Dataset y carga inicial

- [x] 1.1 Incrustar `Humedales_Urbanos.geojson` minificado en `index.html` dentro de un bloque `<script type="application/json" id="humedales-data">` y verificar con la consola que `JSON.parse` devuelve una `FeatureCollection` con 107 features `MultiPolygon`.
- [x] 1.2 Definir la constante `HUMEDALES_SYMBOLOGY` con las categorías `Municipal` y `de Oficio`, sus colores de relleno/borde y el color neutro de respaldo; verificar que la constante sea la única fuente de colores para estilo y leyenda.
- [x] 1.3 Reemplazar `DEMO_GEOJSON` y su llamada de inicialización por la capa `Humedales Urbanos` marcada como `builtIn`, visible al iniciar y con `popupTemplate: 'humedales'`; verificar que el panel muestra una sola capa activa y que el botón de eliminar no aparece para ella.

## 2. Simbología y persistencia

- [x] 2.1 Extender `addGeoJSONLayer` y `reapplyAllLayers` para aceptar opciones de pintura por capa y aplicar expresiones `match` sobre `proceso_humedal_urbano`; verificar que los polígonos `Municipal` usan cian y los `de Oficio` ámbar, con borde diferenciado.
- [x] 2.2 Verificar que la simbología categórica se conserva tras cambiar entre OSM Dark, OSM Bright, OSM Liberty y Satelital, sin recargar la página ni perder la capa.
- [x] 2.3 Verificar que la capa de humedales se oculta y muestra con el control de visibilidad del panel y que su estado se mantiene al cambiar de mapa base.

## 3. Leyenda y popup

- [x] 3.1 Agregar la sección de leyenda `Simbología` al panel lateral con las etiquetas `Municipal` y `de Oficio`; verificar que las muestras de color coinciden exactamente con el relleno y borde del mapa.
- [x] 3.2 Implementar la plantilla de popup de humedales con nombre, código, región, comuna, proceso, hectáreas formateadas, resolución y enlaces `url_res_bcn`/`url_simbio`; verificar que los enlaces se abren en pestaña nueva con `rel="noopener noreferrer"` y que textos/URLs inválidas no generan HTML ejecutable.
- [x] 3.3 Verificar que un clic sobre un humedal abre un único popup y que tras cambiar de mapa base varias veces sigue abriéndose uno solo, sin manejadores duplicados.

## 4. Integración y regresión

- [x] 4.1 Verificar que la cámara inicial sigue centrada en Chile con zoom 4, que el panel y la lista de capas mantienen su estructura y que la consola no muestra errores de inicialización ni de red.
- [x] 4.2 Verificar que la carga local de un GeoJSON distinto sigue funcionando con selector y arrastre, incluyendo `fitBounds`, paleta de color, popup genérico y notificaciones de error.
- [x] 4.3 Verificar el saneamiento de los datos: nombres y URLs con espacios finales se muestran recortados, y una categoría distinta de `Municipal`/`de Oficio` recibe el color neutro de respaldo.

## Workflow follow-up

- Sincronizar las specs principales al archivar el cambio.
- Archivar el cambio una vez completada la verificación en navegador.
