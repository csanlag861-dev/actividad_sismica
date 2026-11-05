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
4. Identifica supuestos, dependencias, límites y decisiones materiales no resueltas. Si una duda material no puede resolverse con las fuentes del proyecto, formula preguntas abiertas al usuario y espera sus respuestas; no conviertas la duda en un requisito, supuesto ni decisión propia.
5. No generes ni avances `plan.md` o `tasks.md`, ni delegues tareas de implementación, mientras quede alguna decisión material pendiente que afecte al alcance, comportamiento esperado, criterios de aceptación o estrategia de validación. Expón qué está bloqueado y las preguntas que debe responder el usuario. Solo aplica valores predeterminados explícitos del proyecto para detalles de bajo impacto y comunícalos.
6. Si se necesitan artefactos, utiliza las skills Speckit existentes (`speckit-clarify`, `speckit-specify`, `speckit-plan`, `speckit-tasks`, `speckit-analyze`, `speckit-implement` o `speckit-converge`) según la etapa. Ejecuta y completa `speckit-clarify` antes del plan cuando haya ambigüedades; no recrees sus plantillas ni generes artefactos innecesarios.
7. Vincula cada requisito con uno o más criterios observables y cada tarea con el criterio que satisface.
8. Contrasta los artefactos con el resultado real y registra lo no verificable, sin afirmar que el plan prueba la implementación.

## Criterios de calidad

- Toda obligación tiene referencia a una fuente y evidencia de aceptación comprobable.
- Opciones y ejemplos no se convierten en requisitos del profesor.
- El alcance no introduce tecnologías o infraestructura adicionales sin justificación.
- Se utiliza el artefacto SDD existente cuando basta; se crea o amplía una spec solo si la petición introduce una funcionalidad o cambio material. No se generan specs, planes o tareas adicionales por formalidad.
- La solución y el conjunto de artefactos son los mínimos suficientes para cumplir los criterios aprobados y poder explicarlos; la documentación puede ampliarse si aporta valor didáctico concreto.
- Los criterios preservan el significado de datos nulos, demo/reales y live/replay.
- Las decisiones abiertas, dependencias y trabajo no verificado quedan explícitos.
- No se crea el plan ni la lista de tareas mientras haya decisiones materiales pendientes del usuario.

## Agentes que la utilizan

Analista de Requisitos para criterios y trazabilidad; Coordinador para encuadrar y ordenar trabajo; Verificador para contrastar solicitud, plan, resultado y fuentes.

## Referencias

- [Constitución](../../../.specify/memory/constitution.md)
- [Enunciado](../../../docs/source-of-truth/ENUNCIADO.md)
- [Contrato de API](../../../docs/source-of-truth/CONTRATO-API.md)
- [Skill Speckit Specify](../speckit-specify/SKILL.md)
- [Skill Speckit Analyze](../speckit-analyze/SKILL.md)
