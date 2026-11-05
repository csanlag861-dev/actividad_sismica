---
name: analisis-calidad-sismica
description: 'Analyze and document quality of the seismic source and prepared data. Use for completeness, uniqueness, validity, consistency, timeliness, accuracy, quarantine decisions, before/after evidence, denominators, or student report/notebook analysis.'
---

# Análisis de calidad de datos sísmicos

## Propósito

Producir evidencias de calidad rigurosas y explicables, separando validación operativa del consumidor del análisis que el alumno presenta en memoria o notebook.

## Cuándo utilizarla

- Al diseñar reglas de validación/transformación del consumidor.
- Al calcular métricas de calidad o redactar el apartado de memoria/notebook.
- Al revisar decisiones sobre nulos, anomalías, duplicados, cuarentena o procedencia.

## Procedimiento

1. Identifica fuente, periodo, filtros y grano medido: mensajes recibidos, versiones distintas o eventos actuales.
2. Selecciona dimensiones pertinentes: completitud, unicidad, validez, consistencia, actualidad y exactitud. Explica las dimensiones que no puedan medirse.
3. Define denominador, consulta/código y regla antes de calcular. Informa numerador, total y porcentaje; las incidencias pueden solaparse, no las sumes como registros distintos.
4. Compara antes/después de una transformación cuando aplique, indicando cuántos registros cambiaron, se conservaron o se pusieron en cuarentena.
5. Conserva los nulos como desconocidos; explica cómo afectan a filtros, medias y denominadores. No conviertas nulo en cero.
6. Investiga anomalías antes de rechazar: magnitud o profundidad negativas no son automáticamente inválidas. Valida tipos, fechas UTC, coordenadas finitas/en rango y geometría según contrato.
7. Documenta consistencia de versiones, claves geográficas, unidades, demora entre evento/actualización/ingesta y límites del catálogo/fuente.
8. Distingue lo que las reglas detectan de la exactitud física: validar rangos no demuestra que una magnitud sea correcta. No inventes errores en datos reales; separa fixtures sintéticos.
9. Presenta código/SQL, resultados observados, acciones justificadas y limitaciones dentro de memoria o notebook. La generación automática de informes, tabla de métricas o panel de calidad es opcional, no requisito.

## Criterios de calidad

- Cada cifra tiene fuente, periodo, filtros, denominador y método reproducible.
- Nulos, exclusiones y duplicados conservan su significado y se contabilizan claramente.
- Rechazos operativos preservan duraderamente original/referencia y motivo.
- Acciones no eliminan datos por una regla no justificada.
- Se separan datos reales, datos sintéticos, resultados medidos y supuestos.

## Agentes que la utilizan

Backend la utiliza para reglas del consumidor y cuarentena; QA Tester para probar validaciones; Documentador para explicar evidencias; Profesor/Mentor para practicar interpretación de resultados.

## Referencias

- [Enunciado](../../../docs/source-of-truth/ENUNCIADO.md)
- [FAQ del alumnado](../../../docs/source-of-truth/FAQ-ALUMNADO.md)
- [Guía de entregables](../../../docs/source-of-truth/Guia-entregables-observatorio-sismico.md)
- [Constitución](../../../.specify/memory/constitution.md)
