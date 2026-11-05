---
name: defensa-tecnica
description: 'Prepare and rehearse the technical defense of the seismic observatory. Use for oral explanation, end-to-end walkthrough, architectural decision justification, evidence review, limitations, or progressive mock questions.'
---

# Preparación de la defensa técnica

## Propósito

Ayudar al alumno a demostrar comprensión y autonomía explicando el sistema real, sus evidencias y decisiones, sin sustituir el trabajo ni inventar resultados.

## Cuándo utilizarla

- Al preparar una exposición, demostración o preguntas del profesor.
- Al practicar el recorrido de un evento desde USGS hasta el panel.
- Al justificar una decisión, explicar un fallo, una limitación o una prueba.

## Procedimiento

1. Revisa la implementación y evidencia disponible. Separa hechos observados, decisiones confirmadas, resultados de pruebas y pendientes.
2. Pide al alumno seleccionar o identifica una pieza concreta a defender. Formula primero una o pocas preguntas y espera su razonamiento antes de mostrar una respuesta modelo.
3. Practica el flujo: productor/USGS → Kafka → consumidor/validación → PostgreSQL → FastAPI → Angular/Leaflet; pregunta qué aporta cada frontera y qué estado conserva.
4. Profundiza en una decisión técnica propia, como identidad/versionado, confirmar offset tras guardar, catálogo espacial, filtros comparables, selección de índices o aislamiento replay. Pide alternativas, coste, beneficio y límites respaldados por el proyecto.
5. Ensaya una demostración reproducible: estado de Compose, última ingesta, endpoint/API, panel, filtros y replay etiquetado. Distingue la fecha del evento de la hora de ingesta/reproducción.
6. Pregunta por evidencias de calidad, pruebas, mediciones y conclusiones: fuente, periodo, filtros, denominador, resultado esperado/observado y limitaciones.
7. Corrige ideas erróneas con explicación breve y propone una pregunta de seguimiento o pequeño ejercicio. Evalúa razonamiento, no memorización textual.
8. Cierra con fortalezas observadas, temas a repasar y sugerencias de práctica dentro de la conversación actual.

## Criterios de calidad

- El alumno practica responder antes de recibir la explicación completa, salvo que la pida.
- Preguntas se basan en código, fuentes y resultados existentes, no en capacidades imaginadas.
- No se confunden datos demo, sintéticos, replay y reales; ni análisis descriptivo con predicción.
- Se indican límites de pruebas, fuentes, despliegue local y servicios externos.
- El progreso se limita al contexto observable; no se afirma memoria entre sesiones.

## Agentes que la utilizan

Profesor/Mentor la usa para práctica interactiva y, tras la implementación verificada y a petición del Coordinador, escribe únicamente `defensa-tecnica.md`; Documentador prepara los demás documentos y preguntas; Coordinador puede incluirla como etapa de preparación, no como sustituto de QA.

## Referencias

- [Enunciado y evaluación](../../../docs/source-of-truth/ENUNCIADO.md)
- [Guía de entregables y defensa](../../../docs/source-of-truth/Guia-entregables-observatorio-sismico.md)
- [FAQ del alumnado](../../../docs/source-of-truth/FAQ-ALUMNADO.md)
- [Validación de la base](../../../docs/source-of-truth/VALIDACION.md)
