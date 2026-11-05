---
name: revision-codigo-observatorio
description: 'Review code changes in the seismic observatory for correctness, regressions, data integrity, API/UI contract issues, and project-specific risks. Use for read-only review of a diff or implementation before merge or delivery.'
---

# Revisión de código del Observatorio

## Propósito

Encontrar defectos concretos en cambios de código y explicar su impacto, sin implementar correcciones ni confundir una revisión con una prueba ejecutada.

## Cuándo utilizarla

- Al solicitar revisión de un diff, una implementación o un cambio antes de integrarlo.
- Al buscar regresiones en ingesta, datos, API, mapa, Compose o manejo de errores.
- Al revisar coherencia técnica después de que QA haya ejecutado sus pruebas.

## Procedimiento

1. Identifica alcance del cambio y archivos afectados. Inspecciona suficiente contexto y el diff; no amplíes la revisión a áreas no relacionadas.
2. Consulta las fuentes pertinentes y contrato afectado. Usa el enunciado como fuente de obligaciones; guías/FAQ aclaran pero no añaden requisitos.
3. Sigue el comportamiento relevante de entrada a salida; comprueba tipos/unidades, límites, errores, persistencia, orden de revisiones, filtros y compatibilidad entre componentes.
4. Busca específicamente fallos acoplados al cambio: duplicación al reintentar, pérdida de datos al fallar, commit antes de guardar, nulls/NaN, geometría/coordenadas, datos truncados, secreto público, respuesta falsa de éxito.
5. Ejecuta tests solo si la solicitud y el permiso lo permiten; en cualquier caso, distingue conclusiones de lectura de resultados realmente observados.
6. Devuelve únicamente hallazgos accionables con severidad, archivo/línea, condición desencadenante, consecuencia y evidencia. Ordena por impacto/confianza.
7. Si no hay hallazgos, indica alcance y límites de la revisión; no inventes riesgos hipotéticos como defectos.

## Criterios de calidad

- Cada hallazgo señala una ruta reproducible al defecto y una consecuencia concreta.
- Alta precisión: no se reportan preferencias de estilo ni mejoras opcionales como bugs.
- Los requisitos fuente y decisiones opcionales se clasifican correctamente.
- La revisión es de solo lectura y no afirma que los tests pasaron si no se ejecutaron.
- Se diferencia de QA (ejecución/evidencia de tests) y del Verificador (cobertura solicitud/plan/fuentes).

## Agentes que la utilizan

Backend y Frontend antes de integrar cambios; Verificador cuando además de cobertura fuente se pide revisión de defectos; Coordinador para solicitar revisión independiente. QA conserva la ejecución de pruebas.

## Referencias

- [Constitución](../../../.specify/memory/constitution.md)
- [Enunciado](../../../docs/source-of-truth/ENUNCIADO.md)
- [Contrato de API](../../../docs/source-of-truth/CONTRATO-API.md)
- [Validación de la base](../../../docs/source-of-truth/VALIDACION.md)
