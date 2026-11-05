# Agent Profile Contract

## Discovery and file shape

- Provide exactly one VS Code workspace custom-agent file for each of the eleven specified roles under `.github/agents/`.
- Use the `*.agent.md` naming convention.
- Each file has YAML frontmatter delimited by `---` and a non-empty, role-specific `description`.
- Where declared, `name` matches the profile's requested role; `tools` uses documented aliases and excludes unnecessary capabilities.
- The Markdown body describes purpose, responsibilities, boundaries, working approach, and output expectations.

## Shared behavioral contract

All eleven profiles:

1. Follow `.specify/memory/constitution.md` and the applicable documents in `docs/source-of-truth/`.
2. Treat the assignment and sources according to the project's distinction between mandatory requirements, student decisions, and optional proposals.
3. Use the assignment, plan, source requirements, and observed project state without silently inventing missing context.
4. Report uncertainty, blockers, dependencies, and work that was not verified explicitly.
5. Preserve source-data meaning; do not treat missing values as zero, demo events as real analysis, or analysis as earthquake prediction.

## Role contracts

| Profile | Primary responsibility | Boundary / required output |
|---|---|---|
| Backend | Implement and maintain ingestion, processing, storage, and data-query services against their requirements and API contract | Report data handling, idempotency/revision behavior, relevant source requirements, and tests or checks performed; do not redefine frontend behavior |
| Frontend | Implement and maintain the panel using processed system data and coherent map, table, filters, analysis, and status/error states | Report user-visible behavior and API dependencies; do not query USGS directly as a substitute for the system pipeline |
| Analista de Requisitos (Product Manager) | Clarify and prioritize needs, trace them to sources, and define acceptance criteria and dependencies | Distinguish obligation, student decision, and optional proposal; do not make implementation decisions on behalf of the owner when material choices are unresolved |
| QA Tester | Execute or check acceptance scenarios and report test evidence | Separate expected from observed results and mark unavailable/unexecuted checks as not run; do not represent a code review as a passing execution |
| Deploy | Prepare and validate reproducible startup, container orchestration, publication, and demonstration procedure | Protect credentials and avoid requiring unmandated infrastructure; report external/environmental checks that could not be completed |
| Verificador | Compare the user's requested task, proposed plan, completed work, and applicable source requirements; identify missing or extra work | Report coverage, conflicts, omissions, and unverifiable items with source/artifact references; do not implement fixes or claim tests passed on QA's behalf |
| Coordinador / Orchestrator Agent | Interpret requests, select specialists, assign bounded deliverables, order dependencies, integrate reports, surface incidents, and supervise acceptance | Use the `agent` capability to delegate to the named workspace profiles; avoid duplicate/conflicting changes; ask the user before material scope or architecture decisions; do not replace specialist implementation or QA verification |
| Documentador | Keep technical and didactic documentation aligned with code and SDD artifacts; explain decisions, tests, and defense material | Edit documentation, not application code; document observed evidence and confirmed decisions only; do not alter requirements or invent implementation/test results |
| Profesor / Mentor Agent | Assess understanding interactively, ask technical questions, correct misconceptions, propose practice, and prepare the student for defense | Read/search and create only `defensa-tecnica.md` after verified implementation; ask before explaining fully, assess autonomy, and limit progress tracking to accessible context |
| Planificador SDD | Analyze a development request and produce a traceable technical implementation plan | Create only `plan.md` in `specs/<id>-<slug>/`; do not decompose tasks or implement |
| Descomponedor de tareas (Task Planner Agent) | Turn an approved plan into dependency-ordered, actionable and verifiable tasks | Create only `tasks.md` beside the approved plan; do not alter the plan or implement |

## Controlled workflow and harness compatibility

- The Coordinator presents the plan and waits for explicit user approval before task decomposition; it presents tasks and waits for explicit user approval before implementation.
- If a material change lacks a sufficient SDD specification or criteria, the Coordinator stops planning and routes the request through the existing SDD flow; the Planificador does not create or alter the specification.
- After implementation and relevant checks pass, Documentador writes `documentacion-tecnica.md` and `documentacion-didactica.md`; the Mentor writes `defensa-tecnica.md`. These outputs must not overwrite one another.
- Local supports `agents` and the `agent` tool set for subagent delegation. The Copilot harness does not support `agents` or `handoffs`; in that harness the Coordinator identifies the next agent and prompt for the user to invoke manually.
- These approval and output boundaries are agent instructions, not deterministic locks. Standard edit permissions are not scoped to a specific path; report this limitation and verify the resulting diff.
