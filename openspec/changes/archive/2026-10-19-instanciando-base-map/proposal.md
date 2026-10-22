# Proposal

## Why

El visor existente ya referencia MapLibre GL JS por CDN y lo instancia en `index.html`, pero su cámara inicial está centrada en Buenos Aires con zoom 3. Se necesita formalizar la instanciación del mapa base sobre el contenedor `#map` existente y fijar una vista inicial de Chile con zoom 4, sin afectar el panel lateral ni los componentes ya presentes.

## What Changes

- Se explicita la instanciación de MapLibre GL JS reutilizando el contenedor `#map` existente, sin crear contenedores nuevos ni reubicar la interfaz.
- Se establece el mapa base oscuro de OpenFreeMap como estilo inicial, sin requerir API key.
- Se modifica la cámara inicial para centrar el mapa en Chile con zoom 4.
- Se mantiene el contenedor del mapa ocupando todo el espacio disponible del viewport y adaptándose al redimensionamiento.
- No se modifica la estructura ni el comportamiento del panel lateral, el selector de mapas base, la gestión de capas, los popups ni los controles existentes.

## Capabilities

### New Capabilities
- `base-map`: instanciación del mapa base MapLibre sobre `#map`, estilo oscuro, vista inicial de Chile y ocupación del espacio disponible.

### Modified Capabilities

Ninguna. La spec principal del proyecto está vacía y se evita crear una capacidad duplicada de `map-viewer`, que existe solo como delta del cambio pendiente `zero-server-gis-viewer`.

## Impact

- Código: `index.html`, bloque de inicialización de `map` y su configuración de cámara y estilo.
- Dependencias: se reutiliza MapLibre GL JS 4.7.1 ya incluido por CDN; no se agregan dependencias nuevas.
- UI: sin cambios en el panel lateral ni en los componentes existentes.
- Red: carga de estilos y mosaicos desde OpenFreeMap.
- OpenSpec: nueva capability `base-map` en este cambio; no se modifican specs principales inexistentes.
