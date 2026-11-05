---
name: api-fastapi-observatorio
description: 'Implement or review the seismic observatory FastAPI service against its API contract. Use for GeoJSON endpoints, filters, statistics, health metadata, CORS, error behavior, or database query handling.'
---

# API FastAPI del Observatorio

## Propósito

Mantener la interfaz de consulta coherente con el contrato versionado y servir datos ya procesados, no sustituir el pipeline.

## Cuándo utilizarla

- Al implementar o modificar rutas, esquemas de respuesta, filtros o consultas PostgreSQL.
- Al depurar errores de contrato, nulos, timestamps, CORS o respuestas truncadas.
- Al coordinar cambios de interfaz entre backend y Angular.

## Procedimiento

1. Lee `CONTRATO-API.md` y los endpoints/campos implementados en el repositorio. Confirma las dependencias/versiones antes de elegir patrones FastAPI.
2. Inspecciona el modelo real de datos, grano actual/versiones y repositorios de consultas. La API consulta datos preparados; no conecta directamente con USGS para alimentar el panel.
3. Implementa `GET /api/eventos` como `FeatureCollection`, también vacía cuando no hay eventos. Mantén orden GeoJSON `[longitud, latitud, profundidad_km]`, tiempos Unix UTC en milisegundos y valores desconocidos como `null`, nunca NaN/Infinity.
4. Alinea filtros `start` inclusivo, `end` exclusivo, `min_mag` y `country_id` con el contrato. Un `min_mag` excluye magnitud nula, que debe seguir contabilizándose por separado cuando se informa.
5. Implementa/ajusta `/api/salud` y `/api/estadisticas` según contrato; propaga los filtros coherentemente. Incluye periodo, filtros, unidad y registros sin valor en estadísticas.
6. Si se limita tamaño, documenta máximo y paginación, devuelve metadatos de truncamiento y total según contrato. No declares completas estadísticas calculadas sobre una muestra truncada.
7. Si la base falla, devuelve un error explícito; no devuelvas cero eventos como éxito. Configura CORS para los orígenes permitidos, `OPTIONS` y la cabecera ngrok aplicable, sin confundir CORS con autenticación.
8. Añade o ajusta validación y pruebas de contrato existentes. Expón los cambios necesarios al frontend antes de alterar semántica.

## Criterios de calidad

- Implementa el contrato solicitado con la solución más pequeña suficiente; no inventes rutas, parámetros ni capas de API adicionales.
- Respuestas válidas se ajustan al contrato en forma, unidades, filtros y semántica.
- Vacío real, dato nulo, truncamiento y fallo de infraestructura son estados distintos.
- Los errores no se silencian ni se transforman en éxito falso.
- Las estadísticas usan el mismo estado actual y filtros que el mapa/tabla.
- No se exponen credenciales ni endpoints de administración.

## Agentes que la utilizan

Backend es responsable de implementar; Frontend la consulta al integrar; QA Tester usa los casos de contrato; Coordinador la usa si hay cambio de interfaz entre agentes.

## Referencias

- [Contrato de API](../../../docs/source-of-truth/CONTRATO-API.md)
- [Enunciado](../../../docs/source-of-truth/ENUNCIADO.md)
- [Guía del material base](../../../docs/source-of-truth/README.md)
