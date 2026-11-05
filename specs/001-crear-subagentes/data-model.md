# Data Model: Subagentes especializados

## Entities

### Agent Profile

A workspace-discoverable specialist definition.

| Field | Meaning | Validation |
|---|---|---|
| `name` | Human-readable role name, optionally declared in frontmatter | Must identify one of the eleven specified roles |
| `description` | Discovery text used in the picker and when matching tasks for delegation | Required, concise, and specific to the role |
| `tools` | Tool aliases available to the profile | Must be limited to capabilities needed by that role |
| `purpose` | Outcome the profile is responsible for | Must not duplicate another profile's primary outcome |
| `responsibilities` | Work and deliverables within the profile's specialty | Must be consistent with applicable project sources |
| `boundaries` | Work the profile must not claim or perform as its primary responsibility | Must preserve hand-offs and scope distinctions |
| `output_format` | Information the profile returns to its caller | Must make completion, evidence, and unresolved items clear |

The eleven profile instances are Backend, Frontend, Analista de Requisitos (Product Manager), QA Tester, Deploy, Verificador, Coordinador / Orchestrator Agent, Documentador, Profesor / Mentor Agent, Planificador SDD, and Descomponedor de tareas (Task Planner Agent).

### Requirement

A project obligation or decision used to guide and assess work.

| Field | Meaning | Validation |
|---|---|---|
| `source` | Applicable authoritative project document | Must identify the source and relevant requirement or section |
| `classification` | Requirement, student decision, or optional proposal | Guides must not elevate optional proposals to mandatory requirements |
| `acceptance_evidence` | Observable result or evidence that demonstrates coverage | Must not be inferred from implementation intent alone |

### Verification Finding

A result reported when the Verificador compares requested work, its plan, the completed result, and source requirements.

| Field | Meaning | Validation |
|---|---|---|
| `comparison` | Compared pair or coverage dimension | Must identify request, plan, result, or source requirement |
| `status` | Covered, discrepancy, missing, or not verifiable | Missing context must result in not verifiable, not assumed compliance |
| `reference` | Requirement or task source supporting the finding | Must be included when a source requirement is implicated |
| `evidence` | Observed artifact or inspection supporting the finding | Must be distinguished from expected behavior |

### Test Result

An observed outcome recorded by QA Tester for an acceptance check.

| Field | Meaning | Validation |
|---|---|---|
| `expected` | Outcome required by the acceptance scenario | Must be stated separately from observation |
| `observed` | Outcome actually seen when executed or otherwise checked | Must not be invented or inferred |
| `evidence` | Log, output, screenshot, or reproducible observation | Required to claim a test passed |
| `status` | Passed, failed, or not run | A not-run test must never be reported as passed |

### Coordination Assignment

A bounded unit of work assigned by the Coordinador to one specialist or a dependency-ordered set of specialists.

| Field | Meaning | Validation |
|---|---|---|
| `request` | User outcome being addressed | Must remain traceable to the original request |
| `assignee` | Specialist profile responsible for a deliverable | Must identify one primary owner per deliverable |
| `deliverable` | Concrete expected result | Must be independently reviewable |
| `dependencies` | Work that must precede the assignment | Must identify order and blockers |
| `approval_required` | Whether an unresolved material scope/architecture decision needs user approval | Must pause that decision until user approval |

### SDD Workflow Checkpoint

A conversational approval boundary before moving from planning to task decomposition and from tasks to implementation.

| Field | Meaning | Validation |
|---|---|---|
| `stage` | Analysis, plan, tasks, implementation, or documentation | Must match the current workflow stage |
| `artifact` | Plan or task list reviewed at the checkpoint | Must be the artifact produced for the task |
| `approval_status` | Pending, approved, or changes requested | Only explicit user approval allows the next stage |
| `next_action` | Action or agent that may proceed | Must remain blocked while approval is pending |
| `harness` | Harness used for the current agent session | Must determine whether delegation is direct or manual |

### Learning Checkpoint

A conversation-scoped record of the learner's demonstrated understanding and next learning need.

| Field | Meaning | Validation |
|---|---|---|
| `concept` | Project concept or decision being assessed | Must relate to inspected or discussed project work |
| `learner_response` | Student's answer to a mentor question | Must be observed before correction is presented |
| `understanding` | Strengths and misconceptions demonstrated | Must be grounded in the response |
| `next_step` | Explanation, exercise, or defense question | Must target a demonstrated learning need |
| `scope` | Available conversation/context lifetime | Must not imply cross-session persistence without an approved accessible record |

## Relationships

- Each Agent Profile is guided by multiple Requirements and has one distinct primary responsibility.
- A Verification Finding links one task comparison to one or more Requirements and supporting evidence.
- A Test Result belongs to a specific acceptance check and records its observed evidence independently from a Verification Finding.
- A Coordination Assignment is owned by a profile and may depend on other assignments.
- An SDD Workflow Checkpoint gates the next phase; approval is conversational and not a deterministic runtime permission lock.
- A Learning Checkpoint belongs to a mentoring conversation and does not imply a persistent student record.

## State Transitions

No persisted runtime entity lifecycle is introduced. Verification statuses are per-review outcomes, not stored application state.
