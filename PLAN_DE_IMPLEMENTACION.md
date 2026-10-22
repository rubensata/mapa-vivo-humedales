# Plan Consolidado de Arquitectura e Implementación
## Visor Web Geoespacial Zero-Server (KISS)

---

## 1. Resumen Ejecutivo y Enfoque

El objetivo de este proyecto es proveer un visor web cartográfico autónomo, ligero y funcional basado en el principio **KISS (Keep It Simple, Stupid)**. La solución adopta un modelo **Zero-Server**, lo que significa que no requiere servidores de backend, APIs intermedias, bases de datos ni herramientas complejas de compilación (como Webpack, Vite o Node.js). Toda la aplicación reside en un único archivo [index.html](file:///c:/app_ia/curso/index.html).

### Características Principales
- **Zero-Server**: Ejecución 100% del lado del cliente en el navegador.
- **Archivo Único**: HTML5, estilos y lógica modular encapsulados en [index.html](file:///c:/app_ia/curso/index.html).
- **Mapa Base Abierto**: Mosaicos estándar de OpenStreetMap vía MapLibre GL JS sin requerir API keys.
- **Dataset de Demostración**: Hitos mundiales con metadatos completos integrados de fábrica.
- **Carga de Archivos Locales**: Soporte para archivos `.geojson` y `.json` mediante selector de archivos o arrastrar y soltar (Drag & Drop) usando `FileReader API`.
- **Interactividad Geoespacial**: Popups dinámicos para inspección de atributos, alternancia de visibilidad de capas, auto-zoom a la extensión de geometrías (`fitBounds`) y notificaciones flotantes (Toast).

---

## 2. Diagrama de Arquitectura de la Solución

```text
+-----------------------------------------------------------------------------------+
| NAVEGADOR WEB DEL CLIENTE (index.html)                                            |
|                                                                                   |
|  +-------------------------------------+   +------------------------------------+ |
|  |           CAPA VISUAL / UI          |   |          MOTOR CARTOGRÁFICO        | |
|  | - Tailwind CSS CDN                  |   | - MapLibre GL JS (WebGL Canvas)    | |
|  | - Header & Status Badge             |   | - Fuente Raster: OpenStreetMap XYZ | |
|  | - Panel Flotante Glassmorphism      |<->| - Capas Vectoriales (Fill/Line/    | |
|  | - Zona Drag & Drop interactiva      |   |   Circle para GeoJSON)             | |
|  | - Sistema de Notificaciones Toast   |   | - Popups interactivos (HTML)       | |
|  +-------------------------------------+   +------------------------------------+ |
|                     ^                                         ^                   |
|                     |                                         |                   |
|                     +--------------------+--------------------+                   |
|                                          |                                        |
|                                          v                                        |
|  +------------------------------------------------------------------------------+ |
|  |                         LÓGICA EN VANILLA JS (ES6+)                          | |
|  | - Inicialización de controles (Zoom, Brújula, Geolocalización, Escala)       | |
|  | - FileReader API nativo (procesamiento de GeoJSON local en cliente)          | |
|  | - Gestor de estado en memoria (registro de capas, visibilidad, paleta)       | |
|  | - Bounding Box Engine (cálculo de límites geográficos y ajuste fitBounds)    | |
|  +------------------------------------------------------------------------------+ |
+-----------------------------------------------------------------------------------+
                                           |
                         Peticiones de Mosaicos Raster (HTTP GET)
                                           v
+-----------------------------------------------------------------------------------+
| PROVEEDOR PÚBLICO EXTERNO (100% Libre / Sin API Key / Sin marcas de agua)         |
| - Servidores OpenFreeMap (Vector tiles OSM: Bright, Liberty, Positron) & ESRI Sat |
+-----------------------------------------------------------------------------------+
```

---

## 3. Matriz de Decisiones Técnicas y Riesgos

| Dimensión | Decisión Adoptada | Justificación (Principio KISS) | Alternativas Descartadas |
| :--- | :--- | :--- | :--- |
| **Infraestructura** | Zero-Server (Single-file) | Despliegue inmediato, portable con doble clic y costo cero de mantenimiento. | Backend Node.js / Python / Docker. |
| **Motor de Mapas** | MapLibre GL JS (v4.7.1) | Estándar de código abierto, renderizado fluido por WebGL y soporte nativo de estilos vectoriales. | Leaflet (no usa WebGL por defecto) o Mapbox GL (requiere API key con cuotas). |
| **Mapa Base** | OpenFreeMap (Bright, Liberty, Positron) + ESRI Satelital | Diseñado específicamente para MapLibre GL JS. Evita el bloqueo HTTP 403 de `tile.openstreetmap.org` y la marca de agua `API KEY REQUIRED` de CARTO. Es 100% libre y sin tokens. | CARTO (exige API Key) y OSM voluntario (bloqueo 403 en `file://`). |
| **Estilos CSS** | Tailwind CSS (Play CDN) | Provee utilidades modernas (glassmorphism, animaciones, layout responsivo) sin bundler. | CSS monolítico escrito a mano o preprocesadores Sass/PostCSS. |
| **Carga de Datos** | Web FileReader API | Permite cargar datos locales sin salir del navegador del usuario ni violar políticas CORS. | Subida a servidor remoto o almacenamiento en nube S3. |

---

## 4. Plan de Implementación por Fases

A continuación se detalla la hoja de ruta que se ejecutó para construir la aplicación:

### Fase 1: Estructura Base y Dependencias
- [x] **1.1 Estructura HTML5 y CDNs**:
  - Declaración del estándar `<!DOCTYPE html>` y metadatos responsivos.
  - Inclusión de las bibliotecas oficiales por CDN: Tailwind CSS, MapLibre GL JS (CSS y JS).
  - *Criterio de éxito*: El documento carga en el navegador sin errores en la consola web.
- [x] **1.2 Layout Responsivo y Canvas**:
  - Contenedor `#map` ocupando el 100% de alto y ancho del viewport (`absolute inset-0`).
  - Maquetación de componentes flotantes con estilos `backdrop-blur` y bordes sutiles.
  - *Criterio de éxito*: El mapa ocupa toda la pantalla y los paneles no bloquean la navegación.

### Fase 2: Configuración del Mapa Base y Controles
- [x] **2.1 Mapa Base OpenStreetMap**:
  - Definición del objeto de estilo MapLibre con tiles en `https://tile.openstreetmap.org/{z}/{x}/{y}.png`.
  - Configuración de atribución oficial para respetar los términos de OSM.
  - Centrado y zoom inicial predeterminado.
  - *Criterio de éxito*: Los mosaicos cartográficos se visualizan con fluidez al desplazarse.
- [x] **2.2 Controles de Navegación Esenciales**:
  - `NavigationControl`: Botones de acercar/alejar, inclinación y brújula para resetear el norte.
  - `GeolocateControl`: Ubicación actual del dispositivo con marcador animado.
  - `ScaleControl`: Escala métrica gráfica dinámica en la esquina inferior izquierda.
  - *Criterio de éxito*: Todos los botones responden a clics y reflejan cambios en el mapa.

### Fase 3: Gestión de Capas y Dataset Demostrativo
- [x] **3.1 Dataset GeoJSON de Demostración**:
  - Incorporación de una colección de hitos internacionales (Obelisco, Torre Eiffel, Coliseo, Estatua de la Libertad, etc.) con propiedades enriquecidas.
  - Estilización de marcadores vectoriales circulares con borde blanco y halo de selección.
  - *Criterio de éxito*: Los puntos de interés se dibujan en sus coordenadas exactas al inicio.
- [x] **3.2 Panel Gestor de Capas**:
  - Panel lateral con conteo dinámico de capas activas.
  - Interruptores/checkboxes interactivos para alternar la propiedad `visibility` (`visible` / `none`) en tiempo real.
  - Opción de minimizar o expandir el panel lateral para mayor visibilidad del mapa.
  - *Criterio de éxito*: Ocultar y mostrar capas de forma instantánea sin parpadeos ni recargas.

### Fase 4: Carga de Datos del Usuario e Interactividad
- [x] **4.1 Ingesta Local vía FileReader y Drag & Drop**:
  - Escucha de eventos `dragover`, `dragleave` y `drop` sobre la ventana para activar una máscara visual de arrastre.
  - Selector de archivo nativo `<input type="file" accept=".geojson,.json">`.
  - Soporte automático para geometrías `Point`, `LineString`, `Polygon` y sus variantes `Multi*`.
  - Cálculo de `LngLatBounds` y ajuste de cámara (`map.fitBounds`) a los datos importados.
  - *Criterio de éxito*: Arrastrar un archivo GeoJSON dibuja inmediatamente la geometría y enfoca el mapa sobre ella.
- [x] **4.2 Popups de Inspección de Atributos**:
  - Eventos de clic sobre geometrías para abrir `maplibregl.Popup`.
  - Transformación automática de las claves y valores de `properties` en una tabla HTML estructurada y legible.
  - Cambio dinámico de cursor a `pointer` al pasar el puntero sobre entidades con datos.
  - *Criterio de éxito*: Clic sobre cualquier punto o área muestra sus metadatos formateados.
- [x] **4.3 Sistema de Notificaciones y Feedback de Errores**:
  - Toasts flotantes con temporizador de descarte automático para confirmaciones de carga y alertas si el JSON es defectuoso.
  - *Criterio de éxito*: Cargar un archivo corrupto muestra una advertencia clara sin romper la aplicación.

---

## 5. Guía de Uso y Verificación

1. **Abrir la aplicación**:
   - Haz doble clic sobre el archivo [index.html](file:///c:/app_ia/curso/index.html) para abrirlo en cualquier navegador web moderno (Google Chrome, Mozilla Firefox, Microsoft Edge, Safari).
2. **Interactuar con el mapa**:
   - Usa la rueda del ratón o los botones en pantalla para hacer zoom.
   - Mantén presionado el botón derecho o la tecla `Shift` mientras arrastras para rotar e inclinar la vista tridimensionalmente.
   - Haz clic sobre cualquiera de los puntos demo para abrir el popup con sus detalles.
3. **Cargar tus propios datos**:
   - Descarga o genera cualquier archivo con extensión `.geojson`.
   - Arrástralo directamente sobre el navegador o pulsa en **"Seleccionar archivo GeoJSON"** en el panel lateral.
   - La capa se sumará al gestor con un color distintivo y el mapa volará a su ubicación automáticamente.
