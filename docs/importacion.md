# Importar el historial

## De dónde salen los datos

| Fuente | Cómo se obtiene | Formato |
|---|---|---|
| Google Maps Timeline (actual) | Desde 2024 Timeline vive en el celular. En la app de Maps: Ajustes → Tu Timeline → Exportar datos de Timeline | Un JSON con `semanticSegments` (visitas y actividades) y `rawSignals` |
| Google Takeout (anterior) | takeout.google.com → Historial de ubicaciones, si se exportó antes de que Google lo moviera al celular | Carpeta `Semantic Location History/<año>/<año>_<MES>.json` con `timelineObjects` (`placeVisit`, `activitySegment`) |
| Fotos de la galería | Ubicación EXIF de cada foto | Opcional, para sugerir "¿estuviste acá?" |

Los dos formatos de Google tienen la misma información: visitas a lugares con horario y, a veces, `placeId`, y recorridos entre ellas.

## Flujo

1. En la pantalla **Importar** elegís el archivo (o un ZIP de Takeout).
2. La app lo sube a Storage, en `imports/<user_id>/`.
3. Una **Edge Function** (`import-timeline`):
   - detecta el formato,
   - convierte cada visita en `{ lat, lng, arrived_at, left_at, google_place_id?, name? }` y cada recorrido en un `LineString`,
   - aplica las mismas reglas de duplicados que el tracking automático ([modelo de datos](modelo-datos.md#reglas-para-evitar-duplicados)),
   - procesa por lotes y guarda el avance, porque un historial de años puede tener decenas de miles de registros.
4. Al terminar, la app muestra un resumen: cuántos lugares nuevos, cuántas visitas y cuántas se fusionaron con datos existentes.

Importar dos veces el mismo archivo no duplica nada: las visitas que coinciden en lugar y horario se fusionan.

## Fotos de la galería (opcional)

Con permiso de galería, se leen fecha y ubicación EXIF. Si una foto cae dentro de una visita, se ofrece adjuntarla. Si no hay visita en ese momento y lugar, se ofrece crear una con `source = import`.
