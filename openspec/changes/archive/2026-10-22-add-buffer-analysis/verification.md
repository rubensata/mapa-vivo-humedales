# Verification

Registro de la verificación funcional del cambio `add-buffer-analysis`, ejecutada con Chrome headless (MapLibre GL JS 4.7.1 + SwiftShader) desde `file://`, sin servidor local.

## Mediciones en navegador

| Operación | Entidades | Tiempo | Salida | Observaciones |
| --- | ---: | ---: | ---: | --- |
| Buffer individual filtrado (Antofagasta, 1.000 m) | 402 | 1,6–1,9 s | 188 KB GeoJSON | Incluye carga inicial de Turf y creación del Worker. |
| Dissolve filtrado (Antofagasta, 1.000 m) | 402 | 2,9 s | 51 KB ZIP / 189 KB GeoJSON | 1 geometría disjunta de varias partes. |
| Buffer individual completo (1.000 m) | 9.013 | 10,7 s | 14,2 MB GeoJSON | Interfaz con 173 ticks de temporizador durante el cálculo. |
| Dissolve completo (1.000 m) | 9.013 | 71,3 s | 3,65 MB GeoJSON | 91.659 vértices; 1 `MultiPolygon`; 710 ticks de temporizador. |
| Dissolve filtrado (RM, 1.000 m) | 3.742 | 65,7 s | — | El costo lo domina el dissolve, no la cantidad de entidades. |

Desviaciones frente a la exploración del design:

- Individual completo: 10,7 s frente a ~11 s (sin desviación relevante).
- Dissolve completo: 71,3 s frente a ~54 s (+32 %), atribuible al navegador, SwiftShader y al entorno headless; se mantiene dentro del rango de "decenas de segundos" previsto.
- Disolve GeoJSON: 3,65 MB / 91.659 vértices frente a 3,27 MB / 82.821 vértices (+12 %); el resultado sigue siendo manejable.
- Individual GeoJSON: 14,2 MB frente a 16,3 MB (-13 %), por la agregación real de los datos de origen.

## Verificaciones ejecutadas

- **Sección y DOM**: la sección `09 Análisis de buffer` queda dentro de `#panel-content`, después de `08 Exportar`; `type="number"` con `min`, `max`, `step`, valor inicial 1.000; radios de alcance, casilla de dissolve, botones y exportaciones con los identificadores previstos.
- **Validación**: vacío, texto, decimal, negativo, 0, 1, 50.000 y 50.001; solo los valores enteros entre 1 y 50.000 habilitan la ejecución. Un envío inválido no modifica el resultado existente.
- **Alcance**: dataset completo = 9.013 entidades (107 humedales + 8.906 establecimientos) con independencia de filtros, capa SINADER oculta y capas locales; solo filtradas = 402 entidades con el filtro Antofagasta.
- **Snapshot**: cambiar filtros durante el cálculo no altera el resultado en curso (9.013 entidades analizadas y contador de filtros actualizado en paralelo).
- **Individual**: 9.013 geometrías poligonales; el span de latitud para buffers de puntos a 1.000 m es 0,018° y a 2.000 m es 0,036°, coherente con el diámetro esperado; las geometrías superpuestas se conservan separadas.
- **Dissolve**: una sola geometría `MultiPolygon` con varias partes disjuntas y atributos `buffer_m`, `modo`, `origen` (`mixto`) y `n_fuentes` (9.013); sin bordes internos por la unión de Turf.
- **Progreso**: se observaron mensajes de preparación, avance de buffers por cada 100 entidades y avance del dissolve por lote (16 lotes para 3.742 entidades).
- **Interfaz utilizable**: temporizadores y cambios de filtro funcionaron durante cálculos completos; el Worker no bloqueó el hilo principal.
- **Ciclo de vida**: cambiar filtros después del análisis conserva la capa y sus métricas; una nueva ejecución reemplaza el resultado; limpiar elimina la capa y deshabilita las exportaciones; la capa de análisis persiste al cambiar de mapa base (se reinserta sobre las capas de datos).
- **Visibilidad**: alternar la casilla de visibilidad cambia el render sin afectar las capas de datos, filtros ni contadores.
- **Exportaciones**: GeoJSON se parsea como `FeatureCollection` en WGS84; el ZIP contiene `areas_influencia.shp`, `.shx`, `.dbf` y `.prj` (WGS84), con campos DBF `buffer_m`, `modo`, `origen`, `fuente_id`, `fuente`, `n_fuentes`, únicos y de hasta 10 caracteres; los nombres de archivo incluyen alcance, distancia y marca temporal.
- **Errores**: con Turf devolviendo 404 el estado muestra un error comprensible, no se publica resultado y los controles se restauran; un fallo de `shpwrite` muestra error, restaura botones y conserva el resultado, y GeoJSON sigue operativo. Reintentar tras desbloquear la dependencia funciona.
- **Red y Zero-Server**: sin solicitudes `POST`; ninguna URL incluye datos de buffer; Turf no se solicita al arrancar (0 solicitudes), se carga una sola vez en el primer análisis y el Worker se reutiliza.
- **Accesibilidad y respuesta visual**: `role="status"` con `aria-live="polite"`, `aria-busy` en la sección, etiquetas explícitas, foco visible, error de distancia en línea, advertencia de dissolve visible solo con dataset completo + dissolve, tabulación operativa y análisis completado con `prefers-reduced-motion: reduce`. Sin overflow horizontal a 375, 768 y 1440 px.
- **errores de consola**: ninguno fuera de los `console.error` intencionales de las pruebas de exportación fallida.
