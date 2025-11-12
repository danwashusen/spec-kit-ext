# Configuration

Spec Kit supports external configuration that all slash commands read at startup to locate your constitution and any supplemental documents. Configuration is optional — projects work out of the box with a minimal default.

## Lookup Order

When a command runs, it loads configuration in this order:

- `.specify.yaml` at the repository root (if present)
- `config-default.yaml` at the repository root (fallback)

If neither file is present, commands rely on hardcoded defaults in templates where applicable.

## Files

- `.specify.yaml` — Your project‑specific configuration. Commit this to your repo if you want to customize behavior.
- `config-default.yaml` — Ships with Spec Kit. Contains:

  - A minimal active default (sets only the constitution path) to keep projects zero‑config.
  - A commented, full example demonstrating how to reference architecture docs, front‑end specs, and other documents. This example is an enhancement/extension and is not required.

## Schema Overview

Configuration is defined under the top‑level `spec-kit` key. The canonical structure is a YAML sequence where each item is a single section object:

```yaml
spec-kit:
  - constitution:
      path: "CONSTITUTION.md"
      documents:
        - path: "docs/architecture.md"
          context: "Documents the architecture of the project and should be considered a primary source of truth."
        - path: "docs/ui-architecture.md"
          context: "Documents the UI architecture of the project and should be considered a primary source of truth."
  - specify:
      documents:
        - path: "docs/prd.md"
          context: "Documents the product requirements and should be considered a primary source of truth."
        - path: "docs/front-end-spec.md"
          context: "Documents the front-end specifications and should be considered a primary source of truth."
  - plan:
      documents:
        - path: "docs/architecture.md"
          context: "Documents the architecture of the project and should be considered a primary source of truth."
        - path: "docs/ui-architecture.md"
          context: "Documents the UI architecture of the project and should be considered a primary source of truth."
        - path: "docs/front-end-spec.md"
          context: "Documents the front-end specifications and should be considered a primary source of truth."
  - tasks:
      documents:
        - path: "docs/architecture.md"
          context: "Documents the architecture of the project and should be considered a primary source of truth."
        - path: "docs/ui-architecture.md"
          context: "Documents the UI architecture of the project and should be considered a primary source of truth."
        - path: "docs/front-end-spec.md"
          context: "Documents the front-end specifications and should be considered a primary source of truth."
  - analyze:
      documents:
        - path: "docs/architecture.md"
          context: "Documents the architecture of the project and should be considered a primary source of truth."
        - path: "docs/ui-architecture.md"
          context: "Documents the UI architecture of the project and should be considered a primary source of truth."
        - path: "docs/front-end-spec.md"
          context: "Documents the front-end specifications and should be considered a primary source of truth."
  - implement:
      documents:
        - path: "docs/work-flow.md"
          context: "Documents the implementation work-flow for the project and should be considered a primary source of truth."
  - audit:
      documents:
        - path: "docs/architecture.md"
          context: "Documents the architecture of the project and should be considered a primary source of truth."
        - path: "docs/ui-architecture.md"
          context: "Documents the UI architecture of the project and should be considered a primary source of truth."
        - path: "docs/front-end-spec.md"
          context: "Documents the front-end specifications and should be considered a primary source of truth."
```

Notes:

- Paths are resolved relative to the repository root.
- Missing optional files are noted and skipped by the agent.
- The commented example above mirrors the structure demonstrated in `config-default.yaml` and is meant as an enhancement; you do not need to include any of it to use Spec Kit.

## Keys and Their Roles

All configuration lives under the `spec-kit` sequence. Each item scopes settings for a specific command family, and every `documents[]` list follows the same shape (`path` + `context`). The CLI currently reads the following keys:

- `constitution.path`
  - Absolute or repo-relative path to the constitution Markdown file.
  - Default (from `config-default.yaml`): `/memory/constitution.md`.

- `constitution.documents[]`
  - Supplemental references for the `/constitution` workflow (e.g., operating models, governance handbooks).
  - Each entry defines where to find the document and how it should influence amendments.

- `specify.documents[]`
  - Background material the `/specify` command should study before writing or updating `spec.md` (PRDs, UX research, story maps).

- `plan.documents[]`
  - Design references consulted by both `/plan` and `/clarify` (architecture diagrams, interface contracts, target platform notes).

- `tasks.documents[]`
  - Authoritative sources the `/tasks` command reads while turning the plan into an executable task list (data models, API specs, process guides).

- `analyze.documents[]`
  - Optional context that enriches the cross-artifact review performed by `/analyze` (domain glossaries, compliance standards).

- `implement.documents[]`
  - Operational guardrails consumed during `/implement` (runbooks, deployment checklists, SLO dashboards).

- `review.documents[]`
  - Governance and policy material loaded by the `/audit` review playbook (quality gates, security standards, regulatory checklists).

Missing optional files are reported but do not stop execution.

## Command Behavior and Configuration

Each slash command begins by loading `.specify.yaml` (or `config-default.yaml` as a fallback), extracts the `spec-kit` section into `SPEC_KIT_CONFIG`, and then reads only the keys it needs:

- `/constitution`: uses `constitution.path` and `constitution.documents[]` when updating the governing document.
- `/clarify`: draws on `plan.documents[]` for additional context and relies on `constitution.path` to keep clarifications compliant.
- `/specify`: reads `specify.documents[]` to ground feature specifications; it does not touch the constitution directly.
- `/plan`: consumes `plan.documents[]` alongside the constitution at `constitution.path` while producing the implementation plan.
- `/tasks`: reads `tasks.documents[]` and the constitution to keep the task list aligned with established constraints.
- `/analyze`: loads `analyze.documents[]` plus the constitution to detect gaps across `spec.md`, `plan.md`, and `tasks.md`.
- `/implement`: references `implement.documents[]` together with the constitution to guide execution of the task plan.
- `/audit`: uses `review.documents[]` (and the constitution) to enforce governance and quality gates during the review playbook.
