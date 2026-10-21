# Roadmap

## Pantallas del MVP

1. **Login** con Google.
2. **Mapa:** pines de lugares y recorridos, con filtro por año y país. Botón "Estoy acá".
3. **Diario:** lista cronológica de visitas y recorridos.
4. **Detalle de lugar:** visitas, opiniones, fotos, agregar nuevas y "Abrir en Google Maps".
5. **Importar:** subir el archivo de Timeline o Takeout.
6. **Ajustes:** prender o apagar el tracking, exportar datos y borrar la cuenta.

## Fases

Se arranca por lo manual porque da algo usable rápido. El tracking automático es lo más delicado (batería y permisos de Android e iOS) y conviene sumarlo con la base ya probada.

### Fase 1: base
- [ ] Proyecto Expo + TypeScript + EAS
- [ ] Proyecto Supabase con PostGIS y migraciones en `supabase/migrations/`
- [ ] Tablas `places`, `visits`, `reviews` y `photos` con RLS
- [ ] Login con Google
- [ ] Mapa con Mapbox y pines de lugares
- [ ] Check-in manual: "Estoy acá" (GPS + lugares cercanos) o buscar un lugar y elegir la fecha
- [ ] Detalle de lugar: opiniones y fotos (comprimidas antes de subir)
- [ ] Diario cronológico

### Fase 2: importación
- [ ] Edge Function `import-timeline` para el formato actual del celular y para Takeout
- [ ] Reglas de duplicados al importar (fusión de visitas superpuestas)
- [ ] Tabla `trips` y dibujo de recorridos en el mapa
- [ ] Pantalla de importación con progreso y resumen

### Fase 3: tracking automático
- [ ] Tarea de ubicación en segundo plano (`expo-location` + `expo-task-manager`)
- [ ] Envío de `raw_points` en lotes, con cola offline
- [ ] Procesamiento de paradas → visitas y recorridos, y limpieza de puntos crudos
- [ ] Detección de rutas frecuentes (`route_group_id`)
- [ ] Ajustes para pausar el tracking

### Fase 4: pulido
- [ ] Estadísticas: países y ciudades visitados, km recorridos por año
- [ ] Mapa del mundo con los países visitados pintados
- [ ] Sugerencias desde fotos de la galería (EXIF)
- [ ] Exportar todos los datos y borrar la cuenta
- [ ] Web de solo lectura (opcional)

## Costos

| Servicio | Costo para uso personal |
|---|---|
| Supabase | Gratis (500 MB de base, 1 GB de fotos) |
| Mapbox | Gratis dentro de la cuota mensual de mapas y búsquedas |
| Android | Gratis (APK instalado directo) |
| iOS | USD 99 por año (Apple Developer) para instalar de forma estable |

## Verificación por fase

- **Fase 1:** login real en el celular, crear un check-in, subir una foto y confirmar con un segundo usuario que no ve nada del primero (RLS).
- **Fase 2:** tests de la Edge Function con un JSON de ejemplo de cada formato. Importar dos veces el mismo archivo no tiene que crear duplicados.
- **Fase 3:** tests de la detección de paradas con puntos sintéticos (misma parada dos veces, ruta repetida). Después, dejar la app un día en segundo plano y comparar las visitas detectadas contra lo real, mirando también el consumo de batería.
