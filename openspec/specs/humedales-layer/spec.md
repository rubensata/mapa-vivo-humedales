# humedales-layer Specification

## Purpose
Define la capa predefinida de humedales urbanos y su representación cartográfica según el proceso de declaración, para que el visor comunique de forma inmediata ambas categorías sin trabajo adicional del usuario.

## Requirements

### Requirement: Capa de humedales predefinida y activa al iniciar
El sistema SHALL incluir una capa GeoJSON predefinida con los 107 humedales urbanos del dataset del proyecto y mostrarla activa al abrir el visor, sin exigir que el usuario seleccione o arrastre un archivo.

#### Scenario: Inicio del visor con la capa visible
- **WHEN** el usuario abre `index.html`
- **THEN** la capa de humedales se muestra en el mapa y aparece como una capa activa en el panel lateral

#### Scenario: Capa disponible sin carga manual
- **WHEN** el usuario abre el visor sin seleccionar ni arrastrar ningún archivo
- **THEN** la capa de humedales ya está disponible y puede alternarse desde el panel de capas

### Requirement: Simbología categórica por proceso de declaración
El sistema SHALL representar los humedales con relleno y borde cuyo color depende exclusivamente del valor de `proceso_humedal_urbano`, asignando un color distinto a `Municipal` y a `de Oficio`, con contraste suficiente sobre los mapas base oscuro, claro y satelital.

#### Scenario: Humedal declarado a solicitud municipal
- **WHEN** la capa de humedales se renderiza
- **THEN** las geometrías cuyo `proceso_humedal_urbano` es `Municipal` usan el color de la categoría municipal en relleno y borde

#### Scenario: Humedal declarado de oficio
- **WHEN** la capa de humedales se renderiza
- **THEN** las geometrías cuyo `proceso_humedal_urbano` es `de Oficio` usan un color distinto al de la categoría municipal en relleno y borde

#### Scenario: Persistencia de la simbología al cambiar de mapa base
- **WHEN** el usuario cambia entre los mapas base disponibles
- **THEN** la capa de humedales conserva la simbología de ambas categorías y sigue visible

### Requirement: Leyenda de simbología
El sistema SHALL mostrar en el panel lateral una leyenda con las categorías `Municipal` y `de Oficio` junto a los colores usados en el mapa.

#### Scenario: Identificación de categorías en la leyenda
- **WHEN** el usuario observa el panel lateral
- **THEN** la leyenda muestra ambas categorías con muestras de color que coinciden con la simbología del mapa

### Requirement: Popup con atributos del humedal
El sistema SHALL abrir un popup al hacer clic sobre un humedal con los campos `nombre_humedal`, `cod_humedal`, `region`, `comuna`, `proceso_humedal_urbano`, `ha`, `resolucion`, `url_res_bcn` y `url_simbio`, presentando los enlaces como referencias accesibles.

#### Scenario: Clic sobre un polígono de humedal
- **WHEN** el usuario hace clic sobre cualquier humedal de la capa
- **THEN** se abre un popup con los atributos del humedal y los enlaces de resolución y ficha disponibles

#### Scenario: Enlaces de referencia
- **WHEN** el popup muestra `url_res_bcn` o `url_simbio`
- **THEN** cada enlace puede abrirse en una pestaña nueva sin cerrar el visor

### Requirement: Filtros combinables de humedales
El sistema SHALL ofrecer en el panel lateral filtros por `region`, `comuna` y `proceso_humedal_urbano`, cada uno con una opción para no filtrar, y SHALL mostrar únicamente los humedales que cumplan todos los filtros de humedales activos; los filtros de rubro y tratamiento de SINADER no alteran esta capa.

#### Scenario: Filtrar por región
- **WHEN** el usuario selecciona una región distinta de `Todas`
- **THEN** la capa de humedales muestra solo las geometrías cuya `region` coincide con la seleccionada

#### Scenario: Filtrar por comuna
- **WHEN** el usuario selecciona una comuna distinta de `Todas`
- **THEN** la capa de humedales muestra solo las geometrías cuya `comuna` coincide con la seleccionada

#### Scenario: Filtrar por proceso de declaración
- **WHEN** el usuario selecciona un proceso distinto de `Todos`
- **THEN** la capa de humedales muestra solo las geometrías cuyo `proceso_humedal_urbano` coincide con el proceso seleccionado

#### Scenario: Combinar varios filtros
- **WHEN** el usuario tiene más de un filtro activo
- **THEN** la capa muestra solo los humedales que cumplen simultáneamente todos los filtros seleccionados

#### Scenario: Filtros de SINADER sin efecto en humedales
- **WHEN** el usuario cambia un filtro de rubro o tratamiento
- **THEN** el subconjunto y el contador de humedales no cambian

### Requirement: Opciones de filtro derivadas del dataset
El sistema SHALL construir las opciones de región y comuna a partir de los valores presentes en humedales y SINADER, y las de proceso a partir del dataset de humedales, ordenadas alfabéticamente y sin valores duplicados.

#### Scenario: Comunas dependientes de la región
- **WHEN** el usuario selecciona una región
- **THEN** el selector de comuna se limita a las comunas existentes en esa región y descarta una comuna previamente elegida que no pertenezca a ella

#### Scenario: Retorno a todas las comunas
- **WHEN** el usuario vuelve a seleccionar `Todas` en región
- **THEN** el selector de comuna vuelve a ofrecer todas las comunas del catálogo combinado

#### Scenario: Regiones y comunas compartidas
- **WHEN** el usuario abre los selectores de región o comuna
- **THEN** las opciones incluyen los valores de humedales y de SINADER sin duplicados

### Requirement: Ajuste de cámara a los resultados filtrados
El sistema SHALL ajustar la cámara a la extensión de los humedales que cumplen los filtros compartidos o de humedales cada vez que cambie la selección, y SHALL mantener la vista actual cuando no existan resultados. Los filtros propios de SINADER SHALL ajustar la cámara a los establecimientos conforme a la capa SINADER.

#### Scenario: Acercamiento a las zonas filtradas
- **WHEN** el usuario aplica o modifica un filtro compartido o de humedales y existen humedales coincidentes
- **THEN** el mapa ajusta su extensión a las geometrías visibles con un margen de padding

#### Scenario: Filtros sin resultados
- **WHEN** la combinación de filtros compartidos o de humedales no devuelve ningún humedal
- **THEN** el mapa conserva su vista y el sistema informa que no hay resultados de humedales

#### Scenario: Limpiar filtros
- **WHEN** el usuario activa la acción `Limpiar filtros`
- **THEN** los cinco selectores vuelven a su opción sin filtrar, ambas capas recuperan todos sus datos y el mapa regresa a la vista inicial de Chile con zoom 4

#### Scenario: Filtros propios de SINADER
- **WHEN** el usuario cambia un filtro de rubro o tratamiento y existen establecimientos coincidentes
- **THEN** el mapa ajusta su extensión a los establecimientos visibles sin modificar el subconjunto de humedales

### Requirement: Contador de resultados filtrados
El sistema SHALL mostrar en el panel lateral la cantidad de humedales que cumplen los filtros activos y, por separado, la cantidad de establecimientos SINADER que cumplen los filtros compartidos y propios.

#### Scenario: Actualización del contador
- **WHEN** el usuario cambia un filtro compartido o un filtro de humedales
- **THEN** el contador de humedales refleja de inmediato la cantidad de humedales visibles según la nueva selección

#### Scenario: Contador SINADER
- **WHEN** el usuario cambia un filtro compartido o un filtro propio de SINADER
- **THEN** el contador de establecimientos refleja de inmediato la cantidad de establecimientos SINADER visibles

#### Scenario: Distinción de ambas cantidades
- **WHEN** el usuario observa el encabezado de filtros
- **THEN** las cantidades de humedales y de establecimientos se muestran de forma separada y comprensible

### Requirement: Persistencia de filtros entre mapas base
El sistema SHALL mantener los filtros aplicados y el subconjunto visible de humedales al cambiar de mapa base o al alternar la visibilidad de la capa.

#### Scenario: Cambio de mapa base con filtros activos
- **WHEN** el usuario cambia de mapa base mientras hay filtros aplicados
- **THEN** la capa de humedales se reconstruye mostrando el mismo subconjunto filtrado y la cámara se conserva

#### Scenario: Visibilidad con filtros activos
- **WHEN** el usuario oculta y vuelve a mostrar la capa de humedales con filtros aplicados
- **THEN** la capa reaparece con el mismo subconjunto filtrado
