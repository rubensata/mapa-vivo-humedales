# Spec Delta

## Purpose

Define una sección de análisis visual que resume el estado filtrado de humedales y SINADER con gráficos nativos, accesibles y coherentes con la simbología del mapa, sin añadir dependencias ni backend.

## ADDED Requirements

### Requirement: Sección de análisis en el panel
El sistema SHALL incluir en el panel lateral una sección numerada de análisis con gráficos de humedales y de SINADER, accesible desde la misma vista y sin sustituir las secciones existentes.

#### Scenario: Sección visible
- **WHEN** el usuario despliega el panel lateral
- **THEN** aparece la sección de análisis después de las secciones existentes, con título e índice numerado

#### Scenario: Panel colapsado
- **WHEN** el usuario colapsa el panel
- **THEN** la sección de análisis se oculta junto con el resto del contenido y vuelve al expandir el panel

#### Scenario: Sin recarga
- **WHEN** el usuario interactúa con filtros, capas o mapas base
- **THEN** los gráficos se actualizan o conservan su estado sin recargar la página

### Requirement: Gráfico de humedales por proceso
El sistema SHALL mostrar un gráfico de barras con la cantidad de humedales filtrados por proceso de declaración, usando los mismos colores de la simbología de humedales.

#### Scenario: Conteo por proceso
- **WHEN** no hay filtros de humedales activos
- **THEN** el gráfico muestra la cantidad total de humedales y su distribución entre `Municipal` y `de Oficio`

#### Scenario: Filtro de proceso
- **WHEN** el usuario selecciona un proceso de declaración
- **THEN** el gráfico muestra solo la categoría seleccionada y su cantidad filtrada

#### Scenario: Coincidencia con el mapa
- **WHEN** el usuario compara el gráfico con los humedales visibles
- **THEN** las cantidades y los colores coinciden con la capa y la leyenda

### Requirement: Gráfico SINADER de tonelaje mensual
El sistema SHALL mostrar un gráfico de barras con el tonelaje SINADER agregado por mes para los establecimientos que cumplen los filtros activos.

#### Scenario: Doce meses
- **WHEN** el gráfico SINADER se renderiza
- **THEN** muestra los doce meses del año 2024 con su tonelaje agregado

#### Scenario: Actualización por filtros
- **WHEN** el usuario cambia un filtro compartido o propio de SINADER
- **THEN** las barras mensuales se recalculan con los establecimientos que siguen visibles

#### Scenario: Suma verificable
- **WHEN** no hay filtros activos
- **THEN** la suma de las doce barras coincide con el tonelaje total de la capa SINADER

### Requirement: Gráfico SINADER por tratamiento
El sistema SHALL mostrar la distribución de tonelaje SINADER entre `Eliminación`, `Recepción y Almacenamiento` y `Valorización`, usando los colores de la simbología de la capa.

#### Scenario: Distribución por categoría
- **WHEN** el gráfico de tratamiento se renderiza
- **THEN** muestra el tonelaje de las tres categorías para los establecimientos filtrados

#### Scenario: Filtro de tratamiento
- **WHEN** el usuario selecciona una categoría de tratamiento
- **THEN** el gráfico muestra el tonelaje de esa categoría y las demás quedan en cero o fuera del gráfico

#### Scenario: Coincidencia con la leyenda
- **WHEN** el usuario compara el gráfico con la leyenda de SINADER
- **THEN** cada categoría usa el mismo color en ambos lugares

### Requirement: Reactividad y estados vacíos
Los gráficos SHALL actualizarse con cada cambio de filtros, SHALL reflejar los filtros activos y SHALL mostrar un estado vacío explícito cuando no haya datos.

#### Scenario: Cambio de filtros
- **WHEN** el usuario cambia cualquier filtro
- **THEN** todos los gráficos afectados se actualizan en la misma interacción

#### Scenario: Limpiar filtros
- **WHEN** el usuario activa `Limpiar filtros`
- **THEN** los gráficos vuelven a los totales completos de ambas capas

#### Scenario: Sin datos
- **WHEN** los filtros activos dejan una capa sin resultados
- **THEN** el gráfico correspondiente muestra un mensaje de ausencia de datos en lugar de barras obsoletas

#### Scenario: Visibilidad de capa
- **WHEN** el usuario oculta una capa sin cambiar los filtros
- **THEN** los gráficos conservan los totales filtrados y no se vacían por la visibilidad

### Requirement: Accesibilidad y estilo Swiss
Los gráficos SHALL usar el sistema visual Swiss International, ofrecer etiquetas y descripciones accesibles, mantener contraste suficiente y adaptarse a los anchos de escritorio, tablet y móvil.

#### Scenario: Descripción accesible
- **WHEN** el usuario navega con lector de pantalla
- **THEN** cada gráfico expone un nombre y una descripción con las categorías y valores representados

#### Scenario: Contraste y textura
- **WHEN** el usuario observa los gráficos
- **THEN** usan superficies planas, bordes visibles, texto legible y colores de datos con contraste suficiente

#### Scenario: Anchos pequeños
- **WHEN** el visor se muestra a 375 px de ancho
- **THEN** los gráficos se apilan sin desbordamiento horizontal y conservan etiquetas legibles

#### Scenario: Movimiento reducido
- **WHEN** el sistema operativo informa preferencia por movimiento reducido
- **THEN** las transiciones decorativas de los gráficos se atenúan o desactivan sin impedir la actualización

### Requirement: Gráficos nativos sin dependencias
El sistema SHALL renderizar los gráficos con SVG nativo incluido en la aplicación, sin librerías de gráficos, servicios externos ni solicitudes de red adicionales.

#### Scenario: Sin librería externa
- **WHEN** el visor carga la sección de análisis
- **THEN** no se descarga ninguna librería de gráficos ni se realiza una petición de datos para los gráficos

#### Scenario: Ejecución client-side
- **WHEN** el usuario aplica filtros con el visor abierto
- **THEN** los gráficos se recalculan en el navegador a partir de los datos ya cargados
