# Design

## Context

Ver `proposal.md` para la motivación. El visor es un único `index.html` con Tailwind CDN, MapLibre GL JS 4.7.1, datos de humedales embebidos y ejecución Zero-Server. La capa de humedales ya usa un registro genérico de capas (`state.layers`), filtros por región/comuna/proceso, popups, leyenda y reconstrucción tras cambiar el mapa base.

El archivo `gi-sinader-2024-ckan.xlsx` no es consumible directamente por el visor: pesa 43,5 MB, su hoja `Datos` contiene 120.843 filas y 40 columnas, y las coordenadas son atributos (`latitud`, `longitud`), no geometrías. La investigación confirmó 8.911 establecimientos, 8.542 coordenadas únicas a seis decimales y 220 coordenadas compartidas por más de un establecimiento.

## Goals / Non-Goals

**Goals:**

- Incorporar una segunda capa de puntos SINADER sin degradar el arranque ni las capacidades existentes.
- Mantener datos y simbología en fuentes separadas y reutilizar la simbología en mapa, leyenda y gráficos.
- Compartir los filtros de región/comuna y conservar los filtros propios de cada capa.
- Añadir análisis visual nativo, accesible y coherente con el sistema Swiss.
- Conservar el modelo Zero-Server y el funcionamiento por doble clic sobre `index.html`.

**Non-Goals:**

- No se añade backend, base de datos, API ni autenticación.
- No se carga el Excel en tiempo de ejecución ni se incorpora una librería de hojas de cálculo.
- No se agregan gráficos interactivos con tooltips, zoom o exportación; los gráficos son resúmenes estáticos y accesibles.
- No se modifica la geometría ni la simbología de humedales.
- No se reemplaza MapLibre, Tailwind ni el sistema visual Swiss existente.

## Decisions

### 1. Preprocesar el Excel en un GeoJSON agregado por establecimiento

La capa se construye fuera del runtime: se agregan las 120.843 declaraciones por `id_vu`, se suman `cantidad_toneladas` y se cuentan registros. El resultado es un `FeatureCollection` de 8.906 puntos, con coordenadas a seis decimales y propiedades preparadas para filtros, popup y gráficos.

Propiedades por feature:

| Propiedad | Uso |
| --- | --- |
| `id`, `nombre`, `razon`, `rut` | Identificación y popup |
| `region`, `provincia`, `comuna`, `rubro` | Filtros y popup |
| `toneladas`, `registros` | Popup, contador y gráficos |
| `tratamiento` | Simbología por tratamiento dominante |
| `meses` | Doce totales mensuales para el gráfico |
| `t_eliminacion`, `t_recepcion`, `t_valorizacion` | Filtro por presencia de tratamiento y gráfico |
| `ler1`, `ler2`, `ler3` | Capítulos LER con mayor tonelaje |

Se descartan 18 filas con coordenadas inválidas y 58 fuera de Chile, pertenecientes a 5 establecimientos. El dataset final conserva 120.767 registros y 5.634.872,476 toneladas; la diferencia con el total original es de 10.562,415 toneladas (0,19 %). Los establecimientos sin coordenada válida no se dibujan. Quedan 8.540 coordenadas únicas, 220 compartidas y un máximo de 19 establecimientos por coordenada.

El GeoJSON minificado mide aproximadamente 4,54 MB. La conversión es un paso de preparación de datos, no una dependencia del visor; se registra el hash del archivo fuente y se validan los totales antes de embeberlo.

**Alternativas descartadas:** cargar el Excel con SheetJS (43,5 MB, 334 MB de XML y una dependencia nueva); leer el xlsx a mano en el navegador (frágil y lento); usar un GeoJSON externo con `fetch` (falla con `file://` por CORS).

### 2. Embeber el GeoJSON y parsearlo después de la carga de MapLibre

El GeoJSON se inserta en un bloque `<script type="application/json" id="sinader-data">` después de los datos de humedales. El texto se conserva como dato, pero `JSON.parse` y el registro de la capa se ejecutan en el evento `load` de MapLibre. Si el evento ya ocurrió, se ejecuta de inmediato con una guarda de idempotencia.

Esto preserva el archivo único, el doble clic y el modelo Zero-Server, y cumple la carga diferida sin descargar el Excel. El costo es un `index.html` de aproximadamente 7,8 MB y un parseo único de 4,54 MB; se acepta frente a añadir un archivo externo o una dependencia.

**Alternativas descartadas:** un `sinader-data.js` externo (funciona con `file://` pero rompe el archivo único); `fetch` de un `.geojson` (CORS en `file://`); cargar el xlsx en runtime (inviable).

### 3. Extender el registro de capas, no duplicar el renderizador

`addGeoJSONLayer` ya crea fuente, subcapas de relleno/línea/círculo, tarjeta, popup y persistencia. Se extiende el registro con:

- `filterKey` (`humedales` o `sinader`) para elegir el constructor de filtros.
- `circleRadius`, `circleOpacity`, `circleStrokeColor` y `circleStrokeWidth` para el punto SINADER.
- `popupTemplate: 'sinader'` para el contenido específico.
- `symbology: 'sinader'` para construir expresiones de color desde la configuración.

`reapplyAllLayers` seguirá reconstruyendo ambas capas tras cambiar el mapa base. La capa SINADER se registra después de humedales para que sus puntos queden por encima de los polígonos.

**Alternativas descartadas:** un renderizador separado para SINADER (duplica persistencia, popups y tarjetas); modificar directamente el HTML de las capas (rompe el patrón existente).

### 4. Modelo de filtros compartido y específico

El estado de filtros pasa a tener cinco claves: `region`, `comuna`, `proceso`, `rubro` y `tratamiento`. Región y comuna son compartidos; proceso es exclusivo de humedales; rubro y tratamiento son exclusivos de SINADER.

Los catálogos se construyen por capa y se combinan para región/comuna:

- Regiones: unión de humedales y SINADER.
- Comunas: unión para la región seleccionada.
- Proceso: solo humedales.
- Rubro y tratamiento: solo SINADER.

Antes de que SINADER termine de cargar, los selectores muestran solo el catálogo de humedales; al registrarse la capa se refrescan las opciones preservando la selección. El filtro de tratamiento usa `t_eliminacion > 0`, `t_recepcion > 0` o `t_valorizacion > 0`, de modo que incluye establecimientos con tonelaje en esa categoría aunque su tratamiento dominante sea otro.

La cámara conserva el comportamiento existente de humedales: un cambio compartido o de proceso ajusta a los humedales filtrados si existen; si no existen, mantiene la vista y advierte. Un cambio de rubro o tratamiento ajusta a SINADER; sin resultados, mantiene la vista y advierte. `Limpiar filtros` restablece las cinco claves y vuela a Chile con zoom 4.

**Alternativas descartadas:** filtros separados de región/comuna para cada capa (duplica controles y rompe la lectura conjunta); reutilizar `filter-proceso` para rubro (mezcla semánticas y valores).

### 5. Simbología separada y consulta de puntos coubicados

`SINADER_SYMBOLOGY` declara las tres categorías con etiqueta, color y tratamiento. Las expresiones de color del mapa, la leyenda y los gráficos se generan desde esa misma configuración. El color representa el tratamiento dominante por tonelaje.

Los 8.906 puntos se dibujan con círculos de radio progresivo por zoom, borde claro y opacidad suficiente para los cuatro mapas base. No se aplica jitter: las coordenadas se conservan exactas. Cuando una coordenada agrupa varios establecimientos (220 casos, máximo 19), el manejador de clic consulta todas las features renderizadas en ese punto y muestra un popup con una lista desplazable de establecimientos; si solo hay uno, muestra el popup individual. Si el popup queda debajo del panel lateral, se desplaza el mapa lo mínimo necesario para dejarlo visible sin cambiar el zoom ni la capa consultada.

**Alternativas descartadas:** dispersar artificialmente los puntos (altera la ubicación); mostrar solo el primer punto (pierde hasta 18 establecimientos por coordenada); clustering (complica el popup y no resuelve la coubicación a máximo zoom).

### 6. Gráficos SVG nativos en una sección `07 Análisis`

Se añade una sección después de `06 Interacción` con tres gráficos:

1. Humedales por proceso: dos barras con los colores de la simbología de humedales.
2. SINADER por mes: doce barras con el tonelaje agregado de los establecimientos filtrados.
3. SINADER por tratamiento: tres barras con el tonelaje de cada categoría.

Los gráficos se construyen con `document.createElementNS` y `viewBox` responsive, sin librerías. Cada SVG incluye `role="img"`, `<title>`, `<desc>` y etiquetas numéricas visibles; no depende de hover. Se actualizan en la misma función que aplica filtros y usan los datos ya cargados. La visibilidad de una capa no altera los totales; solo los filtros. Si no hay datos, muestran un estado vacío explícito.

El establecimiento `72418` (COLEGIO CONCEPCIÓN SAN PEDRO CAMPUS VILLA) aporta 100,741 t, de las cuales 22,514 t no tienen tratamiento clasificado. Por eso las tres barras de tratamiento suman 5.634.849,995 t, 22,481 t menos que el total: 22,514 t sin categoría menos 0,033 t de compensación por redondeo en otros establecimientos. El gráfico excluye esas toneladas sin categoría y el total mensual sí cuadra exactamente; el popup muestra el tratamiento como `—` cuando corresponde.

**Alternativas descartadas:** Chart.js/D3 (dependencia y peso); canvas (peor accesibilidad y resolución); gráficos solo para SINADER (el usuario pidió integración con ambos conjuntos).

### 7. Contador dual y preservación de IDs

Se conservan `filter-region`, `filter-comuna`, `filter-proceso`, `filter-count`, `clear-filters`, `legend-content`, `layers-list` y los ganchos de las capas. Se agregan `filter-rubro`, `filter-tratamiento`, `analytics-section`, `chart-humedales`, `chart-sinader-month` y `chart-sinader-treatment`.

`filter-count` muestra dos cantidades separadas, por ejemplo `107 humedales · 8.906 establecimientos`; las tarjetas de capa conservan el total de su dataset. La cabecera actualiza su metadato de datos para mencionar ambas fuentes.

## Risks / Trade-offs

- **Aumento del HTML a ~7,8 MB** → parseo diferido tras `load`, una sola vez, y medición de tiempo de arranque; el usuario no descarga datos adicionales.
- **Puntos coubicados** → popup de lista con todos los establecimientos; se conserva la coordenada exacta y se prueba el caso máximo de 19.
- **Refactor de filtros puede afectar humedales** → conservar los IDs y el comportamiento de cámara de humedales; ejecutar la suite de regresión de 12 verificaciones y una suite nueva para SINADER.
- **Cambio de opciones de filtro después de cargar SINADER** → refrescar catálogos preservando la selección y probar la cascada región-comuna con ambos datasets.
- **Pérdida de 0,19 % del tonelaje por coordenadas inválidas** → documentar los 5 establecimientos excluidos y validar que los totales incluidos cuadran; no inventar coordenadas.
- **Crecimiento del registro de capas** → extender `addGeoJSONLayer` con opciones opcionales para no alterar las capas de usuario ni humedales.
- **Gráficos sin tooltips** → etiquetas visibles y descripciones accesibles; suficientes para el objetivo analítico y sin dependencias.
- **Conversión no reproducible desde el runtime** → registrar fuente, hash y totales en la verificación; el GeoJSON generado se versiona como artefacto de datos.

## Migration Plan

1. Generar el GeoJSON agregado y validar conteos, tonelaje y coordenadas contra el Excel.
2. Insertar el bloque de datos y registrar la capa SINADER en `load`, sin tocar humedales.
3. Extender el registro de capas, la simbología, la leyenda y el popup.
4. Migrar los filtros a cinco claves y refrescar catálogos al cargar SINADER.
5. Añadir la sección `07 Análisis` y enlazarla a los filtros.
6. Ejecutar regresión funcional y nuevas pruebas de datos, filtros, popups, gráficos, mapas base y responsive.

## Verification

- Fuente `gi-sinader-2024-ckan.xlsx`: SHA-256 `d027976f501dba2d152bffb87734980d491ff9f18940394142079a2c0c97ffcb`, 43.492.428 bytes.
- GeoJSON agregado: 4.754.257 bytes UTF-8, SHA-256 `8d4888c58c120f76b1f894630ffde9eb84dea47c2009e5eeff7956b112da3d25`; idéntico al bloque embebido (4.732.971 caracteres, mismo hash).
- Totales finales: 8.906 establecimientos, 120.767 registros, 5.634.872,476 t; la suma mensual cuadra exacta y la suma por tratamiento es 5.634.849,995 t (22,481 t menos, según la excepción documentada).
- Geometrías: 8.540 coordenadas únicas, 220 compartidas, máximo 19 establecimientos; 18 coordenadas inválidas y 58 fuera de Chile excluidas, con 5 establecimientos sin punto.
- `openspec.cmd validate --all --strict`: 7/7 artefactos válidos.
- Suite CDP de interacción: 28/28 (carga dual, filtros compartidos y propios, cascada, cámara, contador, limpieza, popups individual y de 19 coubicados, visibilidad del popup junto al panel en tablet, ocultar/mostrar, persistencia de mapa base, gráficos y estados vacíos, responsive 375/768/1440, panel colapsado y consola).
- Suite CDP extra: 3/3; arranque con `readyState` en 1,77 s y ambas capas en 3,10 s para un HTML de 7,72 MB; filtro de rubro aplicado en 66 ms; con el bloque SINADER corrupto, humedales y gráficos siguen operativos, la capa SINADER no se registra y aparece el toast de error.
- Capturas responsive revisadas en 375, 768 y 1440 px; sin overflow horizontal en el panel ni en la sección de análisis.

Rollback: retirar el bloque `sinader-data`, la llamada de inicialización, la sección `07` y las opciones de capa; humedales y la carga local no dependen de SINADER, por lo que el visor vuelve al estado anterior sin migración de datos.
