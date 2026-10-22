# Proposal

## Why

El visor solo ofrece la capa de humedales urbanos; el archivo `gi-sinader-2024-ckan.xlsx` aporta una segunda capa de establecimientos industriales con coordenadas, pero no puede consumirse tal cual desde el navegador. Además, el panel no tiene gráficos, por lo que la lectura analítica de los datos queda limitada a la inspección punto a punto.

### Hallazgos del dataset inspeccionado

- Archivo: `gi-sinader-2024-ckan.xlsx`, 43,5 MB; hoja `Datos` con XML interno de 334 MB y hoja `Resumen` descriptiva.
- 120.843 registros, 40 atributos, año 2024.
- Geometría: punto implícito en `latitud` y `longitud`; no es un GeoJSON ni contiene geometrías poligonales.
- Entidades: 8.911 `id_vu` (establecimiento), 8.542 coordenadas únicas y 70.324 `declaracion_id`. Cada establecimiento usa una sola coordenada; 220 coordenadas son compartidas por más de un establecimiento.
- Volumen: 120.825 registros con coordenadas válidas; 18 inválidas y 58 fuera del territorio chileno, concentradas en 5 establecimientos.
- Nulos relevantes: `nombre_establecimiento` 33; `tratamiento_n2_name` 755; `tratamiento_n3_name` 5.169; campos `trazabilidad__*` entre 349 y 3.194. `region`, `comuna`, `rubro`, `latitud`, `longitud` y `cantidad_toneladas` no presentan nulos.
- Valores: 16 regiones, 316 comunas, 19 rubros, 3 tratamientos primarios, 12 secundarios y 63 terciarios; `tipo_recepcion` y `rol_establecimiento` son constantes.
- Tonelaje: 5.645.434,891 t declaradas; 5.634.872,476 t en los registros con coordenadas utilizables.
- Agregación propuesta: 8.906 establecimientos con coordenadas válidas; GeoJSON minificado de aproximadamente 4,54 MB.

## What Changes

- Incorporar una capa predefinida de establecimientos SINADER 2024, construida a partir de las declaraciones y agregada por `id_vu`, con un punto por establecimiento.
- Cargar y registrar la capa únicamente cuando MapLibre haya terminado de inicializarse, sin bloquear el arranque ni la capa de humedales.
- Separar los datos de la simbología: el GeoJSON solo contiene atributos y la configuración visual define categorías, colores, leyenda y gráficos.
- Representar los establecimientos según su tratamiento primario dominante y mostrar en el popup razón social, RUT, región, comuna, rubro, tonelaje total, registros, tratamiento dominante y principales capítulos LER.
- Listar en un mismo popup todos los establecimientos que comparten coordenada, para que ninguna de las 220 ubicaciones compartidas oculte entidades.
- Extender los filtros para que región y comuna afecten a ambas capas, manteniendo `proceso_humedal_urbano` como filtro exclusivo de humedales y agregando filtros de rubro y tratamiento para SINADER.
- Agregar una sección de análisis con gráficos SVG nativos, sin librerías: distribución de humedales por proceso y tonelaje SINADER por mes y por tratamiento, actualizados con los filtros activos.
- Excluir de la capa las coordenadas inválidas o fuera de Chile, sin descartar los registros válidos de otros establecimientos; documentar la pérdida de tonelaje.
- Conservar la carga local, los mapas base, las capas de usuario, los popups, los toasts, el panel colapsable y el modelo Zero-Server.

## Capabilities

### New Capabilities

- `sinader-layer`: capa predefinida de establecimientos SINADER 2024, agregación, carga diferida tras MapLibre, simbología, leyenda, popup, filtros propios y separación datos/simbología.
- `data-charts`: sección de gráficos vinculada a los filtros y a las capas, con SVG nativo, estados vacíos y coherencia visual con la simbología.

### Modified Capabilities

- `humedales-layer`: los filtros de región y comuna pasan a ser compartidos con SINADER, las opciones se derivan del catálogo combinado y el contador distingue ambos conjuntos sin alterar el filtrado de humedales.

## Impact

- `index.html`: nueva capa built-in, orquestación de carga diferida, estado de filtros compartidos, sección `07 Análisis`, estilos y contratos de prueba.
- Datos: nuevo bloque JSON embebido de aproximadamente 4,5 MB, generado a partir del xlsx; el archivo Excel no se carga en tiempo de ejecución.
- Dependencias: ninguna nueva; se mantienen Tailwind CDN y MapLibre 4.7.1.
- Compatibilidad: se preservan IDs, funciones de capas, mapas base, carga local, popups y toasts; no hay backend ni llamadas de datos adicionales.
