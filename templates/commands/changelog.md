---
description: |
  Prepare a detailed changelog entry for the current feature branch and suggest edits to documentation
version: 2.0
semantics:
  notes:
    - This playbook is stack-agnostic and applies to any language or runtime.
    - Always use absolute paths from the repository root for all file operations.
    - Prefer idempotent, minimal modifications; only mutate the designated changelog file when persisting results.
---

# Change Log Playbook

## Overview

Use this playbook to produce a feature-branch changelog from start to finish. Work through the numbered steps in order,
completing every subtask listed under a step before moving on. Produce the evidence or decisions each bullet calls for,
and keep progressing autonomously unless a gate forces you to pause or raise an open question. Treat every step as a
mandatory checkpoint; log and surface any blocking issues before continuing downstream.

## Flow

1. Initialize Spec Kit configuration

    - Check for `/.specify.yaml` at the project root; if present, load it
    - Otherwise load `/config-default.yaml`
    - Extract the root `spec-kit` entry and store it as `SPEC_KIT_CONFIG`
    - Output the resulting `SPEC_KIT_CONFIG` for operator visibility

2. Capture prerequisite signals

    - Run `/scripts/bash/check-prerequisites.sh --json` from the repo root
    - Parse `FEATURE_DIR` and the `AVAILABLE_DOCS` list; all future file paths must be absolute
    - Record sandbox mode, network access, and approval policy before running commands

3. Load and analyze the implementation context

    - **REQUIRED**: Read `tasks.md` for the complete task list and execution plan
    - **REQUIRED**: Read `plan.md` for tech stack, architecture, and file structure
    - **REQUIRED**: Read `spec.md` for the specification
    - **IF EXISTS**: Read `data-model.md` for entities and relationships
    - **IF EXISTS**: Read `contracts/` for API specifications and test requirements
    - **IF EXISTS**: Read `research.md` for technical decisions and constraints
    - **IF EXISTS**: Read `quickstart.md` for integration scenarios

4. Understand the system context

    - If defined, read documents from `SPEC_KIT_CONFIG.changelog.documents`, refer to them as the document context:
        - For each entry, resolve `path` to an absolute path from the repo root
        - Read the file and apply its `context` to understand the intended audience and scope
        - If a document is missing, record it as a risk before continuing
        - Consider the file to be read-only, **do NOT modify the file unless instructed to do so**
    - Read the constitution at `SPEC_KIT_CONFIG.constitution.path` to understand constitutional requirements

5. Consolidate feature inputs

    - Treat `FEATURE_DIR` (e.g., `specs/006-story-2-2/`) as the authoritative dossier; read `plan.md`, `tasks.md`,
      `research.md`, and related outputs to gather every directive that states scope, explicit targets, exclusions, focus
      questions, risks, or acceptance gates
    - Assume implementation occurs on a branch named after the feature directory slug (e.g., branch `006-story-2-2`);
      compare that branch tip to its baseline (`main` unless overridden) to identify the change set under review, and note
      any caller-provided diff hints or commit ranges
    - Derive concrete scope hints by enumerating file paths, directories, and components referenced within `tasks.md` (task
      descriptions, `[P]` tags, dependency notes) and `plan.md` (project structure decisions); store the consolidated list
      as `REVIEW_SCOPE_HINTS`
    - Capture explicit guardrails, risk registers, checklist items, or out-of-scope statements and store them as
      `REVIEW_DIRECTIVES`
    - Parse the `Requirements` section of `spec.md` to build `SPEC_REQUIREMENTS`; for each requirement capture `id`
      (e.g., `FR-###`), verbatim rule text, related acceptance scenarios/tests, and any embedded clarification markers
    - Extract the `Clarifications` section from `spec.md` (if present), normalize each Q/A exchange into `SPEC_CLARIFICATIONS`,
      and retain session markers or dates so changelog readers can trace the dialogue provenance
    - Start a `CONTROL_INVENTORY` by mapping each cross-cutting control mandated in the gathered documents (e.g., logging,
      authentication, telemetry, storage, observability) to the code assets or services that already implement it; record
      the path locator, purpose, and reuse guidance in a structured format to inform downstream summaries
    - Extract the `Assumption Log` section from `tasks.md` (if present), normalize each entry into `ASSUMPTION_LOG`, and capture
      enough context for changelog readers to understand the assumption, its rationale, and any knock-on actions

6. Resolve scope and operating context

    - Normalize every hint in `REVIEW_SCOPE_HINTS` into repository-root globs or explicit absolute paths
    - Union the hints and store the unique result as `RESOLVED_SCOPE`
    - Verify each resolved path exists; if any are missing, halt and flag the gap for operator review
    - Derive the comparison baseline:
        - Default to `main`
        - If `SPEC_KIT_CONFIG.changelog.baseline` is defined, prefer that branch or commit
        - Accept explicit overrides supplied via CLI arguments or environment variables
    - Confirm the working tree is clean before analysis; if not, snapshot the diff and report the dirty files without attempting remediation
    - When sandboxing or policy blocks a required command, request elevation or document the limitation in the final output

7. Gather branch metadata

    - Capture the current branch name (`git rev-parse --abbrev-ref HEAD`) and confirm it aligns with the feature slug
    - Determine the merge base with the baseline (`git merge-base <current> <baseline>`)
    - Collect commit metadata between merge base and branch tip:
        - Author, timestamp, commit message, and body
        - Conventional commit prefixes or ticket references
        - Any `BREAKING CHANGE` annotations or release-significant notes
    - Store these details as `BRANCH_HISTORY`, ordered chronologically, and surface notable themes (feature, fix, refactor)

8. Build change inventory

    - Produce a diff summary (`git diff --stat <baseline>...HEAD`) and capture totals for files changed, insertions, and deletions
    - Enumerate changed files (`git diff --name-status`) and classify each entry:
        - File status (A, M, D, R, C)
        - File category inferred from path (`src`, `tests`, `docs`, `infra`, `scripts`, `config`, etc.)
        - Whether it falls inside `RESOLVED_SCOPE` or is an out-of-scope surprise
    - For each file, gather contextual signals:
        - Detect structural changes (migrations, API signatures, schema evolution) by inspecting headers or prominent code blocks
        - Note large deletions or additions that may signal breaking changes
    - Store the results as `CHANGE_INVENTORY` with enough metadata to support aggregation and reporting

9. Map changes to requirements

    - For every requirement in `SPEC_REQUIREMENTS`, search `CHANGE_INVENTORY` for evidence of fulfillment:
        - Link files, functions, or modules that implement or modify the requirement
        - Quote or summarize line-level changes that satisfy acceptance criteria
    - Record unmet requirements or partial coverage as `REQUIREMENT_GAPS`
    - Highlight new functionality that lacks a traceable requirement and flag it as potential scope creep
    - Update `CONTROL_INVENTORY` with references to code that implements mandated controls

10. Evaluate tests and quality signals

    - Identify modifications to test files or suites and classify by type (unit, integration, acceptance, lint)
    - Check for new or updated fixtures, mocks, or data seeds and ensure they align with requirements
    - Note any absence of tests for newly introduced behaviors and add the findings to `TEST_GAPS`
    - Gather CI/workflow adjustments and infer whether additional validation steps are expected post-merge

11. Assess documentation drift

    - Detect changes to docs, READMEs, help text, or configuration manuals and summarize what shifted
    - Identify areas where documentation should be added or revised but is currently missing; add these to `DOC_ACTIONS`
    - Capture the intended audience for each needed update and propose concrete edit suggestions or locations
    - Record whether the drift blocks release, requires follow-up before merge, or is purely informational

12. Surface risks, mitigations, and follow-ups

    - From `tasks.md`, `research.md`, and the diff review, compile outstanding risks or open questions
    - Capture backward-compatibility concerns, migration steps, or data considerations
    - Note if rollback guidance is absent so the operator can request it separately
    - Aggregate any approvals or sign-offs required before release

13. Draft the changelog entry

    - Load the changelog template at `/templates/changelog-template.md` (treating it as read-only) to keep formatting consistent across runs
    - Populate the template with gathered data:
        - Replace placeholders such as `{{branch_slug}}`, `{{overview}}`, `{{highlights}}`, and `{{requirement_rows}}`
        - Expand list placeholders into multi-line bullet lists or tables as appropriate
        - Ensure every placeholder is fully replaced before presenting the entry to the operator
    - Compose content for the following sections:
        - **Overview**: two to three sentences describing the feature and its impact
        - **Highlights**: bullet list of key changes, referencing absolute paths where helpful
        - **Requirement Coverage**: table mapping requirement IDs to change evidence and status
        - **Testing**: summary of new or updated tests and remaining gaps
        - **Risks & Mitigations**: condensed view of identified risks and mitigation notes
        - **Clarifications**: surface relevant Q&A from `spec.md`, expanding any shorthand so the context is self-contained
        - **Assumption Log**: mirror `tasks.md` assumptions, enriching each with any clarifying context gathered during review
    - Match the tone to project conventions (past tense, imperative, or release-note style) while staying concise

14. Persist the changelog entry

    - Determine the target changelog path:
        - Use `SPEC_KIT_CONFIG.changelog.target` when provided (absolute path from the repo root)
        - Otherwise default to `CHANGELOG.md`
    - Ensure the parent directory exists; create it before writing if necessary
    - If the changelog file is missing, initialize it with a `# Change Log` heading followed by a blank line
    - When rerunning the playbook for the same feature branch:
        - Locate any section whose heading matches `## {branch_slug}` (using the resolved branch slug)
        - Remove the entire section up to but excluding the next `##` heading or end-of-file
    - Insert the newly generated entry immediately after the top-level heading unless `SPEC_KIT_CONFIG` specifies a different insertion point
    - Save the file, ensuring trailing newlines and surrounding spacing remain tidy to minimize merge noise

15. Emit structured output

    - Present key artifacts (`BRANCH_HISTORY`, `CHANGE_INVENTORY`, `SPEC_REQUIREMENTS`, `REQUIREMENT_GAPS`, `TEST_GAPS`, `DOC_ACTIONS`, `CONTROL_INVENTORY`, `SPEC_CLARIFICATIONS`, and `ASSUMPTION_LOG`) as concise, human-readable tables or lists so the operator can act without parsing raw JSON
    - Emit the finalized Markdown changelog entry separately so it can be copied into release notes or documentation
    - Provide a documentation drift spotlight that summarizes key gaps and suggested edits so the operator can act immediately
    - Restate any assumptions or clarifications that require operator validation or documentation updates so they do not get lost between the source documents and the changelog entry
    - If any blocking issues were found (missing files, unresolved scope), end with a clear escalation message summarizing the gap
