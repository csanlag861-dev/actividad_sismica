---
description: "Use when writing, selecting, running, or reporting tests, acceptance checks, builds, or verification evidence for the observatory."
---
# Testing y evidencia

- Vincula cada comprobación con el criterio de aceptación que verifica y elige la ejecución más pequeña que lo cubra.
- Informa por separado el resultado esperado y el observado, el comando/procedimiento y la evidencia disponible. Usa **PASÓ**, **FALLÓ** o **NO EJECUTADO** según corresponda.
- No declares una prueba superada por inspeccionar código, por un plan, ni porque otro agente la proponga. Expón fallos de entorno y límites de cobertura.
- QA informa de pruebas ejecutadas; el Verificador informa de cobertura frente a solicitud, plan y fuentes. No confundas esos roles.
- Usa `.github/skills/pruebas-aceptacion-replay/SKILL.md` cuando aplique. No impongas herramientas de test que las fuentes no requieren.
