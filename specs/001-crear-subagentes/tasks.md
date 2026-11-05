# Tasks: Subagentes especializados

**Input**: Design documents from `specs/001-crear-subagentes/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: No separate automated test suite was requested. Validate the workspace agent files and behavior using the scenarios in `quickstart.md`.

**Organization**: Tasks are grouped by user story. Each profile is implemented in its own file so independent roles can be developed in parallel.

## Phase 1: Setup

**Purpose**: Add ignore rules required for the existing Git and Node.js workspace.

- [X] T001 Add root `.gitignore` patterns for `node_modules/`, build output, logs, local `.env*` files (except `.env.example`), temporary files, and OS/editor artifacts.

---

## Phase 2: Foundational

No separate shared implementation is required: the source hierarchy and shared profile behavior are specified in `contracts/agent-profile-contract.md`; each user-story profile applies that contract independently.

---

## Phase 3: User Story 1 - Alinear el trabajo con los requisitos (Priority: P1)

**Goal**: Provide a requirements analyst that traces work to authoritative sources and produces prioritized, testable criteria without expanding scope.

**Independent Test**: Assign an observatory request containing an optional suggestion; confirm the profile returns traceable requirements and acceptance criteria without misclassifying the suggestion as mandatory.

### Implementation for User Story 1

- [X] T002 [US1] Create `.github/agents/analista-requisitos.agent.md` with discovery metadata, source hierarchy, classification of requirements/decisions/proposals, prioritization, acceptance criteria, dependency reporting, and decision boundaries.

**Checkpoint**: The requirements analyst can independently turn a request into a source-traceable and testable brief.

---

## Phase 4: User Story 2 - Implementar por especialidad (Priority: P1)

**Goal**: Provide separate Backend and Frontend profiles that implement their areas while respecting the shared API/data responsibilities and surfacing cross-area dependencies.

**Independent Test**: Assign a backend data-flow task and a panel task; confirm each profile stays within its role and refers to the project’s applicable requirements and interfaces.

### Implementation for User Story 2

- [X] T003 [P] [US2] Create `.github/agents/backend.agent.md` with ingestion, Kafka, data quality, idempotency/revisions, persistence/quarantine, API-contract responsibilities, and backend-specific output boundaries.
- [X] T004 [P] [US2] Create `.github/agents/frontend.agent.md` with processed-data integration, map/table/filters/analysis/status responsibilities, API-boundary requirements, and frontend-specific output boundaries.

**Checkpoint**: Backend and Frontend profiles can be assigned independently and make cross-area dependencies explicit.

---

## Phase 5: User Story 3 - Verificar resultados con evidencia (Priority: P2)

**Goal**: Provide a QA profile that reports acceptance checks using observed evidence and never labels unexecuted tests as passed.

**Independent Test**: Assign a check that cannot run; confirm the profile reports it as not run, distinguishes expected from observed results, and states the blocker.

### Implementation for User Story 3

- [X] T005 [US3] Create `.github/agents/qa-tester.agent.md` with acceptance-test execution, expected/observed/evidence reporting, pass/fail/not-run states, and a boundary from source-coverage verification.

**Checkpoint**: QA Tester reports only outcomes supported by checks actually performed.

---

## Phase 6: User Story 4 - Preparar un despliegue reproducible (Priority: P2)

**Goal**: Provide a Deploy profile that validates the required local startup and publication procedure without exposing secrets or requiring optional infrastructure.

**Independent Test**: Ask the profile to review the project’s deployment steps; confirm it covers required services, persistence and public access while treating additional infrastructure as optional.

### Implementation for User Story 4

- [X] T006 [US4] Create `.github/agents/deploy.agent.md` with the required Compose services, reproducible startup, tunnel/frontend publication, secret handling, health/retry and deployment-validation boundaries.

**Checkpoint**: Deploy can assess a reproducible demonstration procedure and reports environment-dependent checks it could not perform.

---

## Phase 7: User Story 5 - Verificar la correspondencia del trabajo (Priority: P1)

**Goal**: Provide a read-only Verificador that compares the request, plan, completed work and applicable source requirements, identifying omissions, deviations and unnecessary additions.

**Independent Test**: Provide a plan and result that agree with each other but omit a mandatory source requirement; confirm the profile reports the omission with source references and does not edit the result or claim tests passed.

### Implementation for User Story 5

- [X] T007 [US5] Create `.github/agents/verificador.agent.md` with read/search-only tools, request-plan-result-source comparison, traceable findings, missing/extra work detection, unverifiable status and explicit separation from QA execution.

**Checkpoint**: Verificador flags source omissions even when implementation follows the plan and keeps its review read-only.

---

## Phase 8: User Story 6 - Coordinar el trabajo especializado (Priority: P1)

**Goal**: Provide a Coordinator that understands user requests, assigns bounded work to the specialist profiles, orders dependencies, supervises acceptance, and obtains approval for material decisions.

**Independent Test**: Assign a cross-specialty task with a dependency and unresolved architectural decision; confirm the Coordinator presents an execution sequence, delegates specific deliverables, and asks the user before the material decision.

### Implementation for User Story 6

- [X] T008 [US6] Create `.github/agents/coordinador.agent.md` with `agent` delegation, access to the named specialist profiles, request analysis, assignment/dependency planning, integration and quality gates, incident handling, progress reporting, and user-approval boundaries.

**Checkpoint**: Coordinador / Orchestrator Agent can orchestrate the other eight profiles without taking over their specialties or changing unapproved scope.

---

## Phase 9: User Story 7 - Mantener y explicar la documentación (Priority: P2)

**Goal**: Provide a Documentador that keeps technical and didactic project documentation synchronized with code and SDD, records verified decisions and test evidence, and prepares defense materials.

**Independent Test**: Provide a completed code change, a confirmed decision, and an unexecuted test; confirm the profile documents only supported facts and clearly leaves the test unverified.

### Implementation for User Story 7

- [X] T009 [US7] Create `.github/agents/documentador.agent.md` with documentation-only editing boundaries, technical and learner-level explanations, SDD synchronization, decision alternatives, observed-test reporting, and defense preparation.

**Checkpoint**: Documentador produces traceable, understandable, synchronized documentation without editing application code or inventing evidence.

---

## Phase 10: User Story 8 - Reforzar el aprendizaje y la autonomía (Priority: P2)

**Goal**: Provide a read-only Profesor / Mentor Agent that assesses understanding interactively, corrects misconceptions, supports practice and defense preparation, and limits progress claims to accessible context.

**Independent Test**: Present a completed task; confirm the profile asks technical questions and waits for answers before evaluating them, proposes targeted reinforcement, and does not claim cross-session memory.

### Implementation for User Story 8

- [X] T010 [US8] Create `.github/agents/profesor-mentor.agent.md` with read/search-only tools, interactive question-and-response learning, respectful correction, targeted exercises, autonomy assessment, defense simulation, and explicit conversation-scoped progress tracking.

**Checkpoint**: Profesor / Mentor Agent supports the student's ability to explain, justify, modify, and debug work rather than giving answers before assessing understanding.

---

## Phase 11: Polish & Cross-Cutting Concerns

**Purpose**: Validate that all nine profiles satisfy the shared contract and can be discovered in the intended environment.

- [X] T011 Revalidate all nine `.github/agents/*.agent.md` profiles and the cross-cutting SDD documents against `specs/001-crear-subagentes/quickstart.md`, `.specify/memory/constitution.md`, and `docs/source-of-truth/`; mark unavailable interactive VS Code harness checks as not run.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; add root ignore rules.
- **Foundational (Phase 2)**: No implementation task; the shared contract is already defined in the design artifact.
- **User Stories (Phases 3-10)**: Depend only on setup. The roles have separate files and can be implemented independently.
- **Polish (Phase 11)**: Depends on all nine agent profiles and the SDD updates being present.

### User Story Dependencies

- **User Story 1 (P1)**: Independent after setup.
- **User Story 2 (P1)**: Independent after setup; Backend and Frontend profile files can be created in parallel.
- **User Story 3 (P2)**: Independent after setup.
- **User Story 4 (P2)**: Independent after setup.
- **User Story 5 (P1)**: Independent after setup.
- **User Story 6 (P1)**: Independent after setup; delegates to the other profiles when coordinating real tasks.
- **User Story 7 (P2)**: Independent after setup.
- **User Story 8 (P2)**: Independent after setup.

### Parallel Opportunities

- **T003 and T004** can run in parallel because they create distinct profile files.
- **T002-T010** can be implemented in parallel after setup because each creates a distinct file and the shared behavioral contract already exists.
- **T011** runs after all nine profile implementation tasks and synchronized SDD artifacts.

---

## Parallel Example: User Story 2

```text
Task: T003 Create .github/agents/backend.agent.md
Task: T004 Create .github/agents/frontend.agent.md
```

## Implementation Strategy

### MVP First (User Story 1)

1. Complete setup.
2. Implement the Analyst profile in T002.
3. Validate its source classification and acceptance-criteria output against the US1 independent test.

### Incremental Delivery

1. Add Backend and Frontend profiles (US2) and validate their boundaries independently.
2. Add QA Tester (US3) and Deploy (US4), validating observed evidence and reproducibility.
3. Add Verificador (US5) and validate read-only source/request/plan/result comparison.
4. Add Coordinador / Orchestrator Agent (US6), Documentador (US7), and Profesor / Mentor Agent (US8); validate their delegation, documentation, and learning boundaries.
5. Run T011 across all profiles and the quickstart scenarios.

## Notes

- Every implementation task has a task ID, checkbox, user-story label where applicable, and exact file path.
- `[P]` marks only the distinct Backend and Frontend files that can be authored independently.
- No standalone test-code tasks are included; the feature's runnable validation scenarios are in `quickstart.md`.
