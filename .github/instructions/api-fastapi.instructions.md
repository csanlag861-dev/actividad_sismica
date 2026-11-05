---
description: "Use when implementing or changing FastAPI routes, GeoJSON responses, API filters, metadata, statistics, errors, or CORS."
---
# API con FastAPI

- Trata `docs/source-of-truth/CONTRATO-API.md` como autoridad para rutas, filtros, formato GeoJSON, metadatos, unidades, nulos, truncamiento y errores.
- Si una solicitud o diseño entra en conflicto con el contrato, no introduzcas una interpretación unilateral: expón el conflicto y solicita resolución antes de cambiar el comportamiento compartido.
- No conviertas fallos de base de datos o procesamiento en respuestas válidas con cero eventos ni ocultes errores relevantes.
- Mantén el código y los modelos de respuesta comprensibles para el alumno; prueba los criterios de aceptación aplicables y comunica qué se ejecutó.
- Aplica `.github/skills/api-fastapi-observatorio/SKILL.md` para procedimientos de implementación del contrato.
