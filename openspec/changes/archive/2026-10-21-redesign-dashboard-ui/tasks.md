# Tasks

## 1. Base visual y tokens

- [x] 1.1 Cargar Inter con fallback Helvetica/Arial, mapear los colores `swiss` de Tailwind a variables CSS y definir en `:root` los tokens de color, tipografía, bordes, panel y espaciado; verificar en navegador que las variables resuelven, Inter o el fallback se aplica y no hay errores de consola.
- [x] 1.2 Implementar la base CSS y las texturas de grid, puntos, diagonales y ruido con opacidades bajas, junto con la escala tipográfica uppercase y las reglas de radio cero y sin sombras; verificar visualmente que las texturas se limitan a superficies de UI y que el canvas del mapa no las recibe.

## 2. Estructura estática

- [x] 2.1 Reestructurar el `<header>` como barra superior Swiss con título, badge, metadatos y borde negro; verificar a 1440, 768 y 375 px que no hay desbordes y que el texto conserva su contenido.
- [x] 2.2 Reubicar `#control-panel` a la izquierda a altura completa en escritorio y a ancho completo en móvil, agregar índices `01`–`06` y separadores, conservando `#panel-content`, `#toggle-panel-btn` e `#toggle-icon`; verificar por script que todos los ids existen y que el botón alterna `hidden` y `rotate-180`.
- [x] 2.3 Reestilizar las secciones de mapa base, filtros, carga local, capas, simbología e interacción con bordes, etiquetas y controles Swiss, conservando labels, `aria-label` y `data-basemap`; verificar que los cuatro mapas base cambian y que el estado activo se distingue tras cada cambio.
- [x] 2.4 Reestilizar `#drop-overlay` y `#toast-container`; verificar que arrastrar un archivo muestra el overlay, que al salir se oculta y que un toast de éxito, uno de advertencia y uno de error aparecen y desaparecen con su mensaje intacto.

## 3. Superficies dinámicas y MapLibre

- [x] 3.1 Aplicar CSS Swiss a las tarjetas generadas en `#layers-list` (nombre, conteo, swatch, checkbox, acciones); cargar un GeoJSON local y verificar que la tarjeta se ve consistente y que visibilidad, zoom y eliminación siguen funcionando.
- [x] 3.2 Aplicar CSS Swiss a las filas generadas en `#legend-content`, manteniendo los colores inline de las categorías; verificar que `Municipal` y `de Oficio` conservan sus colores y que la leyenda coincide con la simbología del mapa tras filtrar.
- [x] 3.3 Reestilizar `.maplibregl-popup-content`, el tip y el botón de cierre; verificar un popup de humedal y un popup genérico con atributos, enlaces y cierre, sin perder contenido ni legibilidad.
- [x] 3.4 Reestilizar los controles de MapLibre y los scrollbars; verificar zoom, brújula, geolocalización, foco por teclado y desplazamiento del panel en escritorio y móvil.

## 4. Accesibilidad y responsive

- [x] 4.1 Verificar contraste de texto normal, foco visible rojo de 2 px, nombres accesibles y áreas táctiles de al menos 44×44 px en móvil; corregir cualquier control que no cumpla y documentar la comprobación.
- [x] 4.2 Verificar la composición a 375, 768 y 1440 px: panel apilado o lateral según breakpoint, sin scroll horizontal, mapa utilizable y controles no cubiertos de forma permanente por el panel.
- [x] 4.3 Implementar y verificar `prefers-reduced-motion: reduce`: las transiciones decorativas se atenúan o desactivan y todas las acciones siguen operando.

## 5. Regresión funcional

- [x] 5.1 Registrar el hash del bloque `<script type="module">` antes de implementar y compararlo al final; verificar que no cambió ninguna función JavaScript ni la lógica de MapLibre.
- [x] 5.2 Ejecutar la suite CDP de filtros y capas (12 verificaciones), más carga local, popups, toasts, drag & drop, colapso del panel y cambio de los cuatro mapas base; verificar que todo pasa y que no hay errores de consola.
- [x] 5.3 Revisar cada escenario de `base-map`, `map-viewer`, `layer-management`, `layer-rendering` y `humedales-layer` en el navegador rediseñado; registrar evidencia de que siguen cumpliéndose.

## Workflow follow-up

- Sincronizar las specs principales y archivar el cambio después de la revisión.
- Verificar el resultado archivado.
