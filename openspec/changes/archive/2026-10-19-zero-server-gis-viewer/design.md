# Design

## Context

Véase `proposal.md` para la motivación y objetivos iniciales. El visor opera completamente en el cliente (Zero-Server) dentro de un único archivo [index.html](file:///c:/app_ia/curso/index.html), sin backend ni herramientas de compilación.

## Goals / Non-Goals

**Goals:**
- Implementar un visor web geoespacial en un único archivo [index.html](file:///c:/app_ia/curso/index.html).
- Implementar una estética visual de alto impacto (Sci-Fi / Cyberpunk Cartographic Glassmorphism) que maximice el contraste con los datos espaciales.
- Cargar MapLibre GL JS y Tailwind CSS vía CDN oficial.
- Utilizar mapas base OpenStreetMap vectoriales sin API key (OpenFreeMap) y satelitales (ESRI) sin bloqueos HTTP 403 ni marcas de agua.
- Carga e interactividad con datos GeoJSON (dataset de demostración + archivos locales del usuario mediante Drag & Drop / FileReader).
- Popups interactivos con inspección tabular de atributos.

**Non-Goals:**
- Servidores intermedios o bases de datos backend.
- Dependencias de empaquetado (Node.js, npm, bundlers).
- Formatos geoespaciales binarios pesados en esta fase.

## Decisions

### 1. Sistema de Diseño Visual: Void & Glassmorphism con Acentos Neón
- **Lienzo Base (Void)**: 
  - **Decisión**: Utilizar `bg-slate-950` como fondo base del cuerpo y contenedor del visor.
  - **Razón**: Crea el vacío oscuro absoluto necesario para que los vectores y geometrías espaciales resalten con máxima luminancia y legibilidad.
- **Paneles Translúcidos (Glassmorphism)**: 
  - **Decisión**: Exigir el efecto `backdrop-blur-md bg-white/5` combinado con bordes sutiles `border-white/10`.
  - **Razón**: Genera una apariencia de cristal esmerilado suspendido sobre el mapa cartográfico, integrando la UI con el entorno espacial sin bloquear la visualización subyacente.
- **Acentos de Luz (Neón Cyan)**: 
  - **Decisión**: Instruir el uso de resplandores `shadow-[0_0_15px_#06b6d4]` y toques cyan (`text-cyan-400`, `border-cyan-500/50`) en elementos interactivos, tarjetas activas y bordes destacados.
  - **Razón**: Inyecta resplandores de alto contraste de estilo cibernético / terminal táctica que guían la atención del usuario a los controles clave.

### 2. Proveedor de Mapas Base sin API Key (OpenFreeMap & ESRI)
- **Decisión**: Adoptar estilos vectoriales nativos de OpenFreeMap (Bright, Liberty, Positron) basados en OpenStreetMap, complementados con ESRI Satelital.
- **Razón**: Los servidores de voluntarios de `tile.openstreetmap.org` bloquean con error 403 peticiones desde `file:///` y CARTO exige API key comercial con marca de agua. OpenFreeMap es 100% libre, sin API key ni marcas de agua, y renderiza vectores nítidos mediante MapLibre.

### 3. Carga Local de GeoJSON (Client-side FileReader API)
- **Decisión**: Emplear la API nativa `FileReader` (`readAsText`) combinada con eventos `dragover`, `dragleave` y `drop` sobre la pantalla y selector `<input type="file">`.
- **Razón**: Privacidad absoluta para el usuario, cero latencia de red y soporte offline para sus archivos espaciales.

### 4. Estructura de Código Modular en un Solo Archivo
- **Decisión**: Agrupar constantes, configuración de estilos, instancia del mapa, registro de capas y manejadores de eventos dentro de un `<script type="module">` en [index.html](file:///c:/app_ia/curso/index.html).
- **Razón**: Satisface el principio KISS manteniendo alta legibilidad y facilidad de mantenimiento sin dependencias de compilación.

## Risks / Trade-offs

- **[Carga de fuentes/sprites remotos de estilos vectoriales]** → *Mitigación*: OpenFreeMap aloja glifos y sprites en CDN globales de alta velocidad.
- **[Contraste en mapas claros con paneles oscuros]** → *Mitigación*: El efecto `backdrop-blur-md bg-white/5` sobre `bg-slate-950` con acentos `shadow-[0_0_15px_#06b6d4]` asegura contraste tanto en mapa oscuro como en satelital y claro.

## Migration Plan

No se requiere migración. La actualización se refleja directamente en [index.html](file:///c:/app_ia/curso/index.html).
