# Spec Delta

## MODIFIED Requirements

### Requirement: Instanciación sobre el contenedor existente
El sistema SHALL inicializar MapLibre GL JS utilizando el elemento `#map` ya presente en la interfaz, sin crear contenedores de mapa adicionales; el sistema de diseño SHALL poder reorganizar la composición visual de los componentes existentes sin recrear ni sustituir el contenedor del mapa.

#### Scenario: Arranque del visor
- **WHEN** el usuario abre `index.html` en un navegador
- **THEN** el lienzo de MapLibre se renderiza dentro del elemento `#map` existente y no se crean contenedores de mapa adicionales

#### Scenario: Composición visual actualizada
- **WHEN** se aplica el sistema de diseño Swiss International
- **THEN** `#map` sigue siendo el único contenedor del mapa y los componentes reorganizados conservan sus identificadores y funciones

### Requirement: Preservación de los componentes existentes
El sistema SHALL mantener sin cambios la funcionalidad, los identificadores y los contratos de interacción del panel lateral y de los demás componentes existentes, y SHALL permitir que su estructura y presentación visual se actualicen mediante el sistema de diseño.

#### Scenario: Interfaz intacta tras la instanciación
- **WHEN** el usuario abre el visor
- **THEN** el panel lateral, el selector de mapas base, la lista de capas y los controles se muestran y funcionan igual que antes del cambio

#### Scenario: Contratos de interacción preservados
- **WHEN** el usuario opera filtros, capas, carga local, popups o mapas base
- **THEN** los identificadores y atributos usados por el visor se conservan, y las interacciones producen el mismo resultado
