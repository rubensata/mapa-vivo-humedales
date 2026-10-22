# Spec Delta

## Purpose

Define la capa predefinida de humedales urbanos y su representación cartográfica según el proceso de declaración, para que el visor comunique de forma inmediata ambas categorías sin trabajo adicional del usuario.

## ADDED Requirements

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
