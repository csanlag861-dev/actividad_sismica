---
description: "Use for the Angular and Leaflet observatory panel, including API integration, GeoJSON, map layers, filters, tables, charts, and UI states."
applyTo:
  - "frontend/src/**/*.ts"
  - "frontend/src/**/*.html"
  - "frontend/src/**/*.css"
---
# Frontend Angular y Leaflet

- Consume los datos procesados por la API; no sustituyas el pipeline del proyecto por consultas directas a USGS.
- Mantén coherentes mapa, tabla, filtros y estadísticas, respetando el contrato de API y los estados de carga, error, desactualización y truncamiento que correspondan.
- Distingue datos de demostración, datos reales y modos `live`/`replay`. No presentes datos ficticios como análisis real ni estadísticas de una muestra truncada como globales.
- Conserva los recursos entregados, las atribuciones cartográficas y las prácticas accesibles existentes. El código de interfaz debe ser comprensible para el alumno.
- Aplica `.github/skills/angular-leaflet-observatorio/SKILL.md` para procedimientos específicos del mapa e integración.
