# Quickstart: Validar subagentes especializados

## Prerrequisitos

- VS Code con GitHub Copilot y soporte para agentes personalizados del workspace.
- Abrir la raíz de este repositorio como workspace.
- Nueve archivos `*.agent.md` implementados en `.github/agents/`.

## Comprobaciones estáticas

Desde PowerShell, en la raíz del repositorio:

```powershell
$agents = Get-ChildItem '.\.github\agents\*.agent.md'
if ($agents.Count -ne 9) { throw "Se esperaban 9 perfiles; encontrados: $($agents.Count)" }

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

**Resultado esperado**: hay exactamente nueve definiciones, cada una tiene frontmatter delimitado y una descripción, y `git diff --check` no informa errores de whitespace.

## Comprobaciones de descubrimiento y comportamiento

1. Recarga o abre la lista de agentes personalizados del workspace en VS Code.
   - **Esperado**: aparecen los nueve perfiles pedidos con descripciones que permiten distinguirlos.
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
   - **Esperado**: formula preguntas técnicas y espera las respuestas, corrige con explicaciones y mantiene el progreso dentro del contexto disponible.

## Resultado de validación de esta implementación

- **Pasó** la comprobación estática de que hay exactamente nueve archivos de perfil, cada uno con delimitadores de frontmatter y campos `name`, `description` y `tools`.
- **Pasaron** las comprobaciones de límites de rol: Verificador está limitado a lectura y búsqueda; QA diferencia resultados esperados, observados y no ejecutados.
- **Pasó** la revisión de patrones de exclusión: dependencias, compilados y configuración local quedan ignorados, mientras `.env.example` sigue siendo rastreable.
- **No ejecutado**: abrir el selector de agentes y probar conversaciones reales con cada perfil. Esas comprobaciones dependen del harness interactivo de VS Code y no se pudieron observar desde esta ejecución.

## Referencias

- Consulta [agent-profile-contract.md](./contracts/agent-profile-contract.md) para responsabilidades y formato de salida.
- Consulta [data-model.md](./data-model.md) para entidades de perfil y resultados.
- Los requisitos fuente están en `docs/source-of-truth/`; los principios del proyecto, en `.specify/memory/constitution.md`.
