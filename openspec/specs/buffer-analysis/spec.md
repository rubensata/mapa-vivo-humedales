# buffer-analysis Specification

## Purpose

Permite crear áreas de influencia alrededor de los humedales y establecimientos SINADER directamente en el visor, con distancia configurable, alcance total o filtrado y resultado individual o disuelto, para analizar cobertura territorial sin salir del navegador.

## Requirements

### Requirement: Sección de análisis de buffer en el panel

El sistema SHALL incluir en el panel lateral una sección numerada de análisis de buffer, posterior a la sección de exportación, con controles de distancia, alcance, modo, ejecución, limpieza y exportación del resultado.

#### Scenario: Sección visible y contextual

- **WHEN** el usuario despliega el panel lateral
- **THEN** aparece la sección de análisis con su índice, el campo de distancia en metros, las opciones de alcance y dissolve, y las acciones de ejecutar y limpiar

#### Scenario: Panel colapsado

- **WHEN** el usuario colapsa el panel lateral
- **THEN** la sección de análisis se oculta junto con el resto del contenido y vuelve al expandir el panel

### Requirement: Distancia de buffer validada

El sistema SHALL aceptar distancias enteras en metros entre 1 y 50.000, usar 1.000 metros como valor inicial y rechazar valores vacíos, no numéricos, decimales o fuera de rango sin ejecutar el análisis.

#### Scenario: Distancia válida

- **WHEN** el usuario ingresa una distancia entera dentro del rango permitido
- **THEN** el sistema habilita la ejecución con esa distancia y la conserva como parte del resultado

#### Scenario: Distancia inválida

- **WHEN** el usuario ingresa un valor vacío, no numérico, decimal, cero, negativo o mayor que 50.000
- **THEN** el sistema muestra un error comprensible, no ejecuta el buffer y no modifica el resultado anterior

### Requirement: Alcance total o filtrado con snapshot

El sistema SHALL permitir ejecutar el buffer sobre todas las entidades integradas o solo sobre las que cumplen los filtros activos al iniciar la ejecución, usando una instantánea que no cambie si los filtros se modifican durante el cálculo.

#### Scenario: Dataset completo

- **WHEN** el usuario elige el dataset completo y ejecuta
- **THEN** el análisis incluye todas las entidades de humedales y SINADER, con independencia de los filtros y de la visibilidad de las capas

#### Scenario: Solo filtradas

- **WHEN** el usuario elige solo las entidades filtradas y ejecuta
- **THEN** el análisis incluye exactamente las entidades que cumplían los filtros en el momento del clic

#### Scenario: Cambio de filtros durante el cálculo

- **WHEN** el usuario modifica un filtro después de iniciar el análisis
- **THEN** el resultado corresponde a la instantánea inicial y no mezcla entidades de dos estados distintos

#### Scenario: Capas locales excluidas

- **WHEN** el usuario ha cargado capas GeoJSON propias
- **THEN** el análisis no incluye esas capas locales porque no participan del estado de filtros del visor

#### Scenario: Sin entidades seleccionadas

- **WHEN** el alcance filtrado no contiene ninguna entidad
- **THEN** el sistema informa que no hay entidades para analizar y no genera una capa vacía

### Requirement: Buffers individuales

El sistema SHALL generar, cuando el dissolve está desactivado, un área de influencia independiente por cada entidad de origen y conservar trazabilidad suficiente para identificar la capa, el identificador y el nombre de la entidad que la originó.

#### Scenario: Resultado individual

- **WHEN** el usuario ejecuta el buffer con el dissolve desactivado
- **THEN** la capa resultante contiene una geometría de área por cada entidad analizada y sus atributos identifican la fuente y la distancia aplicada

#### Scenario: Entidades superpuestas

- **WHEN** dos o más entidades generan áreas que se superponen
- **THEN** el modo individual conserva las geometrías separadas y no fusiona sus límites

### Requirement: Dissolve de áreas de influencia

El sistema SHALL ofrecer un modo dissolve que una todas las áreas generadas, elimine solapamientos internos y produzca una o más geometrías disjuntas con atributos genéricos de análisis.

#### Scenario: Dissolve activado

- **WHEN** el usuario ejecuta el buffer con dissolve activado
- **THEN** las áreas solapadas quedan unidas y el resultado no conserva bordes internos entre buffers que se intersectan

#### Scenario: Resultado con varias partes

- **WHEN** las áreas generadas forman grupos separados en el territorio
- **THEN** el resultado puede contener varias geometrías disjuntas dentro de la misma capa de análisis

#### Scenario: Atributos del dissolve

- **WHEN** el usuario inspecciona una geometría disuelta
- **THEN** encuentra la distancia aplicada, el modo de análisis y la cantidad de entidades de origen que contribuyeron al resultado

### Requirement: Visualización del resultado

El sistema SHALL mostrar el resultado como una capa de análisis independiente sobre el mapa, con control de visibilidad, sin modificar las capas de datos, sus filtros, sus contadores ni sus popups.

#### Scenario: Resultado visible

- **WHEN** el análisis termina correctamente
- **THEN** la capa resultante aparece en el mapa con un estilo diferenciado y la sección muestra su cantidad de geometrías y la distancia utilizada

#### Scenario: Ocultar y mostrar

- **WHEN** el usuario alterna la visibilidad de la capa de análisis
- **THEN** solo cambia la visibilidad del resultado y se conservan las capas de datos y sus estados

#### Scenario: Nuevo análisis

- **WHEN** el usuario ejecuta un nuevo buffer con otros parámetros
- **THEN** el resultado anterior se reemplaza de forma atómica por el nuevo resultado

#### Scenario: Limpiar resultado

- **WHEN** el usuario limpia el análisis
- **THEN** la capa resultante desaparece del mapa y las acciones de exportación del resultado quedan deshabilitadas

### Requirement: Progreso y errores

El sistema SHALL informar el inicio, el progreso, el éxito y el error del análisis mediante un estado visible con `aria-live`, mantener la interfaz utilizable durante el cálculo y restaurar los controles al finalizar.

#### Scenario: Inicio y progreso

- **WHEN** el usuario ejecuta un análisis
- **THEN** la sección anuncia que el cálculo está en curso y muestra el avance antes de publicar el resultado

#### Scenario: Interfaz utilizable

- **WHEN** el análisis del dataset completo está en curso
- **THEN** el usuario puede seguir interactuando con el mapa y el resto del visor mientras el resultado no esté listo

#### Scenario: Error de análisis

- **WHEN** el cálculo falla por datos, memoria o fallo de la dependencia
- **THEN** el sistema muestra un error comprensible, restaura los controles y no publica un resultado parcial

#### Scenario: Advertencia de dissolve completo

- **WHEN** el usuario elige el dataset completo con dissolve
- **THEN** el sistema advierte antes de ejecutar que el cálculo puede tardar decenas de segundos

### Requirement: Exportación del resultado

El sistema SHALL permitir exportar la capa de análisis a GeoJSON y a Shapefile ZIP, con geometrías WGS84 y atributos de análisis, únicamente cuando exista un resultado y no haya un cálculo en curso.

#### Scenario: Exportar GeoJSON

- **WHEN** el usuario descarga el resultado en GeoJSON
- **THEN** obtiene una `FeatureCollection` con las geometrías del análisis y sus atributos, sin entidades de datos originales

#### Scenario: Exportar Shapefile

- **WHEN** el usuario descarga el resultado en Shapefile
- **THEN** obtiene un ZIP con una capa de polígonos, sus archivos `.shp`, `.shx`, `.dbf` y `.prj`, proyección WGS84 y atributos DBF compatibles

#### Scenario: Nombre identificable

- **WHEN** el navegador guarda una exportación del resultado
- **THEN** el nombre incluye el prefijo del análisis, el alcance, la distancia y una marca temporal

#### Scenario: Sin resultado o cálculo en curso

- **WHEN** no existe un resultado o el análisis está en curso
- **THEN** las acciones de exportación del resultado aparecen deshabilitadas

### Requirement: Ejecución client-side y compatibilidad con archivo local

Todo el cálculo del buffer y la exportación del resultado SHALL ocurrir en el navegador, sin subir geometrías ni atributos y sin requerir un servidor local para abrir el visor.

#### Scenario: Sin subida de datos

- **WHEN** el usuario ejecuta el análisis
- **THEN** las entidades seleccionadas se procesan localmente y no se envían a un servicio externo

#### Scenario: Apertura desde file://

- **WHEN** el usuario abre `index.html` directamente desde el sistema de archivos y ejecuta el análisis
- **THEN** el buffer se calcula y el resultado se visualiza sin depender de un backend

#### Scenario: Dependencia no disponible

- **WHEN** no se puede cargar la dependencia de análisis
- **THEN** el sistema informa el error y mantiene operativas las demás funciones del visor
