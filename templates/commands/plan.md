---
description: Execute the implementation planning workflow using the plan template to generate design artifacts.
scripts:
  sh: scripts/bash/setup-plan.sh --json
  ps: scripts/powershell/setup-plan.ps1 -Json
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. **Setup**:
   - Load Spec Kit configuration (`/.specify.yaml` if present, otherwise `/config-default.yaml`) and extract the root `spec-kit` entry as `SPEC_KIT_CONFIG`.
   - Run `{SCRIPT}` from repo root and parse JSON for FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Load context**:
   - Re-read FEATURE_SPEC in detail (requirements, success criteria, constraints).
   - Read the constitution from `SPEC_KIT_CONFIG.constitution.path` and enforce every MUST principle.
   - If defined, load each `SPEC_KIT_CONFIG.plan.documents[*]` file (resolve relative to repo root) and treat its `context` as planning guidance. Note missing files without failing.
   - Read the changelog at `SPEC_KIT_CONFIG.changelog.path` (if present) to capture institutional knowledge.
   - Load the IMPL_PLAN template (already copied) to prepare for execution.
   - Treat all referenced documents as read-only unless explicitly instructed otherwise.

3. **Execute plan workflow**: Follow the structure in IMPL_PLAN template to:
   - Fill Technical Context (mark unknowns as "NEEDS CLARIFICATION")
   - Incorporate additional constraints or preferences provided in `{ARGS}` into Technical Context and planning decisions
   - Fill Constitution Check section from constitution
   - Evaluate gates (ERROR if violations unjustified)
   - Phase 0: Generate research.md (resolve all NEEDS CLARIFICATION) and catalog codebase reconnaissance findings
   - Phase 1: Generate data-model.md, contracts/, quickstart.md linked to reconnaissance insights
   - Phase 1: Update agent context by running the agent script
   - Re-evaluate Constitution Check post-design

4. **Stop and report**: Command ends after Phase 2 planning. Report branch, IMPL_PLAN path, and generated artifacts.

## Phases

### Phase 0: Outline & Research

1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Generate and dispatch research agents**:

   ```text
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

4. **Perform codebase reconnaissance**:
   - Run a repository-wide inventory (e.g., `rg --files`, targeted `ls`, dependency manifest inspection) to surface all modules, helpers, configuration knobs, scripts, migrations, fixtures, and tests that influence the feature scope.
   - For each finding record: Story/Decision ID (US*/D*), absolute path from repo root, current responsibility, dependent helpers/config toggles, invariants or edge cases to respect, and recommended verification hooks.
   - Add a `## Codebase Reconnaissance` section to `research.md` with subsections per user story (e.g., `### US1 – <summary>`) and tables keyed by Decision IDs so `/speckit.implement` can jump directly to affected files.
   - Flag any missing intelligence with `TODO` rows and note follow-up owners or blockers that must be resolved before implementation begins.

**Output**: research.md with all NEEDS CLARIFICATION resolved plus Codebase Reconnaissance inventory

### Phase 1: Design & Contracts

**Prerequisites:** `research.md` complete

1. **Extract entities from feature spec** → `data-model.md`:
   - Entity name, fields, relationships
   - Validation rules from requirements
   - State transitions if applicable

2. **Generate API contracts** from functional requirements:
   - For each user action → endpoint
   - Use standard REST/GraphQL patterns
   - Output OpenAPI/GraphQL schema to `/contracts/`

3. **Agent context update**:
   - Run `{AGENT_SCRIPT}`
   - These scripts detect which AI agent is in use
   - Update the appropriate agent-specific context file
   - Add only new technology from current plan
   - Preserve manual additions between markers

4. **Author quickstart scenarios tied to reconnaissance**:
   - For each user story/decision, craft validation walkthroughs that cite the corresponding reconnaissance rows and list exact commands, data fixtures, configuration toggles, and rollback steps.
   - Highlight environment prerequisites (secrets, services, build targets) so implementers can exercise the touched modules without additional discovery.

**Output**: data-model.md, /contracts/*, quickstart.md (annotated with Story/Decision IDs), agent-specific file

## Key rules

- Use absolute paths and reference files relative to repository root (e.g., `/specs/...`)
- ERROR on gate failures or unresolved clarifications
