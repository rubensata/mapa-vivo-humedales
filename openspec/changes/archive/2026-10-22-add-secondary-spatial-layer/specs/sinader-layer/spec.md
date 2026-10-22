# Spec Delta

## Purpose

Incorpora al visor una segunda capa espacial de establecimientos industriales SINADER 2024, agregada por establecimiento y separada de su simbología, para analizar el tonelaje declarado junto a los humedales urbanos sin depender de un backend.

## ADDED Requirements

### Requirement: Capa SINADER predefinida y activa tras el arranque
El sistema SHALL incluir una capa predefinida de establecimientos industriales SINADER 2024, agregada a partir de las declaraciones por `id_vu`, con un punto por establecimiento y visible al completarse la inicialización del visor.

#### Scenario: Inicio del visor con la capa SINADER
- **WHEN** MapLibre termina su carga inicial
- **THEN** la capa SINADER se muestra en el mapa con 8.906 puntos y aparece como una capa activa en el panel lateral

#### Scenario: Capa disponible sin carga manual
- **WHEN** el usuario abre el visor sin seleccionar ni arrastrar ningún archivo
- **THEN** la capa SINADER ya está disponible y puede alternarse desde el panel de capas

#### Scenario: Alternancia de visibilidad
- **WHEN** el usuario oculta y vuelve a mostrar la capa SINADER desde su tarjeta
- **THEN** los puntos desaparecen y reaparecen sin perder los filtros aplicados

### Requirement: Carga diferida después de MapLibre
El sistema SHALL registrar la capa SINADER únicamente después de que MapLibre haya completado su carga inicial, sin bloquear la capa de humedales ni la interacción con el mapa.

#### Scenario: Espera a que MapLibre esté listo
- **WHEN** el visor todavía no completó la carga de MapLibre
- **THEN** no existen fuentes ni capas SINADER registradas en el mapa

#### Scenario: Registro al completar la carga
- **WHEN** MapLibre emite su evento de carga inicial
- **THEN** el sistema registra la fuente y las capas SINADER y las deja visibles

#### Scenario: Arranque no bloqueado
- **WHEN** el usuario abre el visor
- **THEN** la capa de humedales y los controles están disponibles antes de que termine el registro de SINADER

### Requirement: Agregación e integridad del dataset
El sistema SHALL agregar las 120.843 declaraciones por establecimiento, sumando tonelaje y registros, y SHALL excluir los registros con coordenadas inválidas o fuera de Chile sin afectar a los demás establecimientos.

#### Scenario: Una entidad por establecimiento
- **WHEN** se construye la capa SINADER
- **THEN** cada `id_vu` se representa con un único punto y sus declaraciones se consolidan en un solo elemento

#### Scenario: Sumas conservadas
- **WHEN** el usuario consulta un establecimiento
- **THEN** su tonelaje total y su cantidad de registros coinciden con la suma de sus declaraciones incluidas

#### Scenario: Registros descartados
- **WHEN** una declaración tiene coordenadas inválidas o fuera del territorio chileno
- **THEN** esa declaración no se dibuja, pero los registros válidos del mismo establecimiento y del resto del dataset se conservan

#### Scenario: Totales del dataset depurado
- **WHEN** se audita la capa construida
- **THEN** contiene 8.906 establecimientos, 120.767 registros y 5.634.872,476 toneladas, y no incluye geometrías inválidas

### Requirement: Separación entre datos y simbología
Los datos de la capa SHALL contener solo geometría y atributos; la simbología SHALL declararse por separado y ser la única fuente de colores para el mapa, la leyenda y los gráficos.

#### Scenario: Datos sin estilo
- **WHEN** se inspecciona el dataset de la capa SINADER
- **THEN** sus propiedades no contienen colores, radios ni expresiones de estilo

#### Scenario: Coherencia mapa-leyenda-gráficos
- **WHEN** el usuario compara un color de tratamiento en el mapa, la leyenda y los gráficos
- **THEN** los tres usan la misma entrada de la configuración de simbología

#### Scenario: Cambio de estilo sin alterar datos
- **WHEN** el usuario cambia el mapa base
- **THEN** la capa conserva sus atributos y solo se reconstruye su presentación

### Requirement: Simbología por tratamiento primario dominante
El sistema SHALL simbolizar cada establecimiento según su tratamiento primario dominante, con tres categorías distintas y contraste suficiente en los cuatro mapas base.

#### Scenario: Establecimiento de eliminación
- **WHEN** el tratamiento con mayor tonelaje de un establecimiento es `Eliminación`
- **THEN** su punto usa el color de la categoría de eliminación

#### Scenario: Establecimiento de recepción y almacenamiento
- **WHEN** el tratamiento con mayor tonelaje es `Recepción y Almacenamiento`
- **THEN** su punto usa el color de la categoría de recepción y almacenamiento

#### Scenario: Establecimiento de valorización
- **WHEN** el tratamiento con mayor tonelaje es `Valorización`
- **THEN** su punto usa el color de la categoría de valorización

#### Scenario: Persistencia de la simbología
- **WHEN** el usuario cambia entre los cuatro mapas base
- **THEN** la capa SINADER conserva la simbología de las tres categorías y sigue visible

### Requirement: Leyenda conjunta de capas
El sistema SHALL mostrar en el panel una leyenda de SINADER con las tres categorías de tratamiento y sus colores, junto a la leyenda de humedales urbanos.

#### Scenario: Categorías SINADER en la leyenda
- **WHEN** el usuario observa la sección de simbología
- **THEN** aparecen las categorías `Eliminación`, `Recepción y Almacenamiento` y `Valorización` con muestras de color

#### Scenario: Coincidencia con el mapa
- **WHEN** el usuario compara la leyenda con los puntos del mapa
- **THEN** cada categoría de tratamiento usa el mismo color en ambos lugares

#### Scenario: Coexistencia con humedales
- **WHEN** ambas capas están visibles
- **THEN** la leyenda identifica por separado la simbología de humedales y la de SINADER

### Requirement: Popup de establecimiento
Al hacer clic en un establecimiento, el sistema SHALL abrir un popup con nombre o razón social, RUT, región, comuna, rubro, tonelaje total, cantidad de registros, tratamiento dominante y principales capítulos LER.

#### Scenario: Clic sobre un punto SINADER
- **WHEN** el usuario hace clic sobre un establecimiento
- **THEN** el popup muestra los atributos del establecimiento y el resumen de sus declaraciones agregadas

#### Scenario: Campos vacíos
- **WHEN** el establecimiento no tiene nombre, RUT o rubro
- **THEN** el popup muestra un marcador de ausencia de dato sin romper la tabla de atributos

#### Scenario: Popup tras cambiar de mapa base
- **WHEN** el usuario cambia el mapa base y hace clic en un punto SINADER
- **THEN** el popup se abre con los mismos atributos y la capa sigue visible

### Requirement: Consulta de establecimientos coubicados
Cuando varios establecimientos comparten la misma coordenada, el sistema SHALL mostrar en el popup todos los establecimientos de esa ubicación para que ninguno quede inaccesible.

#### Scenario: Clic en una coordenada compartida
- **WHEN** el usuario hace clic en una ubicación ocupada por más de un establecimiento
- **THEN** el popup lista todos los establecimientos coubicados con su nombre, tonelaje y tratamiento dominante

#### Scenario: Establecimientos coubicados visibles
- **WHEN** el usuario recorre la lista de coincidencias
- **THEN** puede consultar los datos de cada establecimiento sin que los puntos se superpongan de forma irreversible

#### Scenario: Sin pérdida de entidades
- **WHEN** se comparan los puntos del mapa con el dataset agregado
- **THEN** los 8.906 establecimientos siguen representados aunque 220 coordenadas sean compartidas

### Requirement: Filtros compartidos de región y comuna
El sistema SHALL aplicar los filtros de región y comuna a las capas de humedales y SINADER de forma simultánea, sin alterar el subconjunto de humedales que cumple los filtros de humedales.

#### Scenario: Filtrar por región
- **WHEN** el usuario selecciona una región distinta de `Todas`
- **THEN** ambas capas muestran solo las entidades cuya `region` coincide con la seleccionada

#### Scenario: Filtrar por comuna
- **WHEN** el usuario selecciona una comuna distinta de `Todas`
- **THEN** ambas capas muestran solo las entidades cuya `comuna` coincide con la seleccionada

#### Scenario: Combinar filtros compartidos
- **WHEN** el usuario tiene región y comuna activas
- **THEN** ambas capas muestran solo las entidades que cumplen simultáneamente ambos filtros

### Requirement: Filtros propios de SINADER
El sistema SHALL ofrecer filtros por rubro y por tratamiento primario para la capa SINADER, con una opción para no filtrar, y SHALL combinarlos con los filtros compartidos.

#### Scenario: Filtrar por rubro
- **WHEN** el usuario selecciona un rubro distinto de `Todos`
- **THEN** la capa SINADER muestra solo los establecimientos de ese rubro

#### Scenario: Filtrar por tratamiento
- **WHEN** el usuario selecciona una categoría de tratamiento
- **THEN** la capa SINADER muestra los establecimientos que registran tonelaje en esa categoría, aunque no sea su tratamiento dominante

#### Scenario: Combinar filtros propios y compartidos
- **WHEN** el usuario combina una región, un rubro y un tratamiento
- **THEN** la capa SINADER muestra solo los establecimientos que cumplen los tres filtros

#### Scenario: Filtros sin resultados
- **WHEN** la combinación de filtros no devuelve establecimientos SINADER
- **THEN** la capa SINADER queda vacía y el sistema informa que no hay resultados para SINADER

#### Scenario: Limpiar filtros
- **WHEN** el usuario activa la acción `Limpiar filtros`
- **THEN** los filtros compartidos y los propios vuelven a su opción sin filtrar, ambas capas recuperan todos sus datos y el mapa regresa a la vista inicial de Chile con zoom 4

### Requirement: Opciones de filtro derivadas de ambos datasets
El sistema SHALL construir las opciones de región y comuna a partir de los valores presentes en humedales y SINADER, y las de proceso, rubro y tratamiento a partir de los valores de su capa correspondiente, sin duplicados y ordenadas alfabéticamente.

#### Scenario: Regiones y comunas combinadas
- **WHEN** el usuario abre los selectores de región o comuna
- **THEN** las opciones incluyen los valores de ambas capas sin duplicados

#### Scenario: Cascada de comunas
- **WHEN** el usuario selecciona una región
- **THEN** el selector de comuna se limita a las comunas de esa región presentes en cualquiera de las dos capas

#### Scenario: Filtros exclusivos de capa
- **WHEN** el usuario abre los selectores de proceso, rubro o tratamiento
- **THEN** proceso ofrece solo valores de humedales y rubro y tratamiento solo valores de SINADER

### Requirement: Contador y cámara de SINADER
El sistema SHALL mostrar la cantidad de establecimientos que cumplen los filtros y SHALL ajustar la cámara a sus resultados cuando cambien los filtros propios de SINADER.

#### Scenario: Actualización del contador
- **WHEN** el usuario cambia un filtro compartido o propio de SINADER
- **THEN** el contador de establecimientos refleja de inmediato la cantidad visible

#### Scenario: Ajuste de cámara a SINADER
- **WHEN** el usuario cambia un filtro de rubro o tratamiento y existen establecimientos coincidentes
- **THEN** el mapa ajusta su extensión a los establecimientos visibles con un margen de padding

#### Scenario: Sin resultados SINADER
- **WHEN** un filtro propio de SINADER no devuelve establecimientos
- **THEN** la cámara se mantiene y se muestra una notificación de advertencia

### Requirement: Persistencia de la capa SINADER
El sistema SHALL mantener los filtros aplicados y el subconjunto visible de SINADER al cambiar de mapa base o al alternar la visibilidad de la capa.

#### Scenario: Cambio de mapa base con filtros activos
- **WHEN** el usuario cambia de mapa base mientras hay filtros aplicados
- **THEN** la capa SINADER se reconstruye mostrando el mismo subconjunto filtrado y la cámara se conserva

#### Scenario: Visibilidad con filtros activos
- **WHEN** el usuario oculta y vuelve a mostrar la capa SINADER con filtros aplicados
- **THEN** la capa reaparece con el mismo subconjunto filtrado

### Requirement: Ejecución client-side y tolerancia a fallos
El sistema SHALL obtener la capa desde datos incluidos con la aplicación, sin solicitudes de red para el dataset ni servicios backend, y SHALL mantener las demás capacidades si esos datos faltan o son inválidos.

#### Scenario: Dataset sin backend
- **WHEN** el visor carga la capa SINADER
- **THEN** no se solicita el archivo Excel ni un servicio externo para obtener sus datos

#### Scenario: Datos ausentes o corruptos
- **WHEN** el bloque de datos SINADER falta o no puede interpretarse
- **THEN** el sistema muestra una notificación de error y mantiene operativos los humedales, los mapas base, la carga local y los popups existentes

#### Scenario: Sin errores de consola
- **WHEN** la capa SINADER se registra, filtra, oculta y vuelve a mostrar
- **THEN** la consola no registra errores nuevos
