---
description: |
  Run a scope-driven review that enforces project governance, quality gates, and
  risk controls
version: 2.0
semantics:
  levels: Levels use RFC 2119 / BCP 14 (MUST/SHOULD/MAY).
  authoritative_sources:
    - NIST SSDF v1.1 (SP 800-218) PW.7–PW.8
    - OWASP ASVS v4.x; OWASP Code Review Guide; OWASP SAMM
    - ISO/IEC 27002:2022 (8.28 Secure Coding); ISO/IEC 27001 ISMS
    - PCI DSS 4.0 (esp. 6.3.2 code review before release)
    - GDPR Article 25 (Data Protection by Design & Default), CCPA
    - SLSA v1.0 (supply chain provenance)
    - Google Engineering Practices (Small CLs, Speed, What to look for)
    - Microsoft Engineering Playbook (Code Reviews)
    - WCAG 2.2 (Level AA)
    - Google SRE (launch/readiness checklists)
  notes:
    - This playbook is stack-agnostic and applies to any language or runtime.
    - Always use absolute paths from the repository root for all file operations.
---

# Code Review Playbook

## Overview

Use this playbook to run a feature review from start to finish. Work through the numbered steps in order, completing every
subtask listed under a step before moving on. After capturing the required evidence or decisions, immediately advance to
the next instruction without waiting for an operator prompt, unless a gate explicitly requires consent or raises an open
question. Surface blockers as soon as they appear, but otherwise keep progressing autonomously. Treat every step as a
mandatory checkpoint; log and surface any blocking issues before continuing downstream.

## Flow

1. Initialize Spec Kit configuration

    - Check for `/.specify.yaml` at the project root; if present, load it
    - Otherwise load `/config-default.yaml`
    - Extract the root `spec-kit` entry and store it as `SPEC_KIT_CONFIG`
    - Output the resulting `SPEC_KIT_CONFIG` for operator visibility
    - **Record** review posture: platform (GitHub/GitLab/etc.), branch protection, required checks, CODEOWNERS availability

2. Capture prerequisite signals

    - Run `/scripts/bash/check-prerequisites.sh --json` from the repo root
    - Parse `FEATURE_DIR` and the `AVAILABLE_DOCS` list; all future file paths must be absolute
    - Record sandbox mode, network access, and approval policy before running commands
    - Scrape the active operator conversation for explicit feedback, clarifications, or directives;
      normalize the results into `SESSION_FEEDBACK` entries (author, timestamp, raw text, priority
      hints)

3. Load and analyze the implementation context

    - **REQUIRED**: Read `tasks.md` for the complete task list and execution plan
    - **REQUIRED**: Read `plan.md` for tech stack, architecture, and file structure
    - **REQUIRED**: Read `spec.md` within `FEATURE_DIR`; extract the `Requirements` section and flag a missing or malformed
      spec as `[Open Question]`
    - **IF EXISTS**: Read `data-model.md` for entities and relationships
    - **IF EXISTS**: Read `contracts/` for API specifications and test requirements
    - **IF EXISTS**: Read `research.md` for technical decisions and constraints
    - **IF EXISTS**: Read `quickstart.md` for integration scenarios
    - **Record** the plan-of-record (POR) change intent

4. Enrich governance context

    - If defined, read documents from `SPEC_KIT_CONFIG.audit.documents`, refer to them as the document context:
        - For each entry, resolve `path` to an absolute path from the repo root
        - Read the file and apply its `context` to guide review heuristics
        - If a document is missing, record it as a risk before continuing
        - Consider the file to be read-only, **do NOT modify the file unless instructed to do so**
    - Read the constitution at `SPEC_KIT_CONFIG.constitution.path` to understand constitutional requirements
    - Load platform enforcement posture (protected branches, required status checks, approval rules, CODEOWNERS)

5. Consolidate feature inputs

    - Treat `FEATURE_DIR` (e.g., `specs/006-story-2-2/`) as the authoritative dossier; read `plan.md`, `tasks.md`,
      `research.md`, and related outputs to gather every directive that states scope, explicit targets, exclusions, focus
      questions, risks, or acceptance gates; merge structured `SESSION_FEEDBACK` conversation items into
      `REVIEW_DIRECTIVES` tagged `source: conversation` so they are audited alongside dossier instructions
    - Assume implementation occurs on a branch named after the feature directory slug (e.g., branch `006-story-2-2`);
      gather candidate diff scopes and present them to the operator as a multi-choice prompt. Default to the full feature
      branch (`HEAD`) against `main`, but also surface:
        - `HEAD` against `SPEC_KIT_CONFIG.audit.baseline` if provided
        - Any CLI or environment overrides supplied at invocation time
        - A manual entry option for custom branch/commit ranges (e.g., `feature..HEAD`, `commitA...commitB`)
      Capture the operator's selection as `SELECTED_BASELINE` and store supplemental details (custom range notes,
      rationale) so downstream steps can reference the agreed comparison target
    - Derive concrete scope hints by enumerating file paths, directories, and components referenced within `tasks.md` (task
      descriptions, `[P]` tags, dependency notes) and `plan.md` (project structure decisions); store the consolidated list
      as `REVIEW_SCOPE_HINTS`
    - Capture explicit guardrails, risk registers, checklist items, or out-of-scope statements and store them as
      `REVIEW_DIRECTIVES`, ensuring every `SESSION_FEEDBACK` item is represented with its current disposition
    - Normalize every risk register entry, unresolved follow-up, conversation-sourced directive, and assumption from
      `tasks.md`, `research.md`, and other dossier sources; add them to `REVIEW_DIRECTIVES` with status markers
      (`mitigated`, `open`, `unknown`) so later steps must reconcile each one
    - Parse the `Requirements` section of `spec.md` to build `SPEC_REQUIREMENTS`; for each requirement capture `id`
      (e.g., `FR-###`), verbatim rule text, related acceptance scenarios/tests, and any embedded clarification markers
    - Start a `CONTROL_INVENTORY` by mapping each cross-cutting control mandated in the gathered documents (e.g., logging,
      authentication, telemetry, storage, observability) to the code assets or services that already implement it; record
      the path locator, purpose, and reuse guidance in a structured format that can be emitted with review outputs
    - Confirm the instruction/assumption handling strategy defined for the feature; if absent, default to raising
      `[Open Question]` items for ambiguous inputs

6. Resolve scope and operating context

    - Normalize the scope into concrete path globs or files using the repository root
    - Union multiple hints and record the result as `RESOLVED_SCOPE`
    - Resolve the comparison baseline using `SELECTED_BASELINE`:
        - Default to `HEAD` vs `main` when the operator accepts the default
        - When a different option is chosen (config override, CLI flag, or manual entry), record the explicit refs or
          commit range as `COMPARISON_TARGET`
        - Confirm the references exist locally; if missing, pause and request clarification before continuing
    - Verify the paths exist; if not, pause and request clarification
    - When required commands are blocked by sandboxing or policy, request elevation or document the limitation

7. Harvest governing requirements

    - **Seed `REVIEW_REQUIREMENTS`** with:
        - **Global Normative Requirements (baseline)** from this playbook (see **Requirements Catalog**)
        - Additional controlling documents from `SPEC_KIT_CONFIG`, `AVAILABLE_DOCS`, and operator-provided standards digests
        - Feature-level directives from `SPEC_REQUIREMENTS`, preserving their IDs and original wording
   - Read each document and **extract explicit mandates** (architecture boundaries, logging strategy, dependency policy,
     quality gates, quality controls governance, testing posture, risk controls), appending them to `REVIEW_REQUIREMENTS`
      with:
       - `id`, `name`, `level` (MUST/SHOULD/MAY), `rule` (one sentence),
         `rationale`, `source`
    - Keep `CONTROL_INVENTORY` in sync with these mandates by capturing canonical implementations, known extension points,
      and noted gaps that require follow-up; flag missing controls as provisional entries pending remediation
    - For every entry in `SPEC_REQUIREMENTS`, append a matching record to `REVIEW_REQUIREMENTS` with `source_requirement`
      set to the original `FR-###`; initialize each entry with `status="unverified"` until evidence or findings resolve it
    - Treat these requirements as authoritative; flag any observed deviation
    - When a mandate is ambiguous, raise `[Open Question]` entries, resolve them, and record outcomes in the decision log

8. Run pre-review gates

    - **Gate tiering**: Reviews run in iterative loops. Evaluate the gates below in sequence so foundational blockers land
      before higher-order controls.
    - **Review Gates (run in order)**:
        - **Context Gate**: required dossier files exist; else → `Blocked: Missing Context`
        - **Change Intent Gate**: change aligns with POR; else → `Blocked: Scope Mismatch`
        - **Unknowns Gate**: unresolved `[NEEDS CLARIFICATION: …]` → `Needs Clarification`
        - **TDD Evidence Gate**: tests drive behavior or equivalent rationale; absence → `TDD Violation`
        - **Environment Gate**: required tooling available; blocked tooling must be resolved before proceeding
        - **Separation of Duties Gate**: author cannot self-approve; emergency bypasses are exceptional and auditable
        - **Code Owners Gate**: changes under owned paths require owner approval (if CODEOWNERS configured)
        - **Quality Controls Gate**: linting/typing/formatting/CI guardrails untouched unless explicitly in scope; unexpected changes or failing/missing evidence → `Quality Controls Violation`
        - **Requirements Coverage Gate**: every `FR-###` in `SPEC_REQUIREMENTS` maps to implementation/test evidence, an
          explicit finding, or a documented clarification request; include previously logged risks and assumptions—if still
          unresolved, treat them as coverage gaps that must become findings or clarifications. Otherwise → `Changes Requested`
          (missing implementation), `Needs Clarification` (ambiguous requirement), or `Blocked: Scope Mismatch`
          (requirement intentionally deferred)
        - **Feedback Alignment Gate**: each `SESSION_FEEDBACK` entry maps to confirming evidence, a linked finding, or a
          documented clarification; unresolved items block approval (`Needs Clarification` or `Review Pending`) until coverage
          exists
        - **Security & Privacy Gate**: security/privacy checklist attached and applicable items addressed; else →
          `Security Gate Failure` or `Privacy Gate Failure`
        - **Supply Chain Gate**: dependency/SBOM/provenance checks enabled per policy (SLSA/CIS posture); else →
          `Supply Chain Violation`
    - If any gate fails, log those findings and set the current review status to the blocking gate (or `Review Pending`
      when multiple gates are open). Record in the decision log which remaining gates still require evaluation, then
      continue directly to Step 14 (Archival write-back and task generation) before pausing further analysis and
      instructing the operator to rerun the audit after remediation.
    - When later gates surface issues, capture targeted findings and, if they block approval, pause before entering
      additional specialized reviews (e.g., accessibility, observability) to keep context focused for the next fix→audit
      loop.

9. Collect objective evidence

    - Identify mandatory quality commands from `REVIEW_REQUIREMENTS` (e.g., lint, typecheck, test,
      build, contract checks); execute them repo-wide unless governance specifies a narrower scope
    - Capture pass/fail status and key diagnostics for each command; aggregate multiple errors per category into single
      findings
    - Establish baseline and current dependency snapshots using the project's package manager; compare for new
      vulnerabilities, deprecated packages, or policy breaches
    - Record runtime warnings or deprecation notices (non-blocking unless policy elevates)
    - Store raw outputs or pointers needed to substantiate findings without exposing secrets

10. Analyze implementation changes

    - Defer deeper analysis until the gating sequence clears; when foundational blockers exist, summarize their impact and log
      deferred review dimensions for the next audit iteration instead of generating exhaustive findings.
    - Review diffs/files for compliance with each item in `REVIEW_REQUIREMENTS`
    - For each `FR-###` requirement, trace implementation changes and associated tests; cite concrete files, commands, or
      scenarios that satisfy the requirement, and open findings when evidence is absent or contradictory
    - Revisit every entry in `REVIEW_DIRECTIVES` (including conversation-sourced `SESSION_FEEDBACK`), `ASSUMPTION_LOG`, and
      `CONTROL_INVENTORY`; confirm mitigation evidence or document why the directive no longer applies
    - When mitigation is missing or unclear, open a finding or `[Open Question]` citing the original directive and source
      document so closure cannot be skipped
    - Record resolved directives and assumptions in the decision log so later audit loops understand their disposition
    - Map changes to architecture layers or boundaries; flag violations or missing abstractions with proposed remediations
    - Verify cross-cutting controls (validation, logging, error handling, accessibility, performance, security,
      observability) adhere to the mandates
    - Prefer reuse: consult `CONTROL_INVENTORY` (and the broader codebase) before recommending new infrastructure
    - Evaluate test coverage for new behavior, error paths, and regressions; highlight gaps and recommend targeted tests
    - Inspect dependency additions/upgrades for justification, maintenance health, license alignment, and version currency
    - Note strengths that should be preserved (patterns, abstractions, tests, documentation)

11. Assemble review deliverables

    - If `{FEATURE_DIR}/audit.md` already exists, load it before drafting the new report and cache the current
      `### Active Findings (Current Iteration)` entries plus the full `### Historical Findings Log`. Preserve this data so
      the refreshed report can merge seamlessly with prior iterations.
    - On first-run dossiers (no existing `audit.md`), initialize an empty historical log per `/templates/audit-template.md`.
    - Populate `/templates/audit-template.md` exactly as structured; the template is the canonical output
      format, but inject the cached historical entries alongside the new iteration so the working draft already contains
      the combined `Historical Findings` content before write-back.
    - Record the **Final Status** using the defined taxonomy:
      `Approved`, `Changes Requested`, `Blocked: Missing Context`, `Blocked: Scope Mismatch`, `Needs Clarification`,
      `TDD Violation`, `Quality Controls Violation`, `Security Gate Failure`, `Privacy Gate Failure`,
      `Supply Chain Violation`, `Dependency Vulnerabilities`, `Deprecated Dependencies`, `Review Pending`.
    - When the audit halts during gate evaluation, mark the report as partial: set Final Status to the highest-severity
      blocking gate (or `Review Pending` when awaiting fixes) and add a `Partial coverage` note in Findings and the
      Decision Log that lists deferred gates (e.g., Security & Privacy Gate) so the next loop resumes there.
    - Complete the **Resolved Scope** section with branch, baseline, diff source, narrative, concrete paths, and
      implementation scope bullets grounded in the analyzed artifacts.
    - Populate **Outstanding Risks & Follow-Ups** with the current status of every directive and assumption, explicitly
      enumerating each `SESSION_FEEDBACK` item; any open feedback must appear under `### Active Findings` or
      `### Historical Findings Log` with a linked remediation or clarification request
    - In **Findings**, order items from highest to lowest severity and provide the full metadata set for each entry:
      `category`, `severity`, `confidence`, `impact`, `evidence`, `remediation`, `source_requirement`, `files`.
    - Render Findings using two subsections in the final report template:
        - `### Active Findings (Current Iteration)` lists only the findings created in the present audit cycle.
        - `### Historical Findings Log` persists all prior findings (resolved, accepted risk, or still open) in reverse
          chronological order so the dossier keeps institutional memory.
    - Historical entries must retain their original `F###` identifiers, title, severity at report time, the iteration
      timestamp (e.g., `Reported: 2025-10-03`), reviewer handle, and a `Status:` line such as `[Resolved 2025-10-05]`,
      `[Accepted Risk 2025-10-07]`, or `[Open – pending verification]`.
      When a prior `audit.md` exists, migrate any items previously listed under `Active Findings` into the historical log
      before overwriting content and append a brief resolution note referencing `tasks.md` (e.g., the completed Phase
      4.R task id or PR link) plus the evidence command that validated closure.
    - Document **Strengths** with supporting citations and list outstanding `[NEEDS CLARIFICATION: …]` items in the
      dedicated section.
    - Add a `### Feedback Traceability` subsection summarizing each `SESSION_FEEDBACK` entry, the evidence gathered, and any
      linked findings or remediation tasks so conversation directives stay auditable (keep schema synced with
      `/templates/audit-template.md`)
    - Publish the current `CONTROL_INVENTORY` in the provided table (or link to its structured artifact if stored
      elsewhere).
    - Summarize lint, typecheck, test, and build evidence under **Quality Signal Summary**, capturing status and key
      diagnostics for each command.
    - Capture **Dependency Audit Summary** details: baseline vs current severity counts, new CVEs, deprecated packages,
      justifications, and version currency observations.
    - Update the **Requirements Compliance Checklist** with pass/fail status and notes per requirement group.
    - Generate a **Requirements Coverage Table** sourced from `SPEC_REQUIREMENTS` with columns for requirement id, summary,
      implementation evidence, validating tests, and linked finding/clarification ids.
    - Append the **Decision Log** with resolved assumptions, constitutional interpretations, control reuse, and
      remediation-audit outcomes.
    - Ensure the **Remediation Logging** section is populated per Step 12 when high-severity findings or non-approved
      statuses are present.

12. Log remediation tasks

    - For every finding, outstanding risk, or assumption that remains open (severity `Critical`, `Major`, or status other
      than `Approved`), draft remediation tasks that capture the exact file path, root cause, impact, recommended fix
      steps, and references to the relevant entry in `CONTROL_INVENTORY` when one exists
    - Prefer remediations that align with existing controls or explicitly document when the inventory confirms a gap
    - Surface these remediation drafts in a dedicated **Remediation Logging** section using the template
      `Context → Control Reference → Actions → Verification`
    - Ensure all tasks cite exact file paths, root cause, impact, and steps; link the `CONTROL_INVENTORY` entry (or state
      no reusable control exists)
    - If write-back cannot be completed, preserve the wording in the review output, note the blocking condition, and
      provide clear manual application instructions
    - Maintain a non-destructive posture; remediation logging updates remain the single authorized exception

13. Audit remediation alignment

    - Compare newly appended remediation tasks against `CONTROL_INVENTORY` and the findings to verify each task references
      an appropriate existing control or documented gap
    - If any task lacks that linkage, add a reviewer warning and update the remediation text before finalizing the review
    - Preserve the audit outcome (pass/warn) in the decision log

14. **MANDATORY** Archival write-back and task generation

    - Reference `FEATURE_DIR` gathered in Step 2.
    - Review output filing: present the rendered, merged report to the operator, then create or amend `audit.md` inside
      `FEATURE_DIR` without discarding past findings history.
        - If a prior report was loaded, confirm the assembled markdown still contains `### Historical Findings Log`
          before write-back; treat a missing section as a blocking issue that must be resolved prior to saving.
        - Promote the cached `### Active Findings (Current Iteration)` entries from the prior run into the historical log,
          updating their `Status:` note using `tasks.md` Phase 4.R (checked tasks → `[Resolved {YYYY-MM-DD}]`, unchecked
          → `[Open – pending remediation]`, accepted-risk flag → `[Accepted Risk {YYYY-MM-DD}]`).
        - Prepend the new iteration (with its refreshed `### Active Findings` section) ahead of the retained history and
          write the merged markdown back to `FEATURE_DIR/audit.md`.
    - Open `tasks.md` in `FEATURE_DIR` and overwrite or create a **Phase 4.R: Review Follow-Up** section using
      the format defined in `/templates/tasks-template.md` with a new task per finding surfaced in the current iteration.
        - Each finding under `### Active Findings (Current Iteration)` must map to a new unchecked Phase 4.R task using
          the pattern `- [ ] F{FINDING_ID} Finding {FINDING_ID}: {FINDING_TITLE} as described in audit.md`, and must
          reference `audit.md` (or explicit evidence paths) for context.
        - When findings migrate into the historical log on later runs, update or check off the corresponding Phase 4.R
          tasks before creating new tasks for the latest findings to keep report, task list, and history synchronized.

    - Helper workflow for merging reports:

      ```text
      prior_report = read_if_exists(FEATURE_DIR/audit.md)
      prior_active, prior_history = extract_sections(prior_report)
      new_report = render_template(current_iteration, prior_history + normalize(prior_active))
      assert "### Historical Findings Log" in new_report when prior_report exists
      write(FEATURE_DIR/audit.md, new_report)
      sync_phase_4r_tasks(current_iteration.findings)
      ```

---

## Requirements Catalog (Global Normative Baseline)

> The following baseline requirements are **seeded into `REVIEW_REQUIREMENTS`** in Step 7. Add project-specific mandates
> and citations as needed.
 
### Governance & Process

- **GOV-01 Small, Focused Changes** — **SHOULD**: Prefer small, coherent CLs with clear descriptions and linked issues.
  *Rationale*: Improves review quality and speed. *Source*: Google Eng Practices.

### Security & Privacy

- **SEC-01 Secure Coding Baseline** — **MUST**: Adopt and apply secure coding standards/checklists. *Rationale*:
  Systematic security. *Source*: OWASP ASVS; ISO 27002 8.28.
- **SEC-02 Threat-Relevant Review** — **MUST**: Examine authN/Z, input validation, secrets, crypto, injection, SSRF,
  deserialization, data exposure proportional to change risk. *Rationale*: Vulnerability prevention. *Source*: ASVS;
  OWASP Code Review Guide.
- **SEC-03 Expert Review for High-Risk Changes** — **MUST**: Security specialist reviews
  auth/crypto/sandbox/privilege-sensitive changes. *Rationale*: Depth for critical areas. *Source*: NIST SSDF PW.7
  examples.
- **SEC-04 Static Analysis & Secrets Gate** — **MUST**: Required SAST/linters/secret scanning pass before merge;
  findings triaged in PR. *Rationale*: Automated defense + human judgment. *Source*: NIST SSDF PW.7–PW.8.
- **SEC-05 Dependency Risk & SBOM** — **MUST**: New/updated dependencies pass vulnerability, license, and SBOM checks
  per policy. *Rationale*: Supply-chain risk control. *Source*: SLSA; CIS.
- **SEC-06 PCI Scope** — **MUST**: For cardholder-data impact, perform pre-release code review focused on security
  defects. *Rationale*: Regulatory compliance. *Source*: PCI DSS 6.3.2.
- **SEC-07 Privacy by Design** — **MUST**: For personal-data changes, verify minimization, lawful basis, access
  controls, data-by-default, and logging scrubs. *Rationale*: Privacy compliance. *Source*: GDPR Art. 25; CCPA.

### Quality & Maintainability

- **QUAL-01 Correctness & Tests** — **MUST**: New behavior has tests; affected tests updated; all tests pass.
  *Rationale*: Regression prevention. *Source*: Google Eng Practices.
- **QUAL-02 Readability & Consistency** — **SHOULD**: Follow project style; clear naming; comments explain non-obvious
  decisions. *Rationale*: Evolvability. *Source*: Google Eng Practices.
- **QUAL-03 Complexity Boundaries** — **SHOULD**: Flag excessive function/module complexity; request refactors when
  comprehension suffers. *Rationale*: Reviewability and reliability. *Source*: Empirical review research.
- **QUAL-04 Architecture & Interfaces** — **SHOULD**: Check API boundaries, error handling, compatibility impacts.
  *Rationale*: System integrity. *Source*: Google/Microsoft guidance.
- **QUAL-05 Documentation Deltas** — **MUST**: Update docs/runbooks/API references when behavior or interfaces change.
  *Rationale*: Operational clarity. *Source*: Microsoft; SRE checklists.

### Reliability & Operations

- **REL-01 Production Readiness** — **MUST**: For availability/perf-affecting changes, confirm monitoring, alerts,
  capacity estimates, rollback, and staged rollout/canaries. *Rationale*: Reduce incident risk. *Source*: Google SRE.
- **REL-02 Defensive Coding** — **SHOULD**: Verify timeouts, retries, circuit breakers, idempotency, and failure modes.
  *Rationale*: Resilience. *Source*: SRE practices.

### Testing & CI

- **TEST-01 Required Checks** — **MUST**: CI required status checks (build/tests/linters/SAST) block merge on failure.
  *Rationale*: Quality gate. *Source*: Platform + SSDF.
- **TEST-02 Security Testing Linkage** — **SHOULD**: For security-sensitive areas, include dynamic/fuzz/IAST evidence
  when applicable. *Rationale*: Complement code review. *Source*: SSDF PW.8.

### Accessibility (UI Changes)

- **A11Y-01 WCAG 2.2 AA** — **MUST**: UI changes meet WCAG 2.2 AA (keyboard/focus/contrast/accessible auth).
  *Rationale*: Legal/compliance/usability. *Source*: W3C/WAI.
- **A11Y-02 ARIA Use** — **SHOULD**: Prefer native semantics; apply ARIA per APG when needed. *Rationale*: Avoid harming
  accessibility. *Source*: WAI-ARIA APG.

---

## Checklists (to attach/evaluate per change)

> Attach the relevant checklist(s) to the PR and link them in the deliverable.

### Security & Privacy Checklist (subset)

- Secrets absent / not in code; env/secret manager used
- Input validation & output encoding; injection risks mitigated
- AuthN/Z flows and permission checks intact; least privilege enforced
- Sensitive data minimized; logging scrubs PII; retention noted
- Dependency diff reviewed; vulnerabilities triaged; licenses aligned
- High-risk changes reviewed by security specialist
- SBOM/provenance evidence captured if required by policy (SLSA target)

### Reliability & Operations Checklist (subset)

- Monitoring/metrics/alerts updated; SLO/SLA considerations noted
- Rollback plan & feature-flag strategy documented
- Timeouts/retries/circuit breakers; idempotency for retried ops
- Runbooks updated; on-call impacted teams notified if relevant

### Accessibility (UI) Checklist (subset)

- Keyboard navigation (tab order, focus visible), skip links if applicable
- Semantic structure correct; ARIA only where necessary
- Color/contrast meet WCAG AA; no color-only signalling
- Accessible authentication patterns where relevant

---

## Review Content (unchanged structure, augmented expectations)

- **Governance Inputs**: `SPEC_KIT_CONFIG`, constitution path, and any `SPEC_KIT_CONFIG.audit.documents` define
  baseline guardrails and MUST be loaded before analysis. Confirm platform enforcement posture (protected branches,
  required checks, CODEOWNERS).
- **Prerequisite Signals**: Outputs from `/scripts/bash/check-prerequisites.sh --json` (
  especially `FEATURE_DIR` and `AVAILABLE_DOCS`) guide dossier discovery, command authorization, and absolute-path
  enforcement.
- **Conversation Feedback**: `SESSION_FEEDBACK` captures operator-provided feedback, clarifications, and priorities from
  the current chat; treat these entries as authoritative directives that must map to evidence, findings, or
  clarification requests in the deliverable.
- **Feature Dossier**: Files inside `FEATURE_DIR` (`plan.md`, `tasks.md`, `research.md`, plus optional `data-model.md`,
  `contracts/`, `quickstart.md`) capture scope, risks, exclusions, and priorities; use as the authoritative source for
  `REVIEW_SCOPE_HINTS` and `REVIEW_DIRECTIVES`.
- **Scope & Assumptions**: Normalized `RESOLVED_SCOPE`, recorded sandbox/network/approval posture, and the
  assumption-handling strategy govern artifact inspection and `[Open Question]` handling.
- **Control Inventory**: Maintain a structured mapping of cross-cutting controls (logging, auth, telemetry, storage,
  observability, privacy, security) to implementations or known gaps; all mandates, recommendations, and remediations
  should reference these entries where reuse applies.
- **Requirement Catalog**: Combine template-derived gates, constitution mandates, feature-specific directives, and the *
  *Global Normative Baseline** into `REVIEW_REQUIREMENTS`; each requirement includes `id`, `name`, `level`, `rule`,
  `rationale`, `source`.
- **Evidence Expectations**: Quality commands (lint, typecheck, tests, builds, contract checks), dependency baselines vs
  current state, runtime warnings, and diff analyses supply objective signals that substantiate findings.
- **Analysis Dimensions**: Evaluate architecture alignment, cross-cutting controls, regression risks, dependency health,
  security posture, accessibility, performance, and documentation strength before concluding.
- **Deliverable Schema**: Review outputs MUST be rendered from `/templates/audit-template.md`, populating every
  section (Final Status, Resolved Scope narrative, severity-ordered findings with metadata, strengths, outstanding
  `[NEEDS CLARIFICATION]` items, published `CONTROL_INVENTORY`, Quality Signal Summary, Dependency Audit Summary,
  Requirements Compliance Checklist, Decision Log, Remediation Logging).
- **Remediation Standards**: High-severity findings require tasks formatted
  `Context → Control Reference → Actions → Verification`, tied to the relevant control entry or documented gap.
- **Decision Logging**: Record assumptions, constitutional interpretations, control-reuse decisions, remediation audits,
  and `[Open Question]` outcomes for traceability.

---

## Roles & Responsibilities (for reviewers and authors)

- **Authors MUST**: keep CLs small/coherent; link tickets/design docs; run/fix CI; include/update tests; attach relevant
  checklists; request appropriate reviewers/owners; avoid self-approval.
- **Reviewers MUST**: check correctness/design/tests/security/privacy/operational impacts; insist on clarity; be civil
  and actionable; respond within SLA; approve when the CL clearly improves code health.
- **Automation SHOULD**: gate merges (required checks), run SAST/linters/formatters, dependency/SBOM/license checks,
  secret scanning, and (where applicable) verify provenance.

---

## Minimal Viable Set (quick reference)

1. Peer review required for merges to protected branches; no self-approval
2. Required CI checks pass (build/tests/linters/SAST/secret scan)
3. Security & privacy checklists completed; expert review for high-risk changes
4. Traceability (reviewers, outcomes, timestamps, evidence) captured in PR
5. Small, coherent CLs with clear descriptions; linked issue/design
6. Tests added/updated for new behavior; results visible in PR
7. Code Owners approval required for owned paths
8. Dependency/SBOM checks (vulns/licenses/provenance) per policy
9. Production readiness gates for availability/perf-sensitive changes
10. A11y gates for UI changes (WCAG 2.2 AA)
