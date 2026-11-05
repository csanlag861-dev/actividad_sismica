---
name: consumidor-calidad-postgresql
description: 'Implement or troubleshoot the Python aiokafka consumer and PostgreSQL persistence for the seismic observatory. Use for validation, transformations, spatial country enrichment, quarantine, event revisions, idempotency, transactions, or offset handling.'
---

# Consumidor, calidad y PostgreSQL

## Propósito

Proteger el significado y la trazabilidad de los mensajes desde Kafka hasta PostgreSQL, incluyendo revisiones, datos rechazados y enriquecimiento geográfico.

## Cuándo utilizarla

- Al implementar el consumidor o cambiar su esquema, transacciones o persistencia.
- Al modificar reglas de validación, limpieza, cuarentena o asignación de país.
- Al investigar duplicados, replay mezclado con live, offsets o revisiones.

## Procedimiento

1. Inspecciona la implementación, esquema, dependencias, política de commit y pruebas. No asumas nombres de tablas ni añadas bibliotecas sin necesidad justificada.
2. Define grano e identidad: evento por `id`; versión por `id` y `updated`. Comprueba qué campos son obligatorios y cuáles admiten `null`.
3. Conserva el original y metadatos necesarios. Convierte unidades y fechas de forma explícita; USGS usa timestamps Unix en milisegundos y tiempos UTC. No intercambies longitud/latitud.
4. Valida tipos y rangos sin convertir desconocido en cero. No rechaces automáticamente magnitud o profundidad negativas; investiga su significado. Filtra tipos no sísmicos con motivo.
5. Enriquece usando el catálogo Natural Earth entregado, con CRS compatibles y punto dentro de polígono. Documenta fronteras/coincidencias; deja país desconocido si no hay cobertura. Nunca asignes el país más cercano ni derives el país del texto `place`.
6. Persiste versiones de forma idempotente. Solo actualiza el estado actual si la versión recibida es más reciente; mantiene la versión observada y aplica las restricciones/upserts del esquema real.
7. Guarda los cambios PostgreSQL en una transacción antes de confirmar el offset Kafka. Para un mensaje rechazado, persiste cuarentena con motivo y referencia al original antes de avanzar. No se requiere transacción distribuida.
8. Aísla replay o identifica su modo sin mezclarlo con live. Confirma que las agregaciones se basan en estado actual y no cuentan revisiones como eventos nuevos.
9. Ejecuta pruebas para mensaje repetido, revisión posterior, revisión antigua, coordenadas inválidas y reinicio; informa evidencia observada.

## Criterios de calidad

- Reprocesar un mensaje no duplica efectos y una versión antigua no revierte el estado.
- El commit del offset ocurre después de persistencia confirmada.
- Rechazos duraderos conservan el motivo y referencia; errores de base no se disfrazan como resultado correcto.
- Nulos permanecen nulos y «Sin país asignado» se conserva como categoría válida.
- Transformaciones, unidades, grano e índices son explicables y medibles donde corresponda.

## Agentes que la utilizan

Backend la aplica a consumidor y almacenamiento; QA Tester toma de ella escenarios de idempotencia, cuarentena y recuperación; Verificador contrasta cobertura contra las fuentes sin ejecutar cambios.

## Referencias

- [Enunciado](../../../docs/source-of-truth/ENUNCIADO.md)
- [FAQ del alumnado](../../../docs/source-of-truth/FAQ-ALUMNADO.md)
- [Fuentes y licencias](../../../docs/source-of-truth/FUENTES-Y-LICENCIAS.md)
- [Constitución](../../../.specify/memory/constitution.md)
