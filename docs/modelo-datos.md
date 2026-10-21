# Modelo de datos

La idea central es separar los **lugares**, que aparecen una sola vez, de las **visitas**, una por cada vez que fuiste. Así, ir 200 veces al mismo bar es un lugar con 200 visitas, no 200 pines.

## Tablas

Todas tienen `id uuid`, `user_id uuid` (referencia a `auth.users`) y `created_at`.

| Tabla | Columnas principales | Para qué |
|---|---|---|
| `places` | `name`, `location geography(Point)`, `radius_m`, `category`, `country_code`, `city`, `mapbox_id`, `google_place_id` | Lugar único |
| `visits` | `place_id`, `arrived_at`, `left_at`, `source` (`auto` / `manual` / `import`) | Cada vez que estuviste en un lugar |
| `trips` | `from_visit_id`, `to_visit_id`, `path geography(LineString)`, `mode` (auto, a pie, bici…), `distance_m`, `started_at`, `ended_at`, `route_group_id` | Recorrido entre dos visitas |
| `reviews` | `place_id`, `visit_id` (opcional), `rating` 1–5, `body` | Opinión |
| `photos` | `storage_path`, `place_id`, `visit_id`, `taken_at`, `location` | Foto subida a Storage |
| `raw_points` | `location geography(Point)`, `recorded_at`, `accuracy_m`, `speed` | Puntos GPS crudos. Se borran al procesarlos |

Índices: GiST en `places.location`, `trips.path` y `raw_points.location`; B-tree en `visits(user_id, arrived_at)`.

Se usa `geography` y no `geometry` para que las distancias salgan directo en metros.

## Seguridad (RLS)

Cada tabla lleva la misma política:

```sql
alter table places enable row level security;

create policy "solo mis lugares" on places
  for all using (user_id = auth.uid()) with check (user_id = auth.uid());
```

En Storage, las fotos van en `photos/<user_id>/...`, con una política que solo permite leer y escribir dentro de la carpeta propia.

## Reglas para evitar duplicados

### 1. Puntos GPS → visitas (detección de paradas)

Si los puntos se quedan durante **5 minutos o más** dentro de un radio de **~100 m**, eso es una parada. Se usa un algoritmo de *stay-point detection*: se recorren los puntos en orden y, mientras el siguiente siga dentro del radio del primero, se acumulan; al salir, si pasó el tiempo mínimo, se cierra la parada con el centro del grupo.

Los puntos entre dos paradas forman un **recorrido** (`trips`). Se simplifican con `ST_Simplify` para no guardar miles de vértices.

### 2. Parada → lugar existente o nuevo

```sql
select id
from places
where user_id = auth.uid()
  and ST_DWithin(location, ST_MakePoint(:lng, :lat)::geography, greatest(radius_m, 80))
order by location <-> ST_MakePoint(:lng, :lat)::geography
limit 1;
```

- Si hay resultado, se crea una visita a ese lugar.
- Si no, se crea un lugar nuevo y se le pone nombre con reverse geocoding de Mapbox. El usuario lo puede renombrar o fusionar con otro.

### 3. Misma ruta dos veces

Cada recorrido se guarda como un viaje nuevo, porque es otro día y forma parte del historial. Para el mapa de "lugares recorridos" se dibuja la unión de todos los recorridos (`ST_Union`) o un heatmap, así no se ve duplicado.

Los recorridos casi iguales (`ST_HausdorffDistance` menor a ~150 m, con los mismos lugares de origen y destino) comparten `route_group_id`. Así se detectan rutas frecuentes, como casa → trabajo.

### 4. Datos importados que se solapan con los automáticos

Al importar, si ya existe una visita al mismo lugar con horario superpuesto, se fusionan en una: se queda con el rango más amplio y la fuente de mayor prioridad (`manual` > `auto` > `import`).

## Parámetros a ajustar con datos reales

| Parámetro | Valor inicial |
|---|---|
| Tiempo mínimo de parada | 5 min |
| Radio de parada | 100 m |
| Distancia para considerarlo el mismo lugar | 80 m (o `radius_m` del lugar) |
| Umbral de ruta parecida | 150 m (Hausdorff) |
