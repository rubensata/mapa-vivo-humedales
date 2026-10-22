# Spec Delta

## ADDED Requirements

### Requirement: Filtros combinables de humedales

El sistema SHALL ofrecer en el panel lateral filtros por `region`, `comuna` y `proceso_humedal_urbano`, cada uno con una opción para no filtrar, y SHALL mostrar únicamente los humedales que cumplan todos los filtros activos.

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

### Requirement: Opciones de filtro derivadas del dataset

El sistema SHALL construir las opciones de región, comuna y proceso a partir de los valores presentes en el dataset de humedales, ordenadas alfabéticamente y sin valores duplicados.

#### Scenario: Comunas dependientes de la región

- **WHEN** el usuario selecciona una región
- **THEN** el selector de comuna se limita a las comunas existentes en esa región y descarta una comuna previamente elegida que no pertenezca a ella

#### Scenario: Retorno a todas las comunas

- **WHEN** el usuario vuelve a seleccionar `Todas` en región
- **THEN** el selector de comuna vuelve a ofrecer todas las comunas del dataset

### Requirement: Ajuste de cámara a los resultados filtrados

El sistema SHALL ajustar la cámara a la extensión de los humedales que cumplen los filtros cada vez que cambie la selección, y SHALL mantener la vista actual cuando no existan resultados.

#### Scenario: Acercamiento a las zonas filtradas

- **WHEN** el usuario aplica o modifica un filtro y existen humedales coincidentes
- **THEN** el mapa ajusta su extensión a las geometrías visibles con un margen de padding

#### Scenario: Filtros sin resultados

- **WHEN** la combinación de filtros no devuelve ningún humedal
- **THEN** el mapa conserva su vista y el sistema informa que no hay resultados

#### Scenario: Limpiar filtros

- **WHEN** el usuario activa la acción `Limpiar filtros`
- **THEN** los tres selectores vuelven a su opción sin filtrar y el mapa regresa a la vista inicial de Chile con zoom 4

### Requirement: Contador de resultados filtrados

El sistema SHALL mostrar en el panel lateral la cantidad de humedales que cumplen los filtros activos.

#### Scenario: Actualización del contador

- **WHEN** el usuario cambia cualquier filtro
- **THEN** el contador refleja de inmediato la cantidad de humedales visibles según la nueva selección

### Requirement: Persistencia de filtros entre mapas base

El sistema SHALL mantener los filtros aplicados y el subconjunto visible de humedales al cambiar de mapa base o al alternar la visibilidad de la capa.

#### Scenario: Cambio de mapa base con filtros activos

- **WHEN** el usuario cambia de mapa base mientras hay filtros aplicados
- **THEN** la capa de humedales se reconstruye mostrando el mismo subconjunto filtrado y la cámara se conserva

#### Scenario: Visibilidad con filtros activos

- **WHEN** el usuario oculta y vuelve a mostrar la capa de humedales con filtros aplicados
- **THEN** la capa reaparece con el mismo subconjunto filtrado
