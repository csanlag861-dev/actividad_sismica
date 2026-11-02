# Especificaciones del Observatorio de Actividad Sísmica

## Propósito y límites

`docs/spec/` reúne una lectura trazable de los requisitos, el alcance y las condiciones de aceptación del proyecto. Sirve para preparar el trabajo y revisar su cumplimiento; no es una implementación, no sustituye la rúbrica ni decide detalles que las fuentes dejan abiertos.

Estas especificaciones no sustituyen, modifican ni prevalecen sobre los documentos de [`docs/source-of-truth/`](../source-of-truth/). Si una interpretación entra en conflicto con ellos, se conserva la fuente de mayor autoridad y se registra la duda para revisión.

## Fuentes y autoridad

- [Enunciado](../source-of-truth/ENUNCIADO.md): contiene los requisitos del proyecto y la rúbrica.
- [Contrato mínimo de API](../source-of-truth/CONTRATO-API.md): define el detalle mínimo del contrato de API.
- [README del material base](../source-of-truth/README.md), [FAQ del alumnado](../source-of-truth/FAQ-ALUMNADO.md) y [guía de entregables](../source-of-truth/Guia-entregables-observatorio-sismico.md): aclaran y orientan; no amplían por sí mismos los requisitos.
- [Fuentes y licencias](../source-of-truth/FUENTES-Y-LICENCIAS.md): documenta procedencia y licencias.
- [Validación del material base](../source-of-truth/VALIDACION.md): evidencia declarada sobre las comprobaciones del material entregado; no demuestra que la API, el pipeline o el backend estén implementados o probados.

La jerarquía funcional se limita a lo declarado por esos materiales: el enunciado rige requisitos y evaluación; el contrato concreta la API mínima; README, FAQ y guía explican el material y la entrega sin añadir obligaciones. Las fuentes y la validación aportan procedencia y evidencia del material base, respectivamente.

## Cómo leer las afirmaciones

Cada afirmación de requisito o decisión debe identificarse como una de estas clases:

- **Requisito del profesor**: exigencia explícita del enunciado o del detalle mínimo obligatorio del contrato. Se cita fuente y sección.
- **Aclaración del estudiante**: alcance o preferencia confirmada por el estudiante; no se atribuye al profesor.
- **Recomendación del agente**: sugerencia no obligatoria, indicada expresamente como recomendación.
- **Decisión pendiente**: cuestión no resuelta por las fuentes o por las aclaraciones confirmadas. Se conserva como pendiente, sin completar por suposición.

Cuando las fuentes califican algo como propuesta, ejemplo, opción o ampliación voluntaria, no se eleva a requisito obligatorio.

## Identificadores y trazabilidad

Los identificadores son estables dentro de esta especificación y provisionales hasta la revisión del estudiante: `REQ-F-###` (funcional), `REQ-NF-###` (no funcional), `REQ-ARCH-###` (arquitectura), `REQ-TEC-###` (tecnología), `REQ-DATA-###` (datos), `REQ-QUAL-###` (calidad), `REQ-TEST-###` (pruebas), `REQ-API-###` (contrato API), `REQ-DEP-###` (despliegue) y `REQ-DOC-###` (documentación y entrega).

Cada requisito incluye fuente y sección, carácter, dependencias, método verificable y estado. Un estado de planificación no implica que la funcionalidad esté implementada ni que la prueba haya pasado.

## Índice y estado

1. [00 — Principios y alcance](00-principios-y-alcance.md): marco de interpretación, límites y estado de las decisiones abiertas. **Estado: actualizado con aclaraciones del estudiante; decisiones de diseño pendientes.**
2. [01 — Requisitos y aceptación](01-requisitos-aceptacion.md): requisitos trazables y métodos de aceptación. **Estado: actualizado con aclaraciones; implementación y pruebas pendientes.**
3. [02 — Arquitectura y flujos](02-arquitectura-flujos.md): componentes, recorridos de datos, fallos y límites de red/despliegue. **Estado: borrador pendiente de revisión; decisiones de diseño abiertas.**
4. [03 — Modelo de datos y calidad](03-modelo-datos-calidad.md): conceptos, granos, campos, calidad, procedencia y criterios de aceptación sin fijar esquema físico. **Estado: borrador pendiente de revisión; propuestas conceptuales no aprobadas.**
5. [04 — Ingesta y procesamiento](04-ingesta-procesamiento.md): extracción live e histórica, validación, versionado, enriquecimiento, persistencia, recuperación y replay. **Estado: borrador pendiente de revisión; decisiones técnicas abiertas.**
6. [05 — API y contratos](05-api-contratos.md): contrato normativo de `/api/eventos` y propuestas para estadísticas, salud, paginación y errores. **Estado: aprobado por el estudiante; no implica implementación ni pruebas.**
7. [06 — Panel y análisis](06-panel-analisis.md): vistas, filtros, estados de interfaz, comparación histórica y criterios verificables del panel Angular/Leaflet. **Estado: borrador pendiente de revisión.**
8. [07 — Despliegue y operación](07-despliegue-operacion.md): arquitectura de despliegue, red, seguridad, Netlify, operación de demostración, recuperación y aceptación. **Estado: borrador pendiente de revisión; opciones operativas abiertas.**
9. [08 — Pruebas, entrega y defensa](08-pruebas-entrega-defensa.md): matriz de aceptación, calidad, mediciones, entregables, defensa y estado de evidencias. **Estado: borrador pendiente de revisión; pruebas no ejecutadas.**

Este README indexa los documentos de especificación existentes en `docs/spec/`; no anticipa documentos adicionales como si ya existieran. Las specs no sustituyen ni modifican `docs/source-of-truth/`.

Las preguntas abiertas registran su estado, el documento donde deben resolverse y si afectan a la planificación o solo a una implementación/configuración concreta. Las rutas futuras citadas dentro de las specs son destinos propuestos, no archivos creados ni aprobados.