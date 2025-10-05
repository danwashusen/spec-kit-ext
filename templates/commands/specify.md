---
description: Create or update the feature specification from a natural language feature description.
scripts:
  sh: scripts/bash/create-new-feature.sh --json "{ARGS}"
  ps: scripts/powershell/create-new-feature.ps1 -Json "{ARGS}"
---

The user input to you can be provided directly by the agent or as a command argument - you **MUST** consider it before proceeding with the prompt (if not empty).

User input:

$ARGUMENTS

The text the user typed after `/specify` in the triggering message **is** the feature description. Assume you always have it available in this conversation even if `{ARGS}` appears literally below. Do not ask the user to repeat it unless they provided an empty command.

Given that feature description, do this:

1. Load Spec Kit configuration:
   - Check for `/.specify.yaml` at the host project root; if it exists, load that file
   - Otherwise load `/config-default.yaml`
   - Extract the root `spec-kit` entry and store it as `SPEC_KIT_CONFIG`

2. Run the script `{SCRIPT}` from the repo root and parse its JSON output for BRANCH_NAME and SPEC_FILE. All future file paths must be absolute.
  **IMPORTANT** You must only ever run this script once. The JSON is provided in the terminal as output - always refer to it to get the actual content you're looking for.

3. If defined, read documents from `SPEC_KIT_CONFIG.specify.documents`, refer to them as the document context:
   - For each item, resolve `path` to an absolute path from the repo root
   - Read the file and consider its `context` to guide specification details
   - If a file is missing, note it and continue
   - Consider the file to be read-only, **do NOT modify the file unless instructed to do so**

4. Read the changelog at the path specified by `SPEC_KIT_CONFIG.changelog.path` and incorporate any relevant historical context or conventions into the specification; if it is missing, note the gap and continue.
 
5. Read the constitution at the path specified by `SPEC_KIT_CONFIG.constitution.path` to understand constitutional requirements that the specification must respect.

6. Load `/templates/spec-template.md` to understand required sections.

7. Write the specification to SPEC_FILE using the template structure, replacing placeholders with concrete details derived from the feature description (arguments) while preserving section order and headings.

8. Report completion with branch name, spec file path, and readiness for the next phase.

Note: The script creates and checks out the new branch and initializes the spec file before writing.

Use repository-root anchored paths in generated docs (e.g., `/frontend/src/components/`). Avoid host-specific prefixes
like `/Users/...` or `/home/...`; treat the repository root as `/` for display. Continue using full absolute paths when
running shell/file operations.
