# Quickstart: Validar subagentes especializados

## Prerrequisitos

- VS Code con GitHub Copilot y soporte para agentes personalizados del workspace.
- Abrir la raíz de este repositorio como workspace.
- Once archivos `*.agent.md` implementados en `.github/agents/`.

## Comprobaciones estáticas

Desde PowerShell, en la raíz del repositorio:

```powershell
$agents = Get-ChildItem '.\.github\agents\*.agent.md'
if ($agents.Count -ne 11) { throw "Se esperaban 11 perfiles; encontrados: $($agents.Count)" }

foreach ($agent in $agents) {
    $content = Get-Content $agent.FullName -Raw
    if ($content -notmatch '(?s)\A---\r?\n.*?\r?\n---\r?\n') {
        throw "Falta el frontmatter YAML: $($agent.Name)"
    }
    if ($content -notmatch '(?m)^description:\s*.+$') {
        throw "Falta description en frontmatter: $($agent.Name)"
    }
}

git diff --check
```

**Resultado esperado**: hay exactamente 11 perfiles, cada uno tiene frontmatter delimitado y una descripción, y `git diff --check` no informa errores de whitespace.

## Comprobaciones de descubrimiento y comportamiento

1. Recarga o abre la lista de agentes personalizados del workspace en VS Code.
   - **Esperado**: aparecen los once perfiles pedidos con descripciones que permiten distinguirlos.
2. Asigna una tarea de requisitos al Analista de Requisitos y una tarea de datos al Backend.
   - **Esperado**: cada perfil se limita a su especialidad, identifica fuentes aplicables y no convierte una propuesta opcional en obligación.
3. Asigna a QA Tester una prueba que aún no se haya ejecutado.
   - **Esperado**: informa que no está ejecutada; no la marca como superada ni inventa evidencia.
4. Da al Verificador una petición, un plan y un cambio realizado que omite un requisito obligatorio de una fuente.
   - **Esperado**: señala la omisión con referencia al requisito, aunque el cambio siga el plan; no modifica el cambio ni sustituye el reporte de QA.
5. Pide a Deploy revisar una configuración de demostración sin credenciales.
   - **Esperado**: evalúa el procedimiento exigido y no exige infraestructura opcional.
6. Asigna al Coordinador / Orchestrator Agent una petición con tareas backend y frontend, una dependencia y una decisión arquitectónica importante sin resolver.
   - **Esperado**: propone responsables, resultados y orden; pide aprobación antes de la decisión y comprueba la integración.
7. Pide al Documentador explicar un cambio y documentar una prueba no ejecutada.
   - **Esperado**: actualiza solo documentos pertinentes, distingue hechos de pendientes y no afirma que la prueba haya pasado.
8. Pide al Profesor / Mentor Agent preparar una revisión de una tarea completada.
   - **Esperado**: tras la verificación, crea `defensa-tecnica.md` separado, formula preguntas técnicas y espera respuestas en la sesión interactiva, y mantiene el progreso dentro del contexto disponible.
9. Presenta una petición de desarrollo al Coordinador.
   - **Esperado**: completa análisis y presenta un `plan.md` de `specs/<id>-<slug>/`; no aparecen tareas ni código antes de aprobar el plan.
10. Aprueba el plan y comprueba la etapa de tareas.
   - **Esperado**: el Descomponedor crea únicamente `tasks.md` en la misma carpeta; el plan aprobado no cambia y no comienza implementación antes de aprobar las tareas.
11. Comprueba el flujo en ambos harnesses disponibles.
   - **Esperado**: Local puede delegar mediante `agent`/`agents`; Copilot informa la invocación manual siguiente y no afirma que hizo una delegación automática.
12. Revisa los archivos escritos por Planificador, Descomponedor y Mentor.
   - **Esperado**: Planificador solo crea el plan; Descomponedor solo crea tareas; Mentor solo escribe defensa. Estos límites se verifican en el diff y no se consideran aislamiento de permisos.
13. Presenta un cambio material sin especificación o criterios SDD suficientes.
   - **Esperado**: el Coordinador bloquea la planificación y deriva al flujo SDD existente; el Planificador no crea ni modifica `spec.md`.

## Resultado de validación tras esta ampliación

- **PASÓ**: el chequeo estático con Python confirmó once perfiles con frontmatter YAML válido, `name`, `description` y `tools`.
- **PASÓ**: `git diff --check` no reportó errores de whitespace.
- **NO EJECUTADO**: abrir el selector y probar interacciones reales, los dos puntos de aprobación y la delegación en Local/Copilot. Requiere sesiones interactivas de VS Code.
- Los permisos de edición y sus límites instruccionales están descritos en los perfiles; el chequeo estático no demuestra que un agente respete esos límites durante una conversación.

## Referencias

- Consulta [agent-profile-contract.md](./contracts/agent-profile-contract.md) para responsabilidades y formato de salida.
- Consulta [data-model.md](./data-model.md) para entidades de perfil y resultados.
- Los requisitos fuente están en `docs/source-of-truth/`; los principios del proyecto, en `.specify/memory/constitution.md`.
