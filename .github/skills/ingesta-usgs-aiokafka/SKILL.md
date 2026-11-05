---
name: ingesta-usgs-aiokafka
description: 'Implement or troubleshoot the seismic data producer in Python with USGS feeds, aiokafka and Kafka. Use for polling, historical loading, version detection, retries, message keys, producer state, or ingestion provenance.'
---

# Ingesta USGS con Python y aiokafka

## Propósito

Guiar la extracción y publicación de eventos y revisiones conforme a las fuentes del proyecto, sin asumir que el backend ya existe ni imponer una estructura de código.

## Cuándo utilizarla

- Al implementar o modificar el productor USGS, la carga histórica o la publicación Kafka.
- Al depurar duplicados, revisiones atrasadas, desconexiones, reintentos o metadatos de ingesta.
- Al preparar datos de reproducción que deban atravesar Kafka.

## Procedimiento

1. Inspecciona el código, dependencias, configuración y pruebas existentes. Confirma la versión y los patrones de `aiokafka`; no inventes módulos ni interfaces.
2. Comprueba en el enunciado los endpoints, campos, filtros y frecuencias requeridos. El feed `all_day` es una ventana móvil aproximada de 24 horas, no una lista de eventos nuevos.
3. Conserva el GeoJSON original y metadatos de consulta/extracción de forma limitada y reproducible. Registra fuente, modo, fecha UTC, filtros y recuento conforme al diseño actual.
4. Filtra los tipos distintos de `earthquake` según el requisito, registrando el motivo de exclusión. Trata `id` como identidad del evento y `updated` como identidad/orden de versión.
5. Compara versiones con estado persistente: publica novedades y revisiones posteriores; no publiques la misma versión ni una versión más antigua como novedad.
6. Publica con `id` como clave y los metadatos requeridos (`source`, `ingested_at`, `mode`, original). Espera la confirmación de Kafka antes de persistir el estado de publicación. Tolera el posible reenvío tras una caída.
7. Maneja timeouts, 429/5xx, errores de JSON y reintentos con espera progresiva sin borrar el estado válido anterior ni ocultar errores permanentes.
8. Separa carga histórica inicial, ingesta live y replay. El replay de demostración debe pasar por Kafka, mantener el tiempo original y quedar aislado/etiquetado como reproducción.
9. Ejecuta las comprobaciones pertinentes existentes; informa comandos, resultados observados y pruebas no ejecutadas.

## Criterios de calidad

- Implementa solo lo necesario para la ingesta solicitada; no añadas componentes, dependencias o abstracciones si el flujo existente satisface los requisitos.
- Estado suficiente para evitar republicaciones triviales sobrevive al reinicio.
- No confunde una respuesta completa de la ventana móvil con eventos nuevos.
- No sobrescribe una versión posterior con una anterior ni pierde la procedencia.
- Errores de fuente y broker se hacen visibles; no se convierten en respuestas de éxito vacías.
- Replay y datos live se distinguen sin alterar fechas originales.
- La configuración no contiene credenciales versionadas.

## Agentes que la utilizan

Backend es el responsable principal; Deploy la consulta al configurar el proceso; QA Tester la utiliza para derivar pruebas de ingesta y recuperación.

## Referencias

- [Enunciado](../../../docs/source-of-truth/ENUNCIADO.md)
- [FAQ del alumnado](../../../docs/source-of-truth/FAQ-ALUMNADO.md)
- [Fuentes y licencias](../../../docs/source-of-truth/FUENTES-Y-LICENCIAS.md)
