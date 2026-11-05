# Implementation Plan: Subagentes especializados

**Branch**: `001-crear-subagentes` | **Date**: 2026-11-03 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `specs/001-crear-subagentes/spec.md`

## Summary

Mantener once agentes personalizados de ámbito workspace para el Observatorio, incorporando Planificador SDD y Descomponedor de tareas, y establecer un flujo SDD de dos aprobaciones coordinado por el usuario. Cada perfil tendrá responsabilidades y límites diferenciados; Planificador y Descomponedor crearán exclusivamente sus artefactos designados. Tras completar y verificar la implementación, Documentador y Mentor generarán documentos distintos en la carpeta de la tarea.

## Technical Context

**Language/Version**: Markdown con frontmatter YAML para perfiles de agente.

**Primary Dependencies**: Descubrimiento de agentes personalizados de VS Code y compatibilidad del harness; once archivos `.agent.md`.

**Storage**: Archivos versionados en `.github/agents/`.

**Testing**: Inspección de frontmatter y contenido, comprobaciones estáticas del repositorio y validación de descubrimiento/uso desde el selector y harnesses disponibles. Los escenarios interactivos de aprobación se reportan como no ejecutados si no se pueden probar.

**Target Platform**: Workspace de este repositorio en VS Code con GitHub Copilot.

**Project Type**: Personalización de agentes para un workspace existente.

**Performance Goals**: No hay objetivo de rendimiento en ejecución; cada agente debe poder identificarse por una descripción breve y específica en el selector y al delegar.

**Constraints**: Exactamente once perfiles; dos aprobaciones explícitas antes de descomponer e implementar; `specs/<id>-<slug>/` para tareas futuras; respetar la constitución y las fuentes; no convertir propuestas opcionales en requisitos; no exponer credenciales; distinguir evidencia observada de supuestos; no modificar código de aplicación en esta configuración. `agents` y `handoffs` son capacidades del harness Local, no del Copilot harness; restricciones de ruta en instrucciones no constituyen aislamiento técnico.

**Scale/Scope**: Once definiciones, cambios acotados del coordinador, documentación y trazabilidad SDD de la configuración en `specs/001-crear-subagentes/`.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principio | Comprobación del plan | Estado |
|---|---|---|
| I. Requisitos del enunciado y la rúbrica | Los agentes consultan el enunciado y los criterios aplicables; los perfiles de backend y frontend no alteran las obligaciones descritas. | PASS |
| II. Trazabilidad y tratamiento fiel de los datos | Los perfiles conservan la procedencia de requisitos y datos; las instrucciones compartidas prohíben convertir ausencias en cero o muestras ficticias en análisis real. | PASS |
| III. Ingesta persistente e idempotente | Backend recibe responsabilidades acordes con revisiones, persistencia e idempotencia; los demás no sustituyen ni redefinen ese comportamiento. | PASS |
| IV. Análisis comparable y evidencia verificable | QA informa resultados observados; Verificador contrasta cobertura y correspondencia sin declarar pruebas ejecutadas por inferencia. | PASS |
| V. Decisiones explícitas y alcance proporcionado | Los perfiles identifican decisiones y propuestas; no exigen servicios o automatizaciones opcionales. | PASS |

**Gate result**: PASS. No se identifican conflictos constitucionales ni excepciones que requieran justificación.

## Project Structure

### Documentation (this feature)

```text
specs/001-crear-subagentes/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── agent-profile-contract.md
└── tasks.md
```

### Source Code (repository root)

```text
.github/
├── agents/
│   ├── backend.agent.md
│   ├── frontend.agent.md
│   ├── analista-requisitos.agent.md
│   ├── qa-tester.agent.md
│   ├── deploy.agent.md
│   ├── verificador.agent.md
│   ├── coordinador.agent.md
│   ├── documentador.agent.md
│   ├── profesor-mentor.agent.md
│   ├── planificador-sdd.agent.md
│   └── descomponedor-tareas.agent.md
└── skills/
    └── ... (existing project skills; unchanged)
```

**Structure Decision**: Usar `.github/agents/` para definiciones workspace y conservar las instrucciones de coordinación en `.github/instructions/`. Las tareas futuras utilizan `specs/<id>-<slug>/`; esta actualización pertenece a la feature existente y no crea una spec para una funcionalidad real del observatorio.

## Design Overview

- Compartir reglas de precedencia de fuentes, alcance y honestidad de evidencia en los once perfiles; adaptar las responsabilidades al dominio de cada rol.
- Analista de Requisitos define criterios y trazabilidad; Backend y Frontend implementan dentro de sus límites; Deploy se ocupa de la puesta en marcha y publicación; QA ejecuta o verifica pruebas; Verificador compara solicitud, plan y resultado con los entregables obligatorios fuente; Coordinador / Orchestrator Agent asigna y supervisa; Documentador mantiene explicaciones y evidencia; Profesor / Mentor Agent entrena la comprensión y autonomía.
- QA Tester y Verificador se mantienen como funciones distintas: QA aporta resultados observados de pruebas; Verificador informa cobertura, discrepancias y elementos ausentes, y declara explícitamente lo que no pudo comprobar.
- Configurar herramientas por mínimo privilegio: roles reciben solo las necesarias. El Mentor necesita `edit` para generar defensa técnica, con una limitación explícita de uso a ese archivo que no es técnicamente imponible mediante un permiso de ruta.
- Implementar el flujo de dos aprobaciones con instrucciones y turnos explícitos: plan antes de tareas, tareas antes de implementación. Las instrucciones orientan, pero no son un mecanismo determinista de control.
- Antes de encargar un plan, comprobar que existe una especificación y criterios SDD suficientes; si falta una base para un cambio material, detenerse y remitir al flujo SDD existente.
- Para Local, configurar delegación por `agent` y lista `agents`; para Copilot, describir invocación manual porque ese harness no admite esos campos.
- No imponer un modelo concreto, herramientas de test, automatización de informes ni infraestructura que las fuentes declaran opcionales.
- El seguimiento de aprendizaje del Profesor/Mentor es conversacional; no se promete persistencia entre sesiones ni se escriben perfiles de aprendizaje sin autorización del alumno. La única excepción de escritura es el documento de defensa aprobado.

## Post-Design Constitution Check

| Principio | Comprobación del diseño | Estado |
|---|---|---|
| I. Requisitos del enunciado y la rúbrica | Las definiciones orientan a cada agente hacia sus obligaciones aplicables y evitan ampliar la rúbrica. | PASS |
| II. Trazabilidad y tratamiento fiel de los datos | Los perfiles deben consultar fuentes y conservar su significado y limitaciones. | PASS |
| III. Ingesta persistente e idempotente | Se asignan estas responsabilidades al perfil Backend y no se contradicen en los demás perfiles. | PASS |
| IV. Análisis comparable y evidencia verificable | QA informa evidencia observada y Verificador informa cobertura; ninguno debe afirmar comprobaciones que no hizo. | PASS |
| V. Decisiones explícitas y alcance proporcionado | Alcance de once agentes, permisos mínimos y opciones no obligatorias están delimitados. | PASS |

**Post-design gate result**: PASS. Los puntos de aprobación son una decisión explícita del estudiante para el flujo de agentes; no alteran las obligaciones del sistema sísmico ni las fuentes del profesor. El límite de edición del Mentor es instruccional y queda documentado como limitación.

## Complexity Tracking

No hay violaciones de la constitución ni complejidad adicional que requiera justificación.
