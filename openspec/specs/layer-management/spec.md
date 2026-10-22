# layer-management Specification

## Purpose
Permite gestionar, alternar y consultar atributos de capas vectoriales en formato GeoJSON, tanto predefinidas en el sistema como suministradas por el usuario.

## Requirements

### Requirement: Carga de archivos GeoJSON locales
El sistema SHALL permitir al usuario cargar archivos GeoJSON directamente desde su equipo mediante un selector de archivos o arrastrándolos sobre la ventana del visor.

#### Scenario: Carga exitosa de archivo GeoJSON
- **WHEN** el usuario selecciona o arrastra un archivo GeoJSON válido
- **THEN** el sistema lee el archivo en el cliente, agrega la capa al mapa y ajusta la extensión visual (fitBounds) a las geometrías importadas

#### Scenario: Archivo GeoJSON inválido
- **WHEN** el usuario proporciona un archivo con formato JSON incorrecto o sin estructura GeoJSON
- **THEN** el sistema muestra una notificación de error clara en la interfaz sin interrumpir la visualización actual

### Requirement: Inspección de atributos mediante popups
El sistema SHALL mostrar una ventana emergente (popup) con las propiedades y atributos de un elemento geométrico al hacer clic sobre él.

#### Scenario: Clic en elemento espacial
- **WHEN** el usuario hace clic sobre un punto o polígono en el mapa
- **THEN** el sistema abre un popup centrado en la geometría mostrando una tabla legible con sus atributos
