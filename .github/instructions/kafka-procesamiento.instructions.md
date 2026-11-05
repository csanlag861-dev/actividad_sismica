---
description: "Use for Kafka producers, consumers, message identity, event revisions, retries, offsets, historical replay, or live processing."
---
# Kafka y procesamiento

- Conserva la identidad del evento y la versión de origen; las repeticiones no deben duplicar efectos y una versión antigua no debe sustituir a otra más reciente.
- Mantén el orden de persistencia y confirmación de offsets exigido por las fuentes del proyecto; informa fallos que puedan afectar a recuperación o pérdida de datos.
- Conserva la procedencia y el modo `live`/`replay`; no mezcles silenciosamente reproducción histórica con datos actuales.
- Basa cada cambio en los requisitos SDD y del enunciado aplicables. Para procedimientos de ingesta o consumidor, aplica las skills `.github/skills/ingesta-usgs-aiokafka/SKILL.md`, `.github/skills/consumidor-calidad-postgresql/SKILL.md` y, cuando corresponda, `.github/skills/pruebas-aceptacion-replay/SKILL.md`.
