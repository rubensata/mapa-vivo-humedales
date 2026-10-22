# Proposal

## Why

Existe la necesidad de disponer de un visor web geoespacial ligero, autónomo y sin dependencias de infraestructura backend (Zero-Server), que permita visualizar mapas base estándar de OpenStreetMap e interactuar con capas de datos GeoJSON (tanto predefinidas como proporcionadas por el usuario). Este enfoque permite desplegar o compartir herramientas de análisis espacial instantáneamente desde un simple archivo HTML, siguiendo el principio KISS.

## What Changes

- Creación de un visor web geoespacial autónomo en un único archivo `index.html`.
- Integración de MapLibre GL JS y Tailwind CSS a través de CDN sin herramientas de compilación o backend.
- Configuración de mapa base raster OpenStreetMap estándar sin requerir claves de API.
- Incorporación de controles de navegación, geolocalización y escala.
- Capacidad de cargar y alternar una capa GeoJSON de demostración integrada.
- Capacidad para que el usuario cargue archivos GeoJSON locales vía arrastrar y soltar (drag & drop) o selector de archivos.
- Inspección interactiva de atributos mediante popups al hacer clic sobre geometrías (puntos, líneas, polígonos).

## Capabilities

### New Capabilities
- `map-viewer`: Renderizado de mapa base estándar OpenStreetMap mediante MapLibre GL JS con navegación y controles esenciales.
- `layer-management`: Carga, alternancia de visibilidad e inspección interactiva mediante popups de capas GeoJSON (predefinidas y locales).

### Modified Capabilities
<!-- Ninguna capacidad previa existente en el repositorio -->

## Impact

- Código nuevo: `index.html` en la raíz del proyecto.
- Dependencias: MapLibre GL JS (vía CDN) y Tailwind CSS (vía CDN Play).
- Red: Peticiones directas desde el navegador del cliente hacia los servidores de mosaicos de OpenStreetMap (`tile.openstreetmap.org`).
