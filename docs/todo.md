1. Crear un documento contexto para distintas IA's.
2. Crear un TODO donde estén las tareas que me quedan por hacer en las sesiones.
3. Que el profesor me cree los docs en una carpeta aparte, y las preguntas en otra carpeta aparte.


---
Prepara SOLO specs/002-modelo-persistencia/plan.md para esta feature. El usuario compartió resultado correcto del comando .specify/scripts/powershell/resolve-template.ps1 spec-template -Json, con TEMPLATE_NAME=spec-template y contenido estándar del template (salida disponible en historial). La decisión y spec aplicable ya están en specs/002-modelo-persistencia/spec.md, aclaraciones completas; constitución en .specify/memory/constitution.md. Petición original: modelo conceptual/lógico/físico y diccionario para originales, estado actual, versiones, procedencia, rechazos motivo+referencia; SQLAlchemy + Alembic con migración revisada usada para schema (no create_all); FastAPI ejecuta alembic upgrade head antes de Uvicorn y falla visible; después de postgres health; no servicio Compose nuevo; mantener nulos/unidades/claves/restricciones e idempotencia; probar DB vacía, sin cambios pendientes post-migración y persistencia tras reinicio; SQL generado desde migraciones enlazado desde memoria o notebook; no implementar rutas finales ni cambiar contrato API. Decisiones acordadas en spec: respuesta completa GeoJSON conserva 90 días, originales de versiones y rechazos duraderos; live/replay comparten tablas, el modo es parte de identidad y consultas deben distinguir para evitar mezcla. Sigue estrictamente tu rol: crear o actualizar ÚNICAMENTE plan.md; no modificar spec, no generar tasks.md ni research/data-model/quickstart/contracts u otros archivos, no implementar código. Revisa fuentes y guía apropiadas, plan acotado y criterios de aceptación trazables. Conserva la metodología de aprobación: esto produce propuesta pendiente de aprobación del usuario.

-
Objetivo y alcance
Continué la fase de planificación para el modelo de persistencia. El Planificador SDD creó plan.md; lo revisé contra la spec, la constitución, el enunciado y el Compose existente.

El plan respeta el alcance: migraciones con SQLAlchemy y Alembic antes de Uvicorn, espera a PostgreSQL saludable, conserva el contrato y las rutas de la API, y no añade servicios Compose.

Propuestas que requieren tu aprobación
El plan propone dos decisiones de arquitectura que no estaban especificadas en la solicitud:

Que el productor guarde en PostgreSQL cada respuesta GeoJSON completa y sus metadatos. Esto añade al productor una dependencia de PostgreSQL, aunque no agrega un servicio.
Que la limpieza de respuestas vencidas se ejecute en el ciclo de ingesta del productor. Los originales asociados a versiones y rechazos quedarían fuera de esa limpieza.
También propone que las claves incluyan el modo: (mode, event_id) para el estado actual y (mode, event_id, updated) para las versiones. Esto implementa la decisión ya acordada de compartir tablas y distinguir live de replay.

Fase SDD y siguiente paso
Estamos en el punto de aprobación del plan. Todavía no he pedido el desglose de tareas ni iniciado la implementación. Para continuar, confirma si apruebas el plan tal como está, incluidos los dos primeros puntos; si no, dime qué deseas cambiar.

Resultado y verificaciones
El único artefacto nuevo de esta fase es plan.md. No se ejecutaron pruebas ni se implementó código; las validaciones descritas en el plan son futuras.