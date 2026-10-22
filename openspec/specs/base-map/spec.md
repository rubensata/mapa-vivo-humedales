# base-map Specification

## Purpose

Define cómo el visor instancia y configura su mapa base para ofrecer una vista inicial consistente sobre el contenedor existente y sin alterar la interfaz ya construida.

## Requirements

### Requirement: Instanciación sobre el contenedor existente
El sistema SHALL inicializar MapLibre GL JS utilizando el elemento `#map` ya presente en la interfaz, sin crear contenedores de mapa adicionales; el sistema de diseño SHALL poder reorganizar la composición visual de los componentes existentes sin recrear ni sustituir el contenedor del mapa.

#### Scenario: Arranque del visor
- **WHEN** el usuario abre `index.html` en un navegador
- **THEN** el lienzo de MapLibre se renderiza dentro del elemento `#map` existente y no se crean contenedores de mapa adicionales

#### Scenario: Composición visual actualizada
- **WHEN** se aplica el sistema de diseño Swiss International
- **THEN** `#map` sigue siendo el único contenedor del mapa y los componentes reorganizados conservan sus identificadores y funciones

### Requirement: Mapa base oscuro sin credenciales
El sistema SHALL cargar un mapa base de estilo oscuro que no requiera API key ni credenciales del usuario.

#### Scenario: Carga inicial del mapa base
- **WHEN** el visor termina de inicializarse
- **THEN** el mapa muestra el estilo oscuro como mapa base y no se solicita ninguna API key ni credencial

### Requirement: Vista inicial centrada en Chile
El sistema SHALL iniciar la cámara del mapa centrada en Chile con un nivel de zoom de 4.

#### Scenario: Cámara inicial
- **WHEN** el usuario abre el visor
- **THEN** el centro del mapa se ubica en Chile y el nivel de zoom inicial es exactamente 4

### Requirement: Ocupación del espacio disponible
El sistema SHALL mostrar el mapa ocupando todo el espacio disponible del viewport y adaptar el lienzo cuando cambie el tamaño de la ventana.

#### Scenario: Redimensionamiento de la ventana
- **WHEN** el usuario cambia el tamaño de la ventana del navegador
- **THEN** el mapa se redimensiona y sigue ocupando el espacio disponible sin recortes ni áreas vacías

### Requirement: Preservación de los componentes existentes
El sistema SHALL mantener sin cambios la funcionalidad, los identificadores y los contratos de interacción del panel lateral y de los demás componentes existentes, y SHALL permitir que su estructura y presentación visual se actualicen mediante el sistema de diseño.

#### Scenario: Interfaz intacta tras la instanciación
- **WHEN** el usuario abre el visor
- **THEN** el panel lateral, el selector de mapas base, la lista de capas y los controles se muestran y funcionan igual que antes del cambio

#### Scenario: Contratos de interacción preservados
- **WHEN** el usuario opera filtros, capas, carga local, popups o mapas base
- **THEN** los identificadores y atributos usados por el visor se conservan, y las interacciones producen el mismo resultado
