# Arquitectura

```
Celular (Expo)                          Supabase
┌──────────────────────────┐            ┌─────────────────────────────┐
│ Pantallas (mapa, diario) │──consultas─▶│ Postgres + PostGIS (RLS)    │
│ Tarea de ubicación       │──puntos───▶│ Edge Functions              │
│   en segundo plano       │            │   - procesar puntos GPS     │
│ Subida de fotos          │──archivos─▶│   - importar Timeline       │
└────────────┬─────────────┘            │ Storage (fotos)             │
             │                          │ Auth (Google)               │
             ▼                          └─────────────────────────────┘
        Mapbox (mapa, búsqueda, reverse geocoding)
```

## 1. App móvil con Expo

El tracking automático necesita leer el GPS con la app cerrada, y un navegador no puede. Por eso la app es nativa.

- **Expo + TypeScript.** Un solo código para Android e iOS.
- **`expo-location` + `expo-task-manager`** registran la ubicación en segundo plano.
- **Batería:** no se guarda un punto cada pocos segundos. Se usan actualizaciones por distancia (por ejemplo, cada 50 m) y un intervalo largo cuando no te movés. Los puntos se mandan al backend en lotes.
- **Instalación:** con EAS Build. En Android se genera un APK que se instala directo, sin pasar por la Play Store. En iOS hace falta una cuenta de Apple Developer (USD 99 por año) para instalarla de forma estable.
- Más adelante se puede sumar una web (Next.js) de solo lectura para ver el mapa en pantalla grande.

## 2. Backend: Supabase

Se eligió Supabase y no Firebase por las consultas geográficas:

- Firestore no tiene consultas geográficas reales. Se simulan con geohashes, que sirven para "puntos cerca de X", pero no para comparar recorridos.
- Supabase es Postgres con **PostGIS**. Con una sola consulta se puede preguntar "¿ya tengo un lugar a menos de 80 m de este punto?" o "¿este recorrido es casi igual a otro que ya tengo?". Eso es lo que permite evitar duplicados (ver [modelo de datos](modelo-datos.md)).
- Incluye **Auth con Google**, **Storage** para las fotos y **Row Level Security** (RLS): la base misma garantiza que cada usuario lea y escriba solo sus filas.
- El plan gratis (500 MB de base, 1 GB de archivos) alcanza para uso personal. Las fotos se comprimen antes de subirlas.

## 3. Mapas: Mapbox

El mapa que se ve no depende del backend: se dibuja en el celular y la base guarda solo coordenadas.

- **Render:** `@rnmapbox/maps` con el estilo "Streets", parecido a Google Maps.
- **Búsqueda y nombres de lugares:** Search API de Mapbox, con búsqueda por texto, lugares cercanos y reverse geocoding (coordenada → nombre).
- **Costumbre de Google Maps:** cada lugar tiene un botón "Abrir en Google Maps" que usa el deep link `https://www.google.com/maps/search/?api=1&query=<lat>,<lng>`. Los lugares importados de Timeline guardan su `google_place_id` cuando viene en el archivo.
- **Cambio de proveedor:** el mapa queda en un único módulo (`src/map/`). Pasar a Google Maps (`react-native-maps`) implicaría tocar solo ese módulo.

## 4. Seguridad y privacidad

La ubicación es un dato sensible:

- Todas las tablas tienen RLS por `user_id = auth.uid()`.
- Los puntos GPS crudos se borran después de procesarlos; quedan solo las visitas y los recorridos simplificados.
- Desde Ajustes se puede pausar el tracking, exportar todos los datos y borrar la cuenta con todo su contenido.
