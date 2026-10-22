# Tasks

## 1. Estructura base del archivo único y dependencias

- [x] 1.1 Crear el archivo `index.html` con la estructura HTML5 semántica y los enlaces CDN oficiales para MapLibre GL JS (CSS y JS) y Tailwind CSS. Verificar visualizando que el documento carga sin errores en la consola.
- [x] 1.2 Implementar el contenedor principal `#map` a pantalla completa y el layout de paneles flotantes semitransparentes con Tailwind CSS. Verificar que la distribución sea responsiva y no bloquee el mapa.

## 2. Inicialización del mapa base con OpenStreetMap

- [x] 2.1 Configurar e inicializar la instancia de MapLibre GL JS utilizando la fuente raster XYZ de OpenStreetMap (`https://tile.openstreetmap.org/{z}/{x}/{y}.png`) con atribución completa. Verificar que los mosaicos de OSM se rendericen correctamente al cargar la página.
- [x] 2.2 Agregar los controles de navegación (NavigationControl con zoom y rotación), geolocalización (GeolocateControl) y escala gráfica (ScaleControl). Verificar que cada control responda adecuadamente a las interacciones del usuario.

## 3. Gestión de capas GeoJSON y datos de demostración

- [x] 3.1 Integrar un dataset GeoJSON de demostración (puntos de interés) y agregar la capa vectorial estilizada al mapa. Verificar que los puntos aparezcan con estilo distintivo.
- [x] 3.2 Crear el panel flotante de control de capas con casillas de verificación para alternar la visibilidad (`setLayoutProperty('visibility')`). Verificar que al desmarcar la casilla la capa se oculte y al marcarla vuelva a mostrarse.

## 4. Carga de datos locales e interactividad

- [x] 4.1 Implementar la lectura de archivos GeoJSON locales mediante selector de archivos `<input type="file">` y eventos Drag & Drop en el viewport usando la API `FileReader`. Verificar cargando un archivo `.geojson` de prueba y comprobando que se dibuje en el mapa con `fitBounds`.
- [x] 4.2 Configurar eventos de interacción del cursor y popups (`maplibregl.Popup`) al hacer clic en las geometrías de las capas activas, mostrando una tabla estilizada con sus propiedades y atributos. Verificar haciendo clic en geometrías y validando la información mostrada.
- [x] 4.3 Añadir manejo de errores visuales (notificación flotante/toast) para archivos JSON inválidos o problemas de renderizado. Verificar arrastrando un archivo de texto plano no válido.
