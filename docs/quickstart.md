# Quick Start Guide

This guide will help you get started with Spec-Driven Development using Spec Kit Ext on an existing project.

> NEW: All automation scripts now provide both Bash (`.sh`) and PowerShell (`.ps1`) variants. The `specify-ext` CLI auto-selects based on OS unless you pass `--script sh|ps`.

## The Process

### 1. Install Specify

Initialize Spec Kit Ext in your project:

```bash
uvx --from git+https://github.com/danwashusen/spec-kit-ext.git specify-ext init --here
```

### 2. Configuration

This fork shines when you feed the CLI the same documents your team already uses. Create `.specify.yaml` at the project root to extend the context passed into each Spec Kit command. A richer example:

```yaml
spec-kit:
  - constitution:
      path: 'CONSTITUTION.md'
      documents:
        - path: 'docs/architecture.md'
          context: 'Documents the architecture of the project and should be considered a primary source of truth.'
        - path: 'docs/ui-architecture.md'
          context: 'Documents the UI architecture of the project and should be considered a primary source of truth.'
  - specify:
      documents:
        - path: 'docs/prd.md'
          context: 'Documents the product requirements and should be considered a primary source of truth.'
        - path: 'docs/front-end-spec.md'
          context: 'Documents the front-end specifications and should be considered a primary source of truth.'
  - plan:
      technical_context:
        - '**Coding Standards**: including Linting & Formatting, TypeScript Compiler Standards, Quality Gates, Quality Controls Governance, Critical Rules [NEEDS CLARIFICATION]'
        - '**Test Strategy and Standards**: [NEEDS CLARIFICATION]'
      documents:
        - path: 'docs/architecture.md'
          context: 'Documents the architecture of the project and should be considered a primary source of truth.'
        - path: 'docs/ui-architecture.md'
          context: 'Documents the UI architecture of the project and should be considered a primary source of truth.'
        - path: 'docs/front-end-spec.md'
          context: 'Documents the front-end specifications and should be considered a primary source of truth.'
  - tasks:
      documents:
        - path: 'docs/architecture.md'
          context: 'Documents the architecture of the project and should be considered a primary source of truth.'
        - path: 'docs/ui-architecture.md'
          context: 'Documents the UI architecture of the project and should be considered a primary source of truth.'
        - path: 'docs/front-end-spec.md'
          context: 'Documents the front-end specifications and should be considered a primary source of truth.'
  - analyze:
      documents:
        - path: 'docs/prd.md'
          context: 'Documents the product requirements and should be considered a primary source of truth, any deviations should be called out.'
        - path: 'docs/architecture.md'
          context: 'Documents the architecture of the project and should be considered a primary source of truth, any deviations should be called out.'
        - path: 'docs/ui-architecture.md'
          context: 'Documents the UI architecture of the project and should be considered a primary source of truth, any deviations should be called out.'
        - path: 'docs/front-end-spec.md'
          context: 'Documents the front-end specifications and should be considered a primary source of truth, any deviations should be called out.'
  - implement:
      documents:
        - path: 'docs/work-flow.md'
          context: 'Documents the implementation work-flow for the project and should be considered a primary source of truth.'
  - audit:
      documents:
        - path: 'docs/prd.md'
          context: 'Documents the product requirements and should be considered a primary source of truth, any deviations should be called out.'
        - path: 'docs/architecture.md'
          context: 'Documents the architecture of the project and should be considered a primary source of truth, any deviations should be called out.'
        - path: 'docs/ui-architecture.md'
          context: 'Documents the UI architecture of the project and should be considered a primary source of truth, any deviations should be called out.'
        - path: 'docs/front-end-spec.md'
          context: 'Documents the front-end specifications and should be considered a primary source of truth, any deviations should be called out.'
```

**Important**: This configuration assumes your (fictional) `docs/prd.md` contains Epics and Stories the agent can reference. Adjust paths and descriptions to mirror your real project. For a deeper dive into every key and field, see [docs/configuration.md](./configuration.md).

Notes:
- Lookup order: `.specify.yaml` → `config-default.yaml`
- Missing optional files are skipped automatically; the agent will highlight anything it expected but couldn’t read.

### 3. Create the Constitution

Use the `/constitution` command to define the constitution for the project you want to build.

### 4. Create the Spec

Use the `/specify` command to define the spec for the story you want to build. This command will create a new feature branch (e.g. `001-epic-1-story-1`) and directory in the `specs` (e.g. `specs/001-epic-1-story-1`) with a `spec.md` describing the feature:

```text
/specify Epic 1, Story 1
```

Verify the `spec.md` file matches your expectations (e.g. `specs/001-epic-1-story-1/spec.md`).

### 5. Clarify the Spec

Use the `/clarify` command to clarify the spec for the story you want to build, resolving any ambiguities. 

```text
/clarify ./specs/001-epic-1-story-1
```

Verify the `spec.md` file matches your expectations (e.g. `specs/001-epic-1-story-1/spec.md`).

### 6. Create a Technical Implementation Plan

Use the `/plan` command to create a technical plan and research documents for the feature derived from the configured documents.

```text
/plan ./specs/001-epic-1-story-1
```

Verify the `plan.md` file matches your expectations (e.g. `specs/001-epic-1-story-1/plan.md`), review the associated `research.md` document (e.g. `specs/001-epic-1-story-1/research.md`).

### 7. Break Down the plan into actionable tasks

Use `/tasks` to create an actionable task list, then ask your agent to implement the feature.

```text
/tasks ./specs/001-epic-1-story-1
```

Verify the `tasks.md` file matches your expectations (e.g. `specs/001-epic-1-story-1/tasks.md`).

### 8. Analyze the documents to ensure they align with the specification

Use `/analyze` to validate the specification against the documents, ensuring that the plan satisfies the specification requirements.

```text
/analyze ./specs/001-epic-1-story-1
```

This process is iterative and can be repeated until the specification is complete.

### 9. Implement the feature

Use `/implement` to implement the feature, to avoid issues introduced by context overflows avoid implementing too many tasks at once.

```text
/implement ./specs/001-epic-1-story-1, focus on Phase 3.1
```

```text
/implement ./specs/001-epic-1-story-1, focus on tasks T001-T003
```

Repeat until all tasks are marked as complete in the `tasks.md` file (e.g. `specs/001-epic-1-story-1/tasks.md`).

### 10. Audit the feature implementation

Use `/audit` to validate the feature implementation against the documents, ensuring that the implementation follows best-practise and meets the specification requirements.

```text
/implement ./specs/001-epic-1-story-1, focus on Phase 3.1
```

```text
/implement ./specs/001-epic-1-story-1, focus on tasks T001-T003
```

If the audit fails, you can use `/implement` to fix the issues (e.g. `/implement ./specs/001-epic-1-story-1, focus on Phase 4.R`) then re-run the audit.

Once the audit completes, you can merge the feature branch into the main branch.