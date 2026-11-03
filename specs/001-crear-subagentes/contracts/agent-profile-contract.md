# Agent Profile Contract

## Discovery and file shape

- Provide exactly one VS Code workspace custom-agent file for each specified role under `.github/agents/`.
- Use the `*.agent.md` naming convention.
- Each file has YAML frontmatter delimited by `---` and a non-empty, role-specific `description`.
- Where declared, `name` matches the profile's requested role; `tools` uses documented aliases and excludes unnecessary capabilities.
- The Markdown body describes purpose, responsibilities, boundaries, working approach, and output expectations.

## Shared behavioral contract

All six profiles:

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
