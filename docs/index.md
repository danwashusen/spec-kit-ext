# Spec Kit

*Build high-quality software faster.*

**An effort to allow organizations to focus on product scenarios rather than writing undifferentiated code with the help of Spec-Driven Development.**

## What is Spec-Driven Development?

Spec-Driven Development **flips the script** on traditional software development. For decades, code has been king — specifications were just scaffolding we built and discarded once the "real work" of coding began. Spec-Driven Development changes this: **specifications become executable**, directly generating working implementations rather than just guiding them.

## Getting Started

- [Installation Guide](installation.md)
- [Quick Start Guide](quickstart.md)
- [Local Development](local-development.md)

## Configuration

Commands read configuration from `.specify.yaml` at your project root when present, or fallback to `config-default.yaml` otherwise. The default file includes:
- A minimal active config (constitution path) to keep things zero‑config.
- A commented full example showing how to reference architecture docs, front‑end specs, and other materials to enhance the workflow. This example is optional and not required.

See `config-default.yaml` in the repository root and [Configuration](configuration.md) for a full example and details.

## Commands

Every slash command loads configuration (preferring `.specify.yaml`, falling back to `config-default.yaml`), stores it as `SPEC_KIT_CONFIG`, and runs a purpose-built script under `scripts/` to discover feature paths. The sections below summarise what each command actually does based on the command templates in `templates/commands/`.

### `/constitution`

- **Purpose**: Create or amend the governing constitution referenced by the rest of the workflow.
- **Configuration**: Reads `SPEC_KIT_CONFIG.constitution.path` and optional `SPEC_KIT_CONFIG.constitution.documents` as supporting material.
- **Execution highlights** (`templates/commands/constitution.md`):
  - Loads the existing constitution (if present) plus the canonical template in `/memory/constitution.md`.
  - Resolves every placeholder token, decides the correct semantic version bump, and enforces ISO dates.
  - Reviews core templates (`plan-template.md`, `spec-template.md`, `tasks-template.md`) and command files to keep their guidance aligned with the updated principles.
  - Writes a “Sync Impact Report” HTML comment to the top of the constitution summarising version changes, impacted artefacts, and TODOs.
- **Outputs**: Updated constitution file at `constitution.path` with all placeholders replaced and dependent-doc alignment checked.

### `/specify`

- **Purpose**: Bootstrap or revise the feature specification (`spec.md`) for the active feature branch.
- **Configuration**: Consumes `SPEC_KIT_CONFIG.specify.documents`; also reuses the constitution path for downstream checks.
- **Execution highlights** (`templates/commands/specify.md`):
  - Runs `scripts/bash/create-new-feature.sh --json` (or PowerShell equivalent) exactly once to create/locate the feature branch and spec file.
  - Reads any configured supporting documents before drafting.
  - Fills `/templates/spec-template.md` with concrete content, ensuring required sections and acceptance checks are populated.
- **Outputs**: Updated `specs/<feature>/spec.md`, branch metadata, and a readiness report for the next phase.

### `/clarify`

- **Purpose**: Systematically remove ambiguity from `spec.md` before planning.
- **Configuration**: Uses `SPEC_KIT_CONFIG.plan.documents` for context and the constitution for compliance.
- **Execution highlights** (`templates/commands/clarify.md`):
  - Runs `check-prerequisites` to discover `FEATURE_DIR` and required files.
  - Audits the spec with a taxonomy covering functional scope, data model, UX, NFRs, integrations, constraints, and more.
  - Asks up to five tightly scoped questions (multiple-choice or short reply), logging each answer under `## Clarifications` → `### Session <date>`.
  - Applies each answer directly to the relevant section (functional requirements, user stories, NFRs, etc.) and validates that ambiguity is resolved.
- **Outputs**: Revised `spec.md` with a structured clarifications log and a coverage summary describing remaining gaps or deferrals.

### `/plan`

- **Purpose**: Produce the detailed implementation plan and design artefacts used by later phases.
- **Configuration**: Reads `SPEC_KIT_CONFIG.plan.documents`, `plan.technical_context`, `plan.additional_research`, plus the constitution.
- **Execution highlights** (`templates/commands/plan.md`):
  - Requires an existing clarifications section or an explicit user override before proceeding.
  - Runs `setup-plan.sh --json` to gather file paths and copy the plan template into place.
  - Reads the feature spec, configured documents, and constitution; populates `plan.md` via `/templates/plan-template.md`.
  - Executes Phase 0 and Phase 1 of the template, generating `research.md`, `data-model.md`, `contracts/`, `quickstart.md`, and any agent-specific files.
  - Re-validates constitutional gates after design work and stops before task generation (Phase 2 is deferred to `/tasks`).
- **Outputs**: `plan.md` populated with technical context, progress tracking, and Constitution Check results, plus supporting design artefacts inside `specs/<feature>/`.

### `/tasks`

- **Purpose**: Convert the plan into an executable task list and enrich research context for downstream automation.
- **Configuration**: Reads `SPEC_KIT_CONFIG.tasks.documents` and the constitution.
- **Execution highlights** (`templates/commands/tasks.md`):
  - Uses `check-prerequisites` to load all available design artefacts (plan, research, data model, contracts, quickstart).
  - Generates `tasks.md` from `/templates/tasks-template.md`, organising work into Setup, Tests, Core, Integration, and Polish phases with dependency notes and `[P]` markers for safe parallelism.
  - Ensures TDD ordering (tests before implementation) and maps tasks to actual file paths.
  - Appends a “System Context” and “Codebase Summary” section to `research.md`, capturing insights needed by AI agents during implementation.
- **Outputs**: Structured `tasks.md` with numbered tasks (T001, T002, …) and updated `research.md` containing system context and codebase overviews.

### `/analyze`

- **Purpose**: Perform a read-only consistency audit across `spec.md`, `plan.md`, and `tasks.md` before coding begins.
- **Configuration**: Uses `SPEC_KIT_CONFIG.analyze.documents` and the constitution for authoritative checks.
- **Execution highlights** (`templates/commands/analyze.md`):
  - Runs `check-prerequisites` with task validation to confirm the required artefacts exist.
  - Builds semantic inventories for requirements, user stories, tasks, and constitutional rules.
  - Detects duplication, ambiguity, underspecification, constitution violations, coverage gaps, and terminology drift.
  - Emits a markdown report featuring a findings table with severity levels, coverage summary, constitution issues, unmapped tasks, and key metrics, then prompts the operator about remediation suggestions.
- **Outputs**: A structured analysis report (no file writes) highlighting critical issues to resolve before `/implement`.

### `/implement`

- **Purpose**: Execute the task list while respecting dependencies, constitutional constraints, and assumption logging rules.
- **Configuration**: Reads `SPEC_KIT_CONFIG.implement.documents` and the constitution.
- **Execution highlights** (`templates/commands/implement.md`):
  - Loads all plan artefacts plus any configured operational guides.
  - Parses `tasks.md` into phases, enforcing sequential vs. parallel execution and TDD ordering.
  - At each decision point, records `[ASSUMPTION]` notes (logged back to `tasks.md`) and halts on unresolvable ambiguities or sandbox restrictions.
  - Flags scope changes with `[PLAN DEVIATION]`, requires explicit user approval to proceed, and marks completed tasks with `[✓]` directly in `tasks.md`.
- **Outputs**: Code changes aligned with the plan, updated `tasks.md` reflecting completion status and assumption logs, and console progress reports per task.

### `/audit`

- **Purpose**: Run the governance-heavy review playbook after implementation, ensuring policy gates and quality controls are satisfied.
- **Configuration**: Consumes `SPEC_KIT_CONFIG.review.documents` alongside the constitution.
- **Execution highlights** (`templates/commands/audit.md`):
  - Executes `check-implementation-prerequisites.sh --json` to gather the feature directory and document inventory.
  - Loads all implementation artefacts (spec, plan, tasks, research, data model, contracts, quickstart) plus review-specific documents and platform controls.
  - Applies multi-pass review gates (scope reconciliation, quality controls, security/privacy, supply chain), running required quality commands where mandated.
  - Produces a detailed review report following `/templates/review-template.md`, archives it to `specs/<feature>/review.md`, and injects follow-up tasks into `tasks.md` under **Phase 4.R: Review Follow-Up**.
- **Outputs**: Governance-aligned `review.md`, augmented `tasks.md` with remediation tasks, and an audit trail of findings and strengths ready for release decisions.

## Contributing

Please see our [Contributing Guide](CONTRIBUTING.md) for information on how to contribute to this project.

## Support

For support, please check our [Support Guide](SUPPORT.md) or open an issue on GitHub.
