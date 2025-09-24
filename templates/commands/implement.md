---
description: Execute the implementation plan by processing and executing all tasks defined in tasks.md
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

The user input can be provided directly by the agent or as a command argument - you **MUST** consider it before proceeding with the prompt (if not empty).

User input:

$ARGUMENTS

1. Load Spec Kit configuration:
   - Check for `/.specify.yaml` at the host project root; if it exists, load that file
   - Otherwise load `/config-default.yaml`
   - Extract the root `spec-kit` entry and store it as `SPEC_KIT_CONFIG`
   - Output the resulting `SPEC_KIT_CONFIG` for operator visibility

2. Run `{SCRIPT}` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All future file paths must be absolute.

3. If defined, read documents from `SPEC_KIT_CONFIG.implement.documents`:
   - For each item, resolve `path` to an absolute path from the repo root
   - Read the file and consider its `context` to guide implementation decisions
   - If a file is missing, note it and continue

4. Read the constitution at the path specified by
   `SPEC_KIT_CONFIG.constitution.path` to understand constitutional
   requirements.

5. Load and analyze the implementation context:
   - **REQUIRED**: Read tasks.md for the complete task list and execution plan
   - **REQUIRED**: Read plan.md for tech stack, architecture, and file structure
   - **IF EXISTS**: Read data-model.md for entities and relationships
   - **IF EXISTS**: Read contracts/ for API specifications and test requirements
   - **IF EXISTS**: Read research.md for technical decisions and constraints
   - **IF EXISTS**: Read quickstart.md for integration scenarios

6. Parse tasks.md structure and extract:
   - **Task phases**: Setup, Tests, Core, Integration, Polish
   - **Task dependencies**: Sequential vs parallel execution rules
   - **Task details**: ID, description, file paths, parallel markers [P]
   - **Execution flow**: Order and dependency requirements

7. Execute implementation following the task plan:
   - **Phase-by-phase execution**: Complete each phase before moving to the next
   - **Respect dependencies**: Run sequential tasks in order, parallel tasks [P] can run together
   - **Deliver full task scope**: Implement each task’s complete, spec-compliant outcome in the current run; do not defer required capabilities (e.g., real persistence) into unstated future work, no hidden follow-up work!
   - **Sandbox restrictions**: If the security sandbox blocks a required action (e.g., dependency installation because of network limits or permission errors), stop immediately, explain the restriction to the user, and request guidance or approval instead of attempting unsanctioned workarounds.
   - **Assumption Handling (per-task decisions)**:
      - All output relating to Assumption Handling must be prefixed with "[ASSUMPTION]".
      - When evaluating implementation options for an individual task:
         - Every potential solution **MUST** confirm the requirements in spec.md remain fully achievable
         - Do **NOT** weigh time-versus-value trade-offs; operate as an AI with effectively unlimited capacity
      - If no clear task-level path exists:
         1. Explain the situation to the user.
         2. Instruct the user to "Please amend the `research.md` document then re-run the implementation playbook".
      - Log all decisions under an "Assumption Log" section in the relevant `tasks.md` file, include enough detail to allow future developers to understand the rationale.
   - **Plan Deviation (task scope changes)**:
      - All output relating to Plan Deviation must be prefixed with "[PLAN DEVIATION]".
      - Do not alter the defined tasks or their intent unless absolutely unavoidable.
      - Deferring scope-critical deliverables for a task (like persistence layers or integrations) counts as a deviation. If you cannot execute the task as written without such a change, explain the situation to the user and HALT immediately, seek explicit guidance from the user, and record the clarification in research.md.
   - **Follow TDD approach**: Execute test tasks before their corresponding implementation tasks
   - **File-based coordination**: Tasks affecting the same files must run sequentially
   - **Code Quality Gates**: You **MUST** ensure code quality gates are satisfied quickly per task and extensively per phase
   - **Validation checkpoints**: Verify each phase completion before proceeding

8. Implementation execution rules:
   - **Setup first**: Initialize project structure, dependencies, configuration
   - **Tests before code**: If you need to write tests for contracts, entities, and integration scenarios
   - **Core development**: Implement models, services, CLI commands, endpoints
   - **Integration work**: Database connections, middleware, logging, external services
   - **Polish and validation**: Run code quality gates (e.g. lint and type checking, etc.), tests, performance optimization, documentation

9. Progress tracking and error handling:
   - Report progress after each completed task
   - Halt execution if any non-parallel task fails
   - For parallel tasks [P], continue with successful tasks, report failed ones
   - Provide clear error messages with context for debugging
   - Suggest next steps if implementation cannot proceed
   - **IMPORTANT** For completed tasks, make sure to mark the task off as completed ([✓]) in the tasks.md file.

10. Completion validation:
   - Verify all required tasks are completed
   - Check that implemented features match the original specification
   - Validate that tests pass and coverage meets requirements
   - Confirm the implementation follows the technical plan
   - Report final status with a summary of completed work

Note: This command assumes a complete task breakdown exists in tasks.md. If tasks are incomplete or missing, suggest running `/tasks` first to regenerate the task list.

Use absolute paths with the repository root for all file operations to avoid path issues.
