# Spec Delta

## Purpose

Proporciona un visor cartográfico web interactivo que renderiza el mapa base estándar de OpenStreetMap y controles de navegación en el navegador sin backend.

## ADDED Requirements

### Requirement: Renderizado de mapa base OpenStreetMap
El sistema SHALL renderizar un mapa interactivo a pantalla completa utilizando mosaicos raster estándar de OpenStreetMap sin requerir registro ni API keys.

#### Scenario: Carga inicial del mapa
- **WHEN** el usuario abre la aplicación en el navegador
- **THEN** el sistema inicializa el lienzo MapLibre y muestra los mosaicos de OpenStreetMap en una vista predeterminada

### Requirement: Controles de navegación y orientación
El sistema SHALL ofrecer controles en pantalla para zoom in, zoom out, rotación hacia el norte y geolocalización del usuario.

#### Scenario: Uso del control de zoom
- **WHEN** el usuario hace clic en el botón de aumento (+)
- **THEN** el nivel de zoom del mapa se incrementa suavemente

#### Scenario: Restablecimiento de orientación norte
- **WHEN** el usuario hace clic en la brújula de orientación
- **THEN** el mapa rota suavemente para alinear el norte hacia la parte superior de la pantalla
