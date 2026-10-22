# Spec Delta

## Purpose

Define un dashboard de análisis cruzado en el que mapa y gráficos comparten un único estado de filtros, de modo que la exploración visual actualice el mapa y los filtros del visor actualicen los gráficos sin duplicar criterios ni introducir inconsistencias.

## ADDED Requirements

### Requirement: Estado único de filtros

El sistema SHALL mantener un único estado canónico para `region`, `comuna`, `proceso`, `rubro` y `tratamiento`, y todos los consumidores SHALL derivar de él sin mantener copias independientes.

#### Scenario: Un solo origen

- **WHEN** el usuario cambia un filtro desde un selector o desde un gráfico
- **THEN** el mapa, los selectores, los contadores, la exportación, el snapshot del buffer y los tres gráficos reflejan el mismo estado

#### Scenario: Sin estado duplicado

- **WHEN** se inspecciona el estado durante una interacción
- **THEN** no existen valores de filtro distintos entre la vista y los consumidores

### Requirement: Sincronización de filtros entre mapa y gráficos

Los filtros del visor SHALL actualizar los gráficos en la misma interacción, y las acciones de los gráficos SHALL actualizar los filtros del visor y el mapa.

#### Scenario: Selector hacia gráficos

- **WHEN** el usuario cambia región, comuna, proceso, rubro o tratamiento en los selectores
- **THEN** los tres gráficos se recalculan con el nuevo subconjunto en la misma interacción

#### Scenario: Gráfico hacia mapa

- **WHEN** el usuario activa una categoría desde un gráfico
- **THEN** el selector correspondiente, el filtro de la capa y la cámara del mapa reflejan esa categoría

### Requirement: Criterios de filtrado compartidos

El sistema SHALL aplicar exactamente los mismos criterios de coincidencia a los datos usados por el mapa, los gráficos, los contadores, la exportación y el buffer.

#### Scenario: Coincidencia entre consumidores

- **WHEN** el usuario compara el conteo de una categoría en un gráfico con el conteo de entidades visibles
- **THEN** ambos valores coinciden para el mismo estado de filtros

#### Scenario: Filtros propios de capa

- **WHEN** hay un filtro de proceso activo
- **THEN** ese filtro solo afecta a humedales y no elimina establecimientos SINADER

#### Scenario: Filtro de tratamiento

- **WHEN** hay un tratamiento activo
- **THEN** SINADER conserva los establecimientos con tonelaje en esa categoría y humedales no se ve afectado

### Requirement: Gráfico de superficie de humedales por región

El sistema SHALL mostrar un gráfico de barras horizontales con la superficie total en hectáreas y el conteo de humedales filtrados por región.

#### Scenario: Agregación por región

- **WHEN** no hay filtros de humedales activos
- **THEN** cada barra muestra la suma de `ha` y el conteo de humedales de una región

#### Scenario: Filtros activos

- **WHEN** el usuario aplica región, comuna o proceso
- **THEN** el gráfico conserva solo las regiones con humedales coincidentes y recalcula sus valores

#### Scenario: Sin humedales

- **WHEN** los filtros activos dejan la capa de humedales sin resultados
- **THEN** el gráfico muestra un estado vacío explícito en lugar de barras obsoletas

### Requirement: Gráfico de tonelaje por rubro y tratamiento

El sistema SHALL mostrar un gráfico de barras horizontales apiladas con el tonelaje SINADER de los diez rubros con mayor tonelaje, desglosado por tratamiento.

#### Scenario: Agregación por rubro

- **WHEN** no hay filtros SINADER activos
- **THEN** cada barra representa un rubro y sus segmentos suman el tonelaje de `Eliminación`, `Recepción y Almacenamiento` y `Valorización`

#### Scenario: Filtros activos

- **WHEN** el usuario aplica filtros compartidos, de rubro o de tratamiento
- **THEN** el gráfico recalcula el top de rubros y sus segmentos con los establecimientos coincidentes

#### Scenario: Sin establecimientos

- **WHEN** los filtros activos dejan SINADER sin resultados
- **THEN** el gráfico muestra un estado vacío explícito

### Requirement: Gráfico mensual por tratamiento

El sistema SHALL mostrar un gráfico de barras verticales apiladas con el tonelaje SINADER de los doce meses de 2024, desglosado por tratamiento.

#### Scenario: Agregación mensual

- **WHEN** no hay filtros SINADER activos
- **THEN** la suma de los doce meses coincide con el tonelaje total de los establecimientos filtrados

#### Scenario: Filtros activos

- **WHEN** el usuario cambia un filtro compartido o propio de SINADER
- **THEN** las doce barras y sus segmentos se recalculan con los establecimientos coincidentes

#### Scenario: Sin establecimientos

- **WHEN** los filtros activos dejan SINADER sin resultados
- **THEN** el gráfico muestra un estado vacío explícito

### Requirement: Interacciones gráfico-mapa

Las categorías de los gráficos SHALL actuar como controles de filtro sobre el mapa y SHALL permitir deshacer el filtro aplicado.

#### Scenario: Región desde humedales

- **WHEN** el usuario activa una región en el gráfico de humedales
- **THEN** se aplica el filtro compartido de región, se limpia la comuna incompatible y el mapa se ajusta a los humedales coincidentes

#### Scenario: Rubro desde SINADER

- **WHEN** el usuario activa un rubro en el gráfico de rubros
- **THEN** se aplica el filtro de rubro y el mapa se ajusta a los establecimientos coincidentes

#### Scenario: Tratamiento desde un segmento

- **WHEN** el usuario activa un segmento de tratamiento en el gráfico de rubros o en el mensual
- **THEN** se aplica el filtro de tratamiento y el mapa se ajusta a los establecimientos coincidentes

#### Scenario: Deshacer desde el gráfico

- **WHEN** el usuario activa de nuevo la categoría que ya estaba filtrada
- **THEN** ese filtro vuelve a la opción sin filtrar y el mapa y los gráficos se actualizan

#### Scenario: Sin resultados

- **WHEN** la acción del gráfico produce un conjunto vacío
- **THEN** el mapa conserva su vista y el sistema informa que no hay resultados

### Requirement: Acciones de filtro accesibles equivalentes

Cada interacción de filtro disponible en un gráfico SHALL tener una acción equivalente accesible por teclado y lector de pantalla.

#### Scenario: Navegación por teclado

- **WHEN** el usuario recorre la tarjeta del gráfico con teclado
- **THEN** encuentra botones o controles enfocables que aplican las mismas categorías que el clic sobre el gráfico

#### Scenario: Descripción del gráfico

- **WHEN** el usuario consulta un gráfico con lector de pantalla
- **THEN** recibe un nombre, un resumen de categorías y valores, y una alternativa tabular de los datos representados

### Requirement: Actualización reactiva sin bucles

El sistema SHALL actualizar la interfaz una sola vez por cambio de filtro y no SHALL reaccionar a sus propias actualizaciones como si fueran una nueva acción del usuario.

#### Scenario: Un solo ciclo

- **WHEN** el usuario cambia un filtro
- **THEN** los gráficos, el mapa y los contadores se actualizan una vez y no se produce una cascada de eventos

#### Scenario: Renderizado sin escritura

- **WHEN** un gráfico se vuelve a dibujar
- **THEN** el redibujado no modifica el estado de filtros ni dispara otra actualización

#### Scenario: Capa oculta

- **WHEN** el usuario oculta una capa sin cambiar los filtros
- **THEN** los gráficos conservan los totales filtrados y no se vacían por la visibilidad

### Requirement: Dependencia de gráficos con carga tolerante a fallos

El sistema SHALL cargar Chart.js 4.5.1 desde CDN con integridad SRI y SHALL mantener operativos el mapa y los filtros si la librería no carga.

#### Scenario: Carga diferida

- **WHEN** el visor termina su carga inicial
- **THEN** Chart.js se solicita una sola vez y no se repite en cada cambio de filtro

#### Scenario: CDN no disponible

- **WHEN** Chart.js no puede cargarse
- **THEN** el mapa, los selectores, los contadores, la exportación y el buffer siguen operativos, y la sección de análisis muestra una alternativa accesible

#### Scenario: Sin datos adicionales

- **WHEN** los gráficos se cargan o actualizan
- **THEN** no se solicitan datasets ni se envían entidades a servidores

### Requirement: Preservación de filtros y funciones existentes

El dashboard SHALL conservar los identificadores, opciones, cascada región-comuna y comportamientos de los filtros existentes al incorporar las interacciones de los gráficos.

#### Scenario: Cascada de comunas

- **WHEN** una región se aplica desde un gráfico
- **THEN** el selector de comuna se limita a las comunas de esa región y descarta una comuna incompatible

#### Scenario: Limpiar filtros

- **WHEN** el usuario activa `Limpiar filtros`
- **THEN** los cinco filtros vuelven a su opción sin filtrar, los tres gráficos recuperan los totales y el mapa regresa a la vista inicial de Chile

#### Scenario: Persistencia entre mapas base

- **WHEN** el usuario cambia de mapa base con filtros de gráfico activos
- **THEN** las capas y los gráficos conservan el mismo subconjunto filtrado

### Requirement: Composición y accesibilidad del dashboard

El dashboard SHALL mantener el mapa como elemento visual principal, integrar los gráficos de forma compacta en el panel y respetar la identidad Swiss, el responsive y la preferencia de movimiento reducido.

#### Scenario: Escritorio y tablet

- **WHEN** el visor se muestra en escritorio o tablet
- **THEN** los gráficos permanecen dentro del panel y el mapa conserva la mayor parte del área visible e interactiva

#### Scenario: Ancho móvil

- **WHEN** el visor se muestra a 375 px de ancho
- **THEN** los gráficos se apilan sin desbordamiento horizontal, los controles mantienen áreas táctiles utilizables y el mapa sigue accesible al colapsar el panel

#### Scenario: Movimiento reducido

- **WHEN** el sistema operativo informa preferencia por movimiento reducido
- **THEN** las animaciones de los gráficos se desactivan o atenúan sin impedir la actualización ni la interacción
