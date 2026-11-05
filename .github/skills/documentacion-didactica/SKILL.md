---
name: documentacion-didactica
description: 'Write or update clear, didactic documentation for the seismic observatory. Use when explaining architecture, Python/Kafka/PostgreSQL/FastAPI/Angular/Leaflet decisions, synchronizing docs with code/SDD, or documenting test evidence.'
---

# Documentación didáctica

## Propósito

Mantener explicaciones técnicamente fieles y útiles para alumnado, sincronizadas con la implementación, las decisiones confirmadas y las evidencias disponibles.

## Cuándo utilizarla

- Al documentar cambios, arquitectura, instalación, modelo de datos o funcionamiento.
- Al explicar un concepto técnico a dos niveles: idea sencilla y detalle aplicable al proyecto.
- Al incorporar resultados de pruebas o análisis a README, memoria o notebook.

## Procedimiento

1. Inspecciona los documentos vigentes, artefactos SDD y código real antes de redactar; identifica el público y la pregunta que la documentación debe responder.
2. Consulta fuente pertinente y clasifica lo escrito: requisito, decisión confirmada, opción, hecho observado, supuesto o pendiente.
3. Actualiza el documento existente más adecuado. Evita duplicar contenidos, crear documentos nuevos por apartado o alterar las fuentes originales del profesor.
4. Explica flujo y conceptos desde contexto del proyecto: componentes y responsabilidades, datos de entrada/salida, unidades, errores, alternativas y consecuencias.
5. Usa ejemplos ficticios claramente marcados. No reutilices datos demo como resultados reales; nunca presentes el observatorio como predictor.
6. Registra pruebas con procedimiento, resultado esperado, observado, evidencia y estado. Marca NO EJECUTADO lo que no se haya probado.
7. Incluye referencias relativas estables a código, contratos y documentos; comprueba enlaces y que la explicación siga el estado actual.

## Criterios de calidad

- Puede ampliarse la explicación más allá del mínimo solicitado cuando aporte valor concreto al aprendizaje o defensa; evita crear documentos redundantes o contenido sin relación con el proyecto.
- Un lector puede seguir el flujo y repetir los pasos sin adivinar configuración.
- Conceptos se explican con vocabulario accesible sin sacrificar exactitud.
- Decisiones y alternativas se presentan solo si están confirmadas/documentadas.
- Resultados no exceden la evidencia observada y fuentes sintéticas se identifican.
- La documentación permanece coherente con el código y los artefactos SDD.

## Agentes que la utilizan

Documentador es responsable; Backend, Frontend y Deploy la usan para actualizar documentación directamente afectada; Coordinador solicita sincronización; Profesor/Mentor puede apoyarse en explicaciones, pero no sustituye la edición documental.

## Referencias

- [Guía de entregables](../../../docs/source-of-truth/Guia-entregables-observatorio-sismico.md)
- [Enunciado](../../../docs/source-of-truth/ENUNCIADO.md)
- [README del material base](../../../docs/source-of-truth/README.md)
- [Constitución](../../../.specify/memory/constitution.md)
