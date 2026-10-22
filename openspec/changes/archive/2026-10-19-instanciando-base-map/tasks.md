# Tasks

## 1. Instanciación y vista inicial

- [x] 1.1 Revisar que `index.html` conserva el CDN de MapLibre GL JS 4.7.1, el contenedor `#map` existente y el estilo `BASEMAPS.dark`, sin agregar dependencias ni contenedores nuevos. Verificación: revisión estática del `<head>` y del bloque de inicialización; no aparecen nuevas etiquetas de mapa ni API keys.
- [x] 1.2 Ajustar la inicialización de `map` a `center: [-71.0, -35.0]` y `zoom: 4`, manteniendo `style: BASEMAPS.dark.style`. Verificación: abrir `index.html` y comprobar que la vista inicial muestra Chile con zoom 4 y mapa base oscuro.
- [x] 1.3 Conservar sin cambios el manejador `style.load`, la reinyección de capas y los controles de navegación. Verificación: cambiar entre mapas base y confirmar que la capa demo reaparece y los controles siguen operativos.

## 2. Layout y espacio disponible

- [x] 2.1 Verificar que `html`, `body` y `#map` mantienen altura completa y que el canvas ocupa el viewport. Verificación: inspeccionar que no aparecen barras de scroll ni áreas vacías alrededor del mapa.
- [x] 2.2 Probar el redimensionamiento de la ventana y confirmar que el lienzo se adapta sin recortes. Verificación: redimensionar el navegador y observar que el mapa llena el área disponible.

## 3. No regresión de componentes existentes

- [x] 3.1 Verificar que el panel lateral, el selector de mapas base, la lista de capas y el botón de minimizar conservan su estructura y comportamiento. Verificación: minimizar/expandir el panel y cambiar de mapa base; todo responde igual que antes.
- [x] 3.2 Verificar popups y carga local de GeoJSON usando `Humedales_Urbanos.geojson`. Verificación: cargar el archivo y hacer clic en un polígono; se abre el popup con atributos y no hay errores en consola.
- [x] 3.3 Comprobar la consola del navegador sin errores de inicialización ni de red. Verificación: abrir DevTools durante la carga inicial y repetir tras redimensionar y cambiar de mapa base.

## Workflow follow-up

- Revisar y aprobar las specs y tareas antes de implementar.
- Archivar el cambio una vez completada y verificada la implementación.
