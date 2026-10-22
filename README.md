# Mapa Vivo de Humedales

Visor geoespacial de **humedales urbanos** y **establecimientos industriales SINADER 2024** en Chile. Funciona como una aplicación web de una sola página, sin servidor propio ni base de datos: los datos viajan incorporados en el propio visor.

## Qué muestra

- **107 humedales urbanos** distribuidos en 15 regiones, con superficie y estado.
- **8.906 establecimientos SINADER** de 16 regiones, agrupados en 19 rubros y tres tratamientos.
- Filtros compartidos por región, comuna, proceso, rubro y tratamiento.
- Tres gráficos interactivos: superficie de humedales por región, tonelaje por rubro y evolución mensual del tonelaje.
- Análisis de buffer entre 1 y 50.000 metros.
- Exportación de los resultados filtrados en GeoJSON y Shapefile.

## Cómo usarlo

1. Abre `index.html` en un navegador moderno (Chrome, Edge o Firefox).
2. Usa el panel de control para filtrar por región, comuna, proceso, rubro o tratamiento.
3. Haz clic en las barras de los gráficos para aplicar ese filtro al mapa.
4. Activa las capas y el análisis de buffer desde el panel.

## Demo

Puedes ver el visor publicado en:

https://rubensata.github.io/mapa-vivo-humedales/

> El visor necesita conexión a internet para cargar el mapa base y la librería de gráficos desde sus CDN.

## Tecnología

- HTML, CSS y JavaScript en un único archivo.
- MapLibre GL JS 4.7.1 y OpenFreeMap para el mapa.
- Chart.js 4.5.1 para los gráficos.
- Turf.js para el cálculo de buffers.
- Sin backend, sin proceso de compilación y sin dependencias instaladas localmente.

## Estructura

```text
mapa-vivo-humedales/
├── index.html      Visor completo: interfaz, datos y lógica
└── README.md       Presentación del proyecto
```

## Datos

Los conjuntos de datos están incorporados en `index.html` para que el visor funcione sin servidor. Incluyen humedales urbanos y establecimientos industriales SINADER 2024.

## Estado

Proyecto en desarrollo y demostración. Las cifras y visualizaciones corresponden a los datos incorporados en esta versión.
