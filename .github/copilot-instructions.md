# Reglas del proyecto: Observatorio de actividad sísmica

## Autoridad y alcance

- Antes de actuar, respeta esta prioridad: requisitos obligatorios del enunciado y su rúbrica; `docs/source-of-truth/CONTRATO-API.md` para la interfaz; principios de `.specify/memory/constitution.md`; decisiones explícitas del alumno; y, subordinados a todo lo anterior, los artefactos SDD de la tarea. La guía y las FAQ aclaran, pero no añaden requisitos.
- No inventes requisitos, amplíes alcance ni introduzcas tecnologías, servicios o infraestructura adicionales sin justificación y aprobación explícita. Distingue obligación, decisión del alumno y propuesta opcional.
- Consulta las reglas SDD y arquitectura en `.github/instructions/arquitectura-sdd.instructions.md` antes de implementar. Consulta también las reglas específicas de cada área aplicable.
- El código y las pruebas observadas describen el estado real; no prevalecen sobre los requisitos. Ante contradicciones, ambigüedad material o decisiones pendientes, detén la parte afectada, explica fuentes e impacto y solicita resolución. No conviertas errores en respuestas que aparenten éxito.
- El código generado debe ser claro, mantenible y estar al nivel necesario para que el alumno pueda explicar su propósito, flujo y decisiones. Prefiere los patrones y dependencias ya presentes.
- Evita la sobreingeniería: entrega la solución completa más pequeña que cumpla requisitos y criterios aprobados. No añadas funcionalidades, abstracciones, dependencias, código, specs, tareas ni artefactos SDD que la petición no necesite. La documentación puede ampliarse como excepción solo si mejora de forma útil la comprensión o defensa del alumno, sin inventar requisitos.

## Calidad y cierre

- Conserva el significado de los datos y las interfaces existentes. Comunica dependencias entre componentes antes de asumir un contrato o comportamiento nuevo.
- Considera una tarea terminada solo cuando cumple el alcance aprobado y sus criterios de aceptación, se han ejecutado o declarado explícitamente no ejecutadas las comprobaciones pertinentes, y se ha actualizado la documentación directamente afectada. Informa fallos, límites y trabajo pendiente; nunca afirmes pruebas o despliegues no observados.
- No modifiques código, SDD, fuentes del profesor ni otros archivos fuera del alcance solicitado. Mantén secretos fuera del repositorio y de la configuración pública.
