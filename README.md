# Reconocimiento mundial

App móvil para llevar el historial de todos los lugares que recorrés en tu vida. Registra visitas y recorridos automáticamente, permite importar tu historial de Google Maps Timeline y, en cada lugar, dejar opiniones y subir fotos. Se ingresa con la cuenta de Google y todo se guarda en la nube.

> **Estado:** en planificación. Todavía no hay código de la app.

## Stack

| Parte | Tecnología |
|---|---|
| App | Expo (React Native) + TypeScript |
| Ubicación en segundo plano | `expo-location` + `expo-task-manager` |
| Mapa y búsqueda de lugares | Mapbox (`@rnmapbox/maps`, Search API) |
| Backend | Supabase: Postgres + PostGIS, Auth (Google), Storage, Edge Functions |
| Build | EAS Build |

## Documentación

- [Arquitectura](docs/arquitectura.md): decisiones técnicas y por qué.
- [Modelo de datos](docs/modelo-datos.md): tablas, seguridad y reglas para evitar duplicados.
- [Importación](docs/importacion.md): cómo traer el historial de Google Timeline.
- [Roadmap](docs/roadmap.md): fases, pantallas, costos y cómo verificar cada etapa.
