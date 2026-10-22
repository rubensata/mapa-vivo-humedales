# Spec Delta

## Purpose

Garantiza que todas las geometrías GeoJSON cargadas se rendericen correctamente y permanezcan visibles y consultables al cambiar el estilo del mapa base.

## ADDED Requirements

### Requirement: Renderizado de todos los tipos de geometría
El sistema SHALL renderizar features de tipo `Point`, `LineString`, `Polygon`, `MultiPoint`, `MultiLineString` y `MultiPolygon` sin errores de validación de filtros.

#### Scenario: Capa demo inicial
- **WHEN** el visor carga la capa de demostración
- **THEN** sus seis puntos se dibujan en el mapa y no aparecen errores de validación de filtros

#### Scenario: Capa MultiPolygon del usuario
- **WHEN** el usuario carga un GeoJSON con geometrías `MultiPolygon`, como `Humedales_Urbanos.geojson`
- **THEN** los polígonos se dibujan en el mapa y no aparecen errores de validación de filtros

### Requirement: Persistencia de capas ante cambios de mapa base
El sistema SHALL mantener visibles las capas de datos cargadas después de cambiar el estilo del mapa base.

#### Scenario: Cambio de mapa base
- **WHEN** el usuario cambia de OSM Dark a OSM Bright o Satelital
- **THEN** la capa demo y las capas cargadas por el usuario siguen visibles sin recargar la página

#### Scenario: Retorno al mapa base oscuro
- **WHEN** el usuario vuelve a OSM Dark
- **THEN** las capas de datos se mantienen visibles y el fondo oscuro se conserva

### Requirement: Inspección de atributos sobre capas renderizadas
El sistema SHALL abrir un popup con las propiedades al hacer clic en una geometría renderizada, incluso después de cambiar el mapa base.

#### Scenario: Clic en un punto demo
- **WHEN** el usuario hace clic en un punto de la capa demo
- **THEN** se abre un popup con sus atributos

#### Scenario: Clic en un polígono tras cambiar el mapa base
- **WHEN** el usuario cambia el mapa base y hace clic en un polígono cargado que sigue visible
- **THEN** se abre un popup con los atributos del polígono
