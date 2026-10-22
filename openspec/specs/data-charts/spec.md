# data-charts Specification

## Purpose

Define la sección de análisis visual del panel lateral: su presencia, composición, reactividad, estados vacíos, accesibilidad y comportamiento sin datos externos. Los gráficos concretos, el estado compartido de filtros y las interacciones cruzadas pertenecen a `cross-filtered-dashboard`; esta capacidad no impone una tecnología de renderizado.

## Requirements

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

### Requirement: Actualización y estados vacíos
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

#### Scenario: Contraste y legibilidad
- **WHEN** el usuario observa los gráficos
- **THEN** usan superficies planas, bordes visibles, texto legible y colores de datos con contraste suficiente

#### Scenario: Anchos pequeños
- **WHEN** el visor se muestra a 375 px de ancho
- **THEN** los gráficos se apilan sin desbordamiento horizontal y conservan etiquetas legibles

#### Scenario: Movimiento reducido
- **WHEN** el sistema operativo informa preferencia por movimiento reducido
- **THEN** las transiciones decorativas de los gráficos se atenúan o desactivan sin impedir la actualización

### Requirement: Renderizado local sin datos externos
El sistema SHALL renderizar los gráficos en el navegador a partir de los datos ya embebidos, sin solicitar datasets ni enviar entidades a servidores. La sección SHALL seguir operativa mediante una alternativa accesible cuando una dependencia de gráficos no esté disponible.

#### Scenario: Sin petición de datos
- **WHEN** el visor carga o actualiza la sección de análisis
- **THEN** no se realiza ninguna petición de datasets para los gráficos

#### Scenario: Ejecución client-side
- **WHEN** el usuario aplica filtros con el visor abierto
- **THEN** los gráficos se recalculan en el navegador a partir de los datos ya cargados

#### Scenario: Dependencia no disponible
- **WHEN** la librería de gráficos no puede cargarse
- **THEN** la sección de análisis muestra una alternativa accesible y el resto del visor sigue operativo
