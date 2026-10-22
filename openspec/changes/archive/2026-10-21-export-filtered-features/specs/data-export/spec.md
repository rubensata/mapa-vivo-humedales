# Spec Delta

## Purpose

Permite al usuario descargar en su equipo el subconjunto exacto de humedales y establecimientos SINADER que está analizando, en formatos GeoJSON y Shapefile, sin backend ni pérdida del estado de filtros.

## ADDED Requirements

### Requirement: Sección de exportación en el panel

El sistema SHALL incluir en el panel lateral una sección numerada de exportación, posterior a las secciones existentes, con el conteo del subconjunto filtrado, dos acciones de descarga y un mensaje de estado accesible.

#### Scenario: Sección visible y contextual

- **WHEN** el usuario despliega el panel lateral
- **THEN** aparece la sección de exportación con su índice, el conteo de humedales y establecimientos filtrados y las acciones de descarga

#### Scenario: Conteo actualizado

- **WHEN** el usuario cambia cualquier filtro
- **THEN** el conteo de la sección de exportación refleja de inmediato las mismas cantidades que el contador de filtros

#### Scenario: Panel colapsado

- **WHEN** el usuario colapsa el panel lateral
- **THEN** la sección de exportación se oculta junto con el resto del contenido y vuelve al expandir el panel

### Requirement: Exportación limitada a las entidades filtradas

El sistema SHALL exportar únicamente las entidades de las capas integradas que cumplen los filtros activos al momento de iniciar la descarga, incluyendo humedales y SINADER con independencia de la visibilidad de sus capas.

#### Scenario: Filtros activos

- **WHEN** el usuario descarga con filtros de región, comuna, proceso, rubro o tratamiento activos
- **THEN** el archivo contiene solo las entidades que cumplen esos filtros y sus cantidades coinciden con el contador del panel

#### Scenario: Capa oculta

- **WHEN** el usuario oculta una capa sin cambiar los filtros y luego exporta
- **THEN** la exportación incluye las entidades filtradas de esa capa aunque no estén visibles en el mapa

#### Scenario: Capas locales ajenas al filtro

- **WHEN** el usuario ha cargado capas GeoJSON propias
- **THEN** la exportación no incluye esas capas locales, porque no participan del estado de filtros del visor

#### Scenario: Cambio de filtros durante la generación

- **WHEN** el usuario modifica un filtro después de iniciar una exportación
- **THEN** el archivo corresponde al subconjunto capturado al iniciar la descarga y no mezcla entidades de dos estados distintos

### Requirement: Acciones deshabilitadas sin resultados

Las acciones de exportación SHALL estar deshabilitadas cuando no exista ninguna entidad filtrada disponible y mientras se esté generando una descarga, y SHALL habilitarse cuando vuelva a haber resultados y no haya una generación en curso.

#### Scenario: Sin resultados

- **WHEN** los filtros activos dejan en cero tanto humedales como establecimientos SINADER
- **THEN** ambas acciones de exportación aparecen deshabilitadas y la sección informa que no hay resultados para exportar

#### Scenario: Resultados restaurados

- **WHEN** el usuario limpia o ajusta los filtros y vuelve a existir al menos una entidad filtrada
- **THEN** las acciones de exportación se habilitan sin recargar la página

#### Scenario: Una sola capa con resultados

- **WHEN** solo una de las capas integradas tiene entidades filtradas
- **THEN** las acciones permanecen habilitadas y exportan únicamente las entidades disponibles

#### Scenario: Generación en curso

- **WHEN** una exportación está generando el archivo
- **THEN** ambas acciones quedan deshabilitadas y se anuncia el estado ocupado hasta que la generación termina o falla

### Requirement: Descarga GeoJSON del subconjunto filtrado

El sistema SHALL descargar un archivo GeoJSON válido con una `FeatureCollection` de las entidades filtradas, conservando sus geometrías y propiedades originales, en coordenadas WGS84 y codificación UTF-8.

#### Scenario: Contenido del archivo

- **WHEN** el usuario descarga GeoJSON
- **THEN** el archivo contiene una `FeatureCollection` con la suma de humedales y establecimientos filtrados, sin entidades adicionales

#### Scenario: Propiedades originales

- **WHEN** el usuario inspecciona una entidad del GeoJSON descargado
- **THEN** conserva las propiedades del dataset original, incluidos valores compuestos como el arreglo mensual de SINADER

#### Scenario: Generación local

- **WHEN** el usuario descarga GeoJSON
- **THEN** el archivo se construye en el navegador y se descarga sin enviar entidades a ningún servidor

#### Scenario: Nombre identificable

- **WHEN** el navegador guarda el archivo GeoJSON
- **THEN** el nombre incluye el prefijo del visor y una marca temporal que evita sobrescribir exportaciones anteriores

### Requirement: Descarga Shapefile comprimido en ZIP

El sistema SHALL descargar un archivo ZIP con shapefiles válidos separados por tipo de geometría, incluyendo las extensiones `.shp`, `.shx`, `.dbf` y `.prj`, con proyección WGS84 y solo los grupos que tengan entidades filtradas.

#### Scenario: Exportación con ambas geometrías

- **WHEN** existen humedales y establecimientos filtrados
- **THEN** el ZIP contiene un shapefile de polígonos para humedales y un shapefile de puntos para SINADER, cada uno con sus cuatro archivos

#### Scenario: Exportación de una sola geometría

- **WHEN** los filtros dejan resultados solo para una de las dos capas
- **THEN** el ZIP contiene únicamente el shapefile correspondiente a esa geometría, sin archivos vacíos de la otra capa

#### Scenario: Proyección declarada

- **WHEN** el usuario abre un shapefile exportado en un SIG de escritorio
- **THEN** el archivo `.prj` declara WGS84 y las coordenadas coinciden con las del visor

#### Scenario: Correspondencia con el panel

- **WHEN** el usuario compara los registros de los shapefiles con el contador filtrado
- **THEN** la cantidad de registros de cada shapefile coincide con la cantidad exportada de su capa

### Requirement: Compatibilidad de atributos en Shapefile

El Shapefile SHALL usar nombres de campo únicos de hasta 10 caracteres, convertir valores no escalares a texto y normalizar los textos a ASCII, de modo que la tabla de atributos se abra sin registros corruptos en herramientas SIG.

#### Scenario: Nombres de campo extensos

- **WHEN** una propiedad original supera los 10 caracteres o coincide con otra después del truncamiento
- **THEN** el DBF usa un nombre corto único y conserva el valor en la columna correspondiente

#### Scenario: Valores compuestos

- **WHEN** una entidad SINADER incluye el arreglo `meses`
- **THEN** el DBF lo almacena como texto serializado y el GeoJSON conserva el arreglo original

#### Scenario: Acentos y eñes

- **WHEN** un atributo contiene tildes o eñes
- **THEN** el DBF lo escribe en ASCII legible y el GeoJSON mantiene el texto UTF-8 original

### Requirement: Feedback de progreso y errores

El sistema SHALL informar el inicio, el éxito y el error de cada exportación mediante un estado visible con `aria-live` y notificaciones, sin bloquear la interacción con el resto del visor.

#### Scenario: Inicio de la generación

- **WHEN** el usuario activa una descarga
- **THEN** la sección muestra un mensaje de generación y anuncia el estado ocupado antes de descargar el archivo

#### Scenario: Éxito

- **WHEN** la generación termina correctamente
- **THEN** el sistema descarga el archivo, muestra una notificación con el formato y los conteos exportados y restaura las acciones

#### Scenario: Error de generación

- **WHEN** la generación falla
- **THEN** el sistema muestra una notificación de error comprensible, restaura las acciones y no descarga un archivo incompleto

#### Scenario: Dependencia de Shapefile no disponible

- **WHEN** no se puede cargar la dependencia que genera Shapefile
- **THEN** la descarga Shapefile informa el error y la descarga GeoJSON sigue disponible y operativa

### Requirement: Ejecución client-side y compatibilidad con archivo local

Todo el procesamiento y la descarga SHALL ocurrir en el navegador, sin backend, sin subir entidades y sin requerir un servidor local para abrir el visor.

#### Scenario: Sin subida de datos

- **WHEN** el usuario exporta cualquier formato
- **THEN** no se envían geometrías ni atributos a un servicio externo y el archivo se origina a partir de los datos ya cargados

#### Scenario: Apertura desde `file://`

- **WHEN** el usuario abre `index.html` directamente desde el sistema de archivos y exporta
- **THEN** las descargas se generan y guardan correctamente sin depender de un servidor

#### Scenario: GeoJSON sin dependencia adicional

- **WHEN** el usuario descarga GeoJSON
- **THEN** la operación funciona con las APIs nativas del navegador, aunque la dependencia de Shapefile no se haya cargado
