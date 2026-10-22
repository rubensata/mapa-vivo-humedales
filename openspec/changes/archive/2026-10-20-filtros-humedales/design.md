# Design

## Context

El visor es un archivo único que mantiene un registro en `state.layers` y reconstruye las capas desde ese registro en `reapplyAllLayers()` después de cargar el estilo o cambiar de mapa base. La capa de humedales es una capa `builtIn` con subcapas de relleno, línea y círculo, y sus datos ya están embebidos en `HUMEDALES_DATA`. Ver `proposal.md` para la motivación.

Los filtros no pueden vivir solo en el estilo de MapLibre: si se aplican con `setFilter` y luego se cambia de mapa base, la reconstrucción de capas perdería la restricción. Por eso la selección debe formar parte del registro de la capa.

## Goals / Non-Goals

**Goals:**

- Filtrar la capa de humedales por región, comuna y proceso sin transformar ni recortar el dataset.
- Hacer que la cámara acompañe al subconjunto filtrado.
- Mantener los filtros tras cambiar de mapa base y al ocultar/mostrar la capa.

**Non-Goals:**

- No se filtran capas cargadas por el usuario.
- No se agregan búsquedas de texto libre, filtros por superficie ni selección múltiple de valores.
- No se modifican la simbología, la leyenda, el popup ni el dataset original.

## Decisions

### 1. Filtro como estado de la capa

Se agrega `state.humedalesFilter = { region: '', comuna: '', proceso: '' }` y una función que traduce ese estado a una expresión de MapLibre:

```js
['all',
  ['==', ['get', 'region'], region],
  ['==', ['get', 'comuna'], comuna],
  ['==', ['get', 'proceso_humedal_urbano'], proceso]
]
```

Solo se incluyen las cláusulas con valor. La expresión se guarda en el registro de la capa (`filterExpression`) y `reapplyAllLayers()` la compone con el filtro geométrico existente de cada subcapa:

```js
['all', filtroGeometrico, filterExpression]
```

Así el filtro sobrevive a cualquier reconstrucción. Alternativa considerada: llamar `setFilter` solo desde el evento `change`; se descarta porque el cambio de mapa base elimina las capas y el filtro se perdería.

### 2. Opciones derivadas del dataset

Al iniciar se recorren las features de `HUMEDALES_DATA` para construir:

- Lista ordenada de regiones.
- Lista ordenada de procesos.
- Mapa `region -> comunas ordenadas`.

Los selectores se llenan con `document.createElement('option')` y `textContent`, con la opción vacía `Todas`/`Todos` según el campo. Al cambiar la región se reconstruye el selector de comuna; si la comuna seleccionada ya no pertenece a la región, se restablece a `Todas`. Se usa `localeCompare('es')` para ordenar con acentos correctamente.

Alternativa considerada: listas fijas en el HTML. Se descarta porque el dataset puede reemplazarse y las opciones quedarían desactualizadas.

### 3. Ajuste de cámara con la extensión filtrada

Tras aplicar los filtros se filtra una copia de `HUMEDALES_DATA.features` y se calcula su extensión con la función existente `calculateGeoJSONBounds()`. Si hay coincidencias, se llama `map.fitBounds(bounds, { padding: 60, maxZoom: 14, duration: 1200 })`. Si no hay coincidencias, no se mueve la cámara y se emite un toast de advertencia.

`maxZoom: 14` evita que un humedal pequeño produzca un acercamiento excesivo. La acción `Limpiar filtros` restablece los selectores y ejecuta `map.flyTo` a la vista inicial (`[-71, -35]`, zoom 4).

### 4. Contador de resultados

La sección de filtros incluye un contador con el formato `N humedales`, que se recalcula con el mismo arreglo de features filtradas que alimenta `fitBounds`. Cuando no hay filtros activos muestra el total (107).

### 5. Ubicación y manejo de eventos

Se agrega una sección `Filtros` al panel lateral entre el selector de mapa base y la carga de archivos locales, con tres `<select>` etiquetados y un botón `Limpiar filtros`. Un único manejador `change` por selector llama a la función `applyHumedalesFilters()`, que actualiza el estado, el contador, el filtro de las subcapas y la cámara.

La capa afectada se identifica con `layer.filterable === true` en su registro, de modo que las capas de usuario nunca reciben el filtro.

## Risks / Trade-offs

- **Nombres de campo inconsistentes** → se usan los nombres exactos observados en el dataset (`region`, `comuna`, `proceso_humedal_urbano`) y se verifica que las opciones provengan de esos mismos valores.
- **Filtros que se pierden al cambiar de mapa base** → la expresión se guarda en el registro y se recompone en `reapplyAllLayers()`; se agrega verificación explícita de este caso.
- **Acercamiento excesivo en humedales pequeños** → `maxZoom` acota el zoom y `padding` deja contexto.
- **Combinaciones sin resultados** → se informa con toast, el contador muestra cero y no se altera la cámara.
- **Región con muchas comunas** → los selectores nativos mantienen una lista manejable para 65 comunas y no requieren componentes adicionales.
- **Datos con acentos** → se usa `localeCompare('es')` y `textContent`, evitando comparaciones dependientes del orden de bytes.

## Migration Plan

1. Agregar la sección de filtros y el contador al panel.
2. Construir las opciones y los manejadores desde el dataset embebido.
3. Guardar y aplicar la expresión de filtro en las subcapas de humedales.
4. Implementar el ajuste de cámara y la limpieza de filtros.
5. Verificar persistencia al cambiar de mapa base y al alternar visibilidad.

Rollback: retirar la sección de filtros, el estado `humedalesFilter`, la composición en `reapplyAllLayers()` y las funciones asociadas. No hay cambios de datos ni dependencias nuevas.

## Open Questions

Ninguna pendiente que afecte el alcance o las specs.
