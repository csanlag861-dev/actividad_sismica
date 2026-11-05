---
description: "Use before implementing project changes, defining requirements, or modifying architecture and interfaces. Covers SDD traceability, source precedence, and cross-component boundaries."
---
# Arquitectura y SDD

- Antes de implementar una tarea, localiza y lee su `spec.md` aplicable. Revisa también `plan.md`, `tasks.md` y los criterios de aceptación existentes que condicionen el cambio.
- Consulta SDD antes de implementar. Crea o amplía una spec solo para una funcionalidad nueva o un cambio material de requisitos/comportamiento; para una corrección o mantenimiento acotado, usa el criterio SDD existente y no generes specs, planes ni tareas por formalidad. Si un cambio material no tiene spec o criterios suficientes, solicita el flujo SDD existente y detén esa implementación. Aplica `.github/skills/sdd-observatorio/SKILL.md` y las skills Speckit pertinentes; no dupliques sus procedimientos.
- Contrasta los artefactos SDD con el enunciado y la rúbrica, la constitución y las secciones pertinentes de `docs/source-of-truth/`. Los artefactos SDD no pueden rebajar ni contradecir requisitos obligatorios.
- Mantén los límites del flujo: productor → Kafka → consumidor → PostgreSQL → FastAPI → Angular/Leaflet. No cambies responsabilidades ni contratos compartidos unilateralmente.
- Vincula los cambios con criterios de aceptación comprobables. Si una contradicción o decisión material afecta al alcance, comportamiento, arquitectura o validación, detén esa parte y pide resolución; no la sustituyas por un supuesto.
- Trata código existente como evidencia del estado actual, no como aprobación de una desviación de los requisitos.
- No añadas alcance, tareas, artefactos ni capas de diseño que no sean necesarios para cumplir los criterios acordados. La documentación explicativa puede ampliarse si aporta valor directo al aprendizaje.
