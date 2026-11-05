---
name: pruebas-aceptacion-replay
description: 'Design or run acceptance and integration tests for the seismic observatory, including Kafka/PostgreSQL revisions, quarantine, API/UI behavior, tunnel recovery, and historical replay. Use when verifying expected versus observed behavior with evidence.'
---

# Pruebas de aceptación y replay

## Propósito

Probar criterios obligatorios mediante resultados observables y repetibles, sin afirmar que una implementación funciona solo por inspeccionar código.

## Cuándo utilizarla

- Al planear o ejecutar pruebas backend, frontend, integración, Compose o aceptación.
- Al verificar idempotencia, revisiones, datos inválidos, reinicios o errores de conexión.
- Al preparar demostración acelerada por Kafka con datos históricos.

## Procedimiento

1. Selecciona un criterio concreto del enunciado o contrato y define precondiciones, acción, resultado esperado y evidencia.
2. Inspecciona los runners, tests y comandos configurados. Ejecuta el alcance más pequeño que comprueba el requisito; no introduzcas tooling sin necesidad.
3. Usa datos reales solo para lo que representan. Para casos dañados o secuencias controladas crea fixtures sintéticos etiquetados y aislados.
4. Cubre, según el cambio: evento nuevo; misma versión repetida; revisión posterior; versión antigua tardía; nulo; coordenadas inválidas y cuarentena; evento sin país; reinicio; contrato/error API; filtros sincronizados; túnel desconectado/recuperado.
5. Para replay, usa selección histórica ordenada, conserva la fecha de ocurrencia, registra hora de reproducción y usa topic/tablas/esquema separados o aislamiento equivalente. Incluye duplicado/revisión controlados y verifica etiqueta visible «REPRODUCCIÓN».
6. Si hay pruebas end-to-end, demuestra recorrido USGS/replay → Kafka → consumidor → PostgreSQL → API → Angular con evidencia en cada frontera relevante.
7. Registra por prueba **esperado**, **observado**, **estado** (PASÓ/FALLÓ/NO EJECUTADO), comando/procedimiento, entorno y evidencia. Si no se ejecutó, nunca marques PASÓ.
8. Separa fallos de producto, limitaciones ambientales y huecos de cobertura. Evita alterar datos persistentes de usuario o servicios fuera del entorno de prueba.

## Criterios de calidad

- La afirmación de éxito se apoya en una observación reproducible.
- Repetición y revisiones prueban recuento actual y versión almacenada, no solo una respuesta HTTP.
- Datos sintéticos quedan identificados y no se usan como análisis real.
- Los errores y pruebas no ejecutadas se comunican con causa y no se disfrazan.
- El replay no contamina live ni cambia timestamps originales.

## Agentes que la utilizan

QA Tester es responsable de ejecutar y reportar; Backend, Frontend y Deploy la aplican al validar cambios propios; Coordinador la usa para solicitar pruebas verificables.

## Referencias

- [Enunciado y aceptación](../../../docs/source-of-truth/ENUNCIADO.md)
- [Guía de entregables](../../../docs/source-of-truth/Guia-entregables-observatorio-sismico.md)
- [FAQ del alumnado](../../../docs/source-of-truth/FAQ-ALUMNADO.md)
- [Validación de la base](../../../docs/source-of-truth/VALIDACION.md)
