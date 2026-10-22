# Design

## Context

Véase `proposal.md` para la motivación. El estado actual de `index.html` ya incluye:

- MapLibre GL JS 4.7.1 por CDN (`unpkg`), junto con su hoja de estilos.
- Un contenedor `#map` con `absolute inset-0 w-full h-full z-0`, dentro de un `body` de altura completa y sin scroll.
- Una instancia de `maplibregl.Map` que usa el estilo oscuro de OpenFreeMap (`BASEMAPS.dark`), centrada en Buenos Aires (`[-58.381555, -34.603722]`) con `zoom: 3`.
- Panel lateral flotante, selector de mapas base, capa de demostración, carga local de GeoJSON, popups y controles, todos independientes del tamaño del contenedor del mapa.
- La spec principal del proyecto está vacía; `map-viewer` solo existe como delta del cambio pendiente `zero-server-gis-viewer`.

## Goals / Non-Goals

**Goals:**

- Explicitar la instanciación del mapa base sobre el `#map` existente.
- Fijar el estilo oscuro de OpenFreeMap y la cámara inicial en Chile con zoom 4.
- Garantizar que el mapa ocupa el viewport y se adapta al redimensionamiento.
- Mantener intactos la estructura HTML, los estilos y la lógica de los componentes existentes.

**Non-Goals:**

- Rediseñar el panel lateral o el sistema visual.
- Cambiar la lógica de capas, popups, drag & drop o controles.
- Agregar nuevos proveedores de mapas base o dependencias.
- Introducir backend, bundler o framework.

## Decisions

### 1. Reutilizar la integración existente de MapLibre GL JS
- **Decisión**: Conservar MapLibre GL JS 4.7.1 cargado por CDN y el bloque de inicialización actual; limitar el cambio a la configuración de la cámara.
- **Razón**: La dependencia ya está integrada y funcionando, por lo que volver a incorporarla violaría KISS y duplicaría código.
- **Alternativas descartadas**: Reemplazar la versión o migrar a otra librería; no aporta valor y agranda el riesgo de regresión.

### 2. Mapa base oscuro sin API key
- **Decisión**: Mantener `BASEMAPS.dark` con el estilo `https://tiles.openfreemap.org/styles/dark` como mapa base inicial.
- **Razón**: Cumple el requisito de estilo oscuro sin credenciales y ya está integrado con MapLibre.
- **Alternativas descartadas**: Mosaicos raster de OSM (bloqueos 403 desde `file://`) y proveedores con API key.

### 3. Cámara inicial: centro de Chile y zoom 4
- **Decisión**: Inicializar el mapa con centro `[-71.0, -35.0]` y `zoom: 4`.
- **Razón**: Es una aproximación al centro geográfico de Chile continental y el zoom 4 entrega contexto nacional, cumpliendo el comportamiento solicitado.
- **Supuesto**: Al no especificarse coordenadas exactas, se usa el centro geográfico del territorio continental. Es una constante fácil de ajustar si se prefiere Santiago u otro punto.
- **Alternativas descartadas**: `fitBounds` sobre un polígono del país (requiere datos adicionales y contradice el zoom fijo 4) y centrar en Santiago (sesga la vista a la capital en lugar de Chile).

### 4. Ocupación del espacio disponible sin cambios de layout
- **Decisión**: Mantener las clases actuales de `html`, `body` y `#map` a pantalla completa y confiar en el ajuste automático de MapLibre al redimensionar la ventana.
- **Razón**: El panel lateral es flotante y no altera el tamaño del contenedor; agregar `ResizeObserver` o código de resize sería complejidad innecesaria.
- **Alternativas descartadas**: Envolver el mapa en un nuevo contenedor o agregar lógica de resize personalizada.

### 5. Cambio mínimo y sin regresiones
- **Decisión**: Modificar únicamente los valores de `center` y `zoom` de la instancia del mapa, conservando el estilo oscuro, el manejador `style.load`, la reinyección de capas y el resto de la lógica.
- **Razón**: Reduce la superficie del cambio y protege el comportamiento existente de la capa demo, el selector de mapas base y los controles.

## Risks / Trade-offs

- **[Punto de centrado subjetivo]** → *Mitigación*: Se documenta `[-71.0, -35.0]` como constante; ajustarlo no afecta la arquitectura.
- **[Zoom 4 no muestra todo Chile por su longitud]** → *Mitigación*: El requisito pide zoom 4 explícito, no encuadrar el país completo; se acepta como vista de contexto nacional.
- **[Disponibilidad de OpenFreeMap]** → *Mitigación*: Se conserva el selector de mapas base existente con Bright, Liberty y satelital como alternativas en tiempo de ejecución.
- **[Regresión en la reinyección de capas al cambiar de estilo]** → *Mitigación*: No se toca el evento `style.load` ni la gestión de capas; se verifica la capa demo tras el cambio.

## Migration Plan

No se requiere migración. El cambio se aplica directamente en `index.html` y puede revertirse restaurando los valores anteriores de `center` y `zoom`.
