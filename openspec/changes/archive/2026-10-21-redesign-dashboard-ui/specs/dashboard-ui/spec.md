# Spec Delta

## Purpose

Define la identidad visual Swiss International y la composición responsive de la interfaz del visor, para que controles, datos y estados se comuniquen con claridad objetiva sin alterar el comportamiento cartográfico.

## ADDED Requirements

### Requirement: Identidad visual Swiss International
La interfaz SHALL usar exclusivamente blanco, negro, gris claro y rojo Swiss `#FF3000` como señal funcional, con superficies planas, esquinas rectas, bordes negros visibles y sin sombras ni degradados decorativos.

#### Scenario: Paleta funcional
- **WHEN** el usuario observa la interfaz
- **THEN** el rojo aparece solo en énfasis, estados activos, foco o llamadas a la acción, y el resto de la interfaz se mantiene en blanco, negro y gris

#### Scenario: Superficies planas
- **WHEN** el usuario recorre header, panel, tarjetas y controles
- **THEN** no hay sombras difusas, glassmorphism ni esquinas redondeadas, y la estructura se percibe mediante bordes y líneas

### Requirement: Tipografía estructural
La interfaz SHALL usar una tipografía grotesca sans-serif con jerarquía marcada, titulares y etiquetas en mayúsculas, alineación a la izquierda y contraste extremo entre títulos y texto de apoyo.

#### Scenario: Jerarquía tipográfica
- **WHEN** el usuario lee el header y las secciones del panel
- **THEN** los títulos y etiquetas se muestran en mayúsculas con tracking definido, y el texto de apoyo mantiene legibilidad sin competir con ellos

#### Scenario: Etiquetado numerado
- **WHEN** el usuario recorre las secciones del panel
- **THEN** cada sección principal muestra un prefijo numerado visible y consistente

### Requirement: Composición de grid y panel
La interfaz SHALL organizar el contenido sobre un grid visible y asimétrico, con el panel de control a la izquierda a altura completa en escritorio y apilado a ancho completo en móvil, sin cubrir de forma permanente los controles esenciales del mapa.

#### Scenario: Composición de escritorio
- **WHEN** el usuario abre el visor en una pantalla de escritorio
- **THEN** el panel se ubica a la izquierda con bordes que separan sus secciones y el mapa permanece visible y utilizable en el resto del viewport

#### Scenario: Composición móvil
- **WHEN** el usuario abre el visor en un ancho menor a 768 px
- **THEN** el panel se presenta a ancho completo, las secciones se apilan y los controles mantienen áreas táctiles utilizables

#### Scenario: Colapsar y expandir
- **WHEN** el usuario activa el control de colapso del panel
- **THEN** el contenido del panel se oculta o muestra sin perder el estado de filtros, capas ni mapas base

### Requirement: Texturas de profundidad
La interfaz SHALL incorporar texturas sutiles de grid, puntos, diagonales y ruido en superficies de la UI, sin aplicarlas al lienzo del mapa ni reducir la legibilidad.

#### Scenario: Textura sobre superficies de UI
- **WHEN** el usuario observa el header y las secciones del panel
- **THEN** las texturas aportan profundidad con baja opacidad y no interfieren con la lectura de etiquetas ni valores

#### Scenario: Lienzo del mapa sin textura
- **WHEN** el usuario observa el mapa
- **THEN** las texturas del sistema visual no se dibujan sobre el lienzo ni alteran la visualización de las capas

### Requirement: Estados interactivos bold
La interfaz SHALL comunicar interactividad mediante inversión de color, cambio a rojo, desplazamiento o rotación, con transiciones breves y sin depender de sombras o cambios sutiles de opacidad.

#### Scenario: Hover en controles
- **WHEN** el usuario pasa el puntero sobre un botón o una tarjeta interactiva
- **THEN** el control cambia de fondo o color de forma inmediata y visible

#### Scenario: Mapa base activo
- **WHEN** el usuario cambia de mapa base
- **THEN** el botón seleccionado se distingue con el tratamiento Swiss de estado activo y los demás vuelven a su estado inactivo

#### Scenario: Foco de teclado
- **WHEN** el usuario navega con teclado
- **THEN** el elemento enfocado muestra un indicador de foco rojo de alto contraste y visible

### Requirement: Superficies dinámicas consistentes
Las tarjetas de capas, la leyenda, los popups y las notificaciones SHALL adoptar la identidad Swiss International conservando su contenido, sus datos y sus acciones.

#### Scenario: Tarjeta de capa
- **WHEN** el usuario carga o alterna una capa
- **THEN** su tarjeta muestra nombre, conteo, color, visibilidad y acciones con el tratamiento Swiss, y las acciones siguen funcionando

#### Scenario: Leyenda de humedales
- **WHEN** el usuario consulta la leyenda
- **THEN** las categorías, colores y valores se muestran con el tratamiento Swiss y coinciden con la simbología del mapa

#### Scenario: Popup de atributos
- **WHEN** el usuario hace clic en una geometría
- **THEN** el popup presenta los mismos atributos y enlaces con tipografía y bordes Swiss

#### Scenario: Notificación toast
- **WHEN** el sistema muestra una notificación de éxito, error, advertencia o información
- **THEN** la notificación usa el tratamiento Swiss y se distingue por tipo sin perder su mensaje ni su duración

### Requirement: Accesibilidad del sistema visual
La interfaz SHALL mantener contraste alto, foco visible, etiquetas accesibles, áreas táctiles mínimas de 44 por 44 píxeles en móvil y respeto por la preferencia de movimiento reducido.

#### Scenario: Contraste y foco
- **WHEN** el usuario navega con teclado o lector de pantalla
- **THEN** los controles conservan nombres accesibles, orden lógico y foco visible sobre cualquier superficie

#### Scenario: Movimiento reducido
- **WHEN** el sistema operativo informa preferencia por movimiento reducido
- **THEN** las animaciones decorativas y los desplazamientos se atenúan o desactivan sin afectar la operación

### Requirement: Preservación funcional durante el rediseño
El rediseño SHALL conservar todos los identificadores, ganchos de interacción, textos funcionales y comportamientos existentes del visor, sin modificar la lógica de MapLibre ni las funciones JavaScript.

#### Scenario: Controles existentes
- **WHEN** el usuario interactúa con filtros, mapas base, carga local, capas, leyenda o popups
- **THEN** cada control responde igual que antes del rediseño y mantiene sus identificadores y atributos de enganche

#### Scenario: Regresión funcional
- **WHEN** se ejecutan las verificaciones de las specs funcionales existentes
- **THEN** todas siguen cumpliéndose y la consola no registra errores nuevos
