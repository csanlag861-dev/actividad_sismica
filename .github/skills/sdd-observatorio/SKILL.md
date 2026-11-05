---
name: sdd-observatorio
description: 'Apply project-specific Spec-Driven Development to the seismic observatory. Use when clarifying scope, creating or reviewing specs, plans, tasks, acceptance criteria, or checking implementation traceability.'
---

# SDD del Observatorio sísmico

## Propósito

Aplicar el flujo SDD existente sin duplicar las skills Speckit y manteniendo cada requisito trazable a la fuente correcta.

## Cuándo utilizarla

- Al iniciar una funcionalidad, cambio de alcance o requisito nuevo.
- Al revisar `spec.md`, `plan.md`, `tasks.md` o criterios de aceptación.
- Al detectar diferencias entre una petición, los artefactos SDD, la implementación y las fuentes del proyecto.

## Procedimiento

1. Resume el resultado que pide el usuario y el alcance explícito.
2. Consulta `.specify/memory/constitution.md` y únicamente las secciones pertinentes de `docs/source-of-truth/`.
3. Clasifica cada punto como **requisito obligatorio**, **decisión del estudiante** o **propuesta opcional**. El enunciado y su rúbrica determinan obligaciones; `CONTRATO-API.md` rige la interfaz; guía y FAQ aclaran, no añaden requisitos.
4. Identifica supuestos, dependencias, límites y decisiones materiales no resueltas. Solicita aprobación antes de convertir una decisión material en alcance.
5. Si se necesitan artefactos, utiliza las skills Speckit existentes (`speckit-clarify`, `speckit-specify`, `speckit-plan`, `speckit-tasks`, `speckit-analyze`, `speckit-implement` o `speckit-converge`) según la etapa. No recrees sus plantillas ni generes artefactos innecesarios.
6. Vincula cada requisito con uno o más criterios observables y cada tarea con el criterio que satisface.
7. Contrasta los artefactos con el resultado real y registra lo no verificable, sin afirmar que el plan prueba la implementación.

## Criterios de calidad

- Toda obligación tiene referencia a una fuente y evidencia de aceptación comprobable.
- Opciones y ejemplos no se convierten en requisitos del profesor.
- El alcance no introduce tecnologías o infraestructura adicionales sin justificación.
- Los criterios preservan el significado de datos nulos, demo/reales y live/replay.
- Las decisiones abiertas, dependencias y trabajo no verificado quedan explícitos.

## Agentes que la utilizan

Analista de Requisitos para criterios y trazabilidad; Coordinador para encuadrar y ordenar trabajo; Verificador para contrastar solicitud, plan, resultado y fuentes.

## Referencias

- [Constitución](../../../.specify/memory/constitution.md)
- [Enunciado](../../../docs/source-of-truth/ENUNCIADO.md)
- [Contrato de API](../../../docs/source-of-truth/CONTRATO-API.md)
- [Skill Speckit Specify](../speckit-specify/SKILL.md)
- [Skill Speckit Analyze](../speckit-analyze/SKILL.md)
