# Implementation Plan: Subagentes especializados

**Branch**: `001-crear-subagentes` | **Date**: 2026-11-03 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `specs/001-crear-subagentes/spec.md`

## Summary

Crear nueve agentes personalizados de ámbito workspace para VS Code Copilot: Backend, Frontend, Analista de Requisitos (Product Manager), QA Tester, Deploy, Verificador, Coordinador / Orchestrator Agent, Documentador y Profesor / Mentor Agent. Cada perfil tendrá un propósito, descripción descubrible, permisos mínimos de herramientas, responsabilidades, límites y formato de salida. Sus instrucciones seguirán la constitución y las fuentes de verdad, separarán obligaciones de decisiones y propuestas opcionales, y mantendrán diferenciadas la implementación, la ejecución de pruebas, la verificación, la coordinación, la documentación y la mentoría.

## Technical Context

**Language/Version**: Markdown con frontmatter YAML para perfiles de agente.

**Primary Dependencies**: Descubrimiento de agentes personalizados de VS Code Copilot; seis archivos `.agent.md`.

**Storage**: Archivos versionados en `.github/agents/`.

**Testing**: Inspección de frontmatter y contenido, comprobaciones estáticas del repositorio y validación de descubrimiento/uso desde el selector de agentes de VS Code.

**Target Platform**: Workspace de este repositorio en VS Code con GitHub Copilot.

**Project Type**: Personalización de agentes para un workspace existente.

**Performance Goals**: No hay objetivo de rendimiento en ejecución; cada agente debe poder identificarse por una descripción breve y específica en el selector y al delegar.

**Constraints**: Exactamente nueve perfiles solicitados; permisos mínimos por rol; respetar la constitución y los documentos fuente; no convertir propuestas opcionales en requisitos; no exponer credenciales; distinguir evidencia observada de supuestos; no modificar el código de la aplicación como parte de esta feature.

**Scale/Scope**: Nueve definiciones, una por rol, y documentación de su contrato y validación en `specs/001-crear-subagentes/`.

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
│   └── profesor-mentor.agent.md
└── skills/
    └── ... (existing project skills; unchanged)
```

**Structure Decision**: Usar `.github/agents/` para definiciones de ámbito workspace compartidas con el repositorio y visibles en VS Code. No crear agentes a nivel de usuario, instrucciones globales ni agentes adicionales. Los artefactos de diseño y validación permanecen en el directorio de esta feature.

## Design Overview

- Compartir reglas de precedencia de fuentes, alcance y honestidad de evidencia en los nueve perfiles; adaptar las responsabilidades al dominio de cada rol.
- Analista de Requisitos define criterios y trazabilidad; Backend y Frontend implementan dentro de sus límites; Deploy se ocupa de la puesta en marcha y publicación; QA ejecuta o verifica pruebas; Verificador compara solicitud, plan y resultado con los entregables obligatorios fuente; Coordinador / Orchestrator Agent asigna y supervisa; Documentador mantiene explicaciones y evidencia; Profesor / Mentor Agent entrena la comprensión y autonomía.
- QA Tester y Verificador se mantienen como funciones distintas: QA aporta resultados observados de pruebas; Verificador informa cobertura, discrepancias y elementos ausentes, y declara explícitamente lo que no pudo comprobar.
- Configurar herramientas por mínimo privilegio: roles de implementación y documentación reciben solo las herramientas necesarias; QA y Deploy reciben ejecución según su tarea; Verificador y Profesor/Mentor permanecen en modo de lectura; Coordinador dispone de delegación y lectura, pero no de permisos para realizar en nombre de otros cambios no revisados.
- No imponer un modelo concreto, herramientas de test, automatización de informes ni infraestructura que las fuentes declaran opcionales.
- El seguimiento de aprendizaje del Profesor/Mentor es conversacional; no se promete persistencia entre sesiones ni se escriben perfiles de aprendizaje sin autorización del alumno.

## Post-Design Constitution Check

| Principio | Comprobación del diseño | Estado |
|---|---|---|
| I. Requisitos del enunciado y la rúbrica | Las definiciones orientan a cada agente hacia sus obligaciones aplicables y evitan ampliar la rúbrica. | PASS |
| II. Trazabilidad y tratamiento fiel de los datos | Los perfiles deben consultar fuentes y conservar su significado y limitaciones. | PASS |
| III. Ingesta persistente e idempotente | Se asignan estas responsabilidades al perfil Backend y no se contradicen en los demás perfiles. | PASS |
| IV. Análisis comparable y evidencia verificable | QA informa evidencia observada y Verificador informa cobertura; ninguno debe afirmar comprobaciones que no hizo. | PASS |
| V. Decisiones explícitas y alcance proporcionado | Alcance de nueve agentes, permisos mínimos y opciones no obligatorias están delimitados. | PASS |

**Post-design gate result**: PASS. No se requieren excepciones ni seguimiento adicional de la constitución.

## Complexity Tracking

No hay violaciones de la constitución ni complejidad adicional que requiera justificación.
