# Technical Design Document: Antigravity Global Environment

This document defines the system architecture, data models, and specific implementation mechanisms required to fulfill the functional requirements outlined in the v4.4 PRD (`requirements.md`). 

## User Review Required
> [!IMPORTANT]
> **TDD Finalized via Agent Review**: The Agent Team reviewed this document and strongly recommended the Unix-native `ripgrep` indexing over SQLite to prevent Git merge conflicts and binary bloat in your `~/vibe/skills` repository. The schemas have been updated to support this. If you approve this final design, we are ready to execute!

---

## 1. Zero-Friction Project Initialization Engine (Ref: PRD 3.1)
This component handles the "start project" trigger.

### Architecture
- **Input Parsing**: The AI analyzes the user prompt to deduce the intended Tech Stack.
- **Path Validation Jail**: Before executing any command, the target path must be strictly within `^${AG_WORKSPACE}/.*`. Validation must use canonicalized absolute paths to prevent `../` directory traversal attacks.
- **Scaffolding Passthrough**: The AI executes native package managers strictly in non-interactive mode (e.g., `--yes`, `--quiet`) to prevent terminal hangs.
  - *Example*: `npm create vite@latest <dir> -- --template react-ts`
- **CI/CD Injection**: The AI copies a standard workflow template from a local base (e.g., `${AG_WORKSPACE}/skills/templates/ci-base.yml`) into the newly created `.github/workflows/`.

---

## 2. Execution Orchestration Layer (Ref: PRD 3.2)
This component governs how complex code is written and synthesized.

### Architecture
- **State Machine**:
  - *Sequential Mode*: The primary agent writes directly to the file system, updating `task.md` sequentially.
  - *Lead Orchestrator Mode*: Triggered by the TPM persona for multi-component tasks.
- **Subagent Delegation**: The primary agent calls `invoke_subagent` with specific Role constraints.
- **Branch/State Isolation**: Subagents are directed to output their work to isolated scratch directories or Git branches to prevent collision. The primary agent (TPM) reviews the diffs and merges them into the main branch.

---

## 3. Continuous Learning & Memory Architecture (Ref: PRD 3.3)
The data plane responsible for cross-project intelligence. All learned intelligence resides in the `${AG_WORKSPACE}/skills` Git repository to ensure version control and portability.

### Data Models & Schemas
#### [NEW] `${AG_WORKSPACE}/skills/memories.md`
- **Format**: Markdown list of explicit behavioral directives with standard tags for indexing.
- **Schema**: `[#tags] | [Context Trigger] -> [Explicit Directive]`
  - *Example*: `[#buganizer #sql] | [Buganizer API] -> Always query SQL issuestats table first.`

#### [NEW] `${AG_WORKSPACE}/skills/summaries/<session_id>.md`
- **Format**: Anonymized Markdown with YAML frontmatter.
- **Schema**: 
  - Frontmatter: `Tech Stack`, `Tags/Keywords`, `Project Domain`
  - Body: `Tools Used`, `Errors Encountered`, `Resolution Steps`. No raw logs or PII.

#### [NEW] `${AG_WORKSPACE}/skills/bin/ag-<skill_name>`
- **Format**: Executable Bash/Python scripts.
- **Constraint**: Must begin with `ag-` to prevent binary shadowing. Must include standard `--help` output.

### Indexing & Discovery Engine
- Based on architectural review, the system avoids both SQLite and RAG. Instead, Antigravity will natively keep track of available skills by using its built-in directory scanning and text-search tools (`grep_search`, `list_dir`) to read `memories.md` and the `${AG_WORKSPACE}/skills` repository at the beginning of a session or when generating an Insights Report.

---

## 4. Security & Governance Control Plane (Ref: PRD 3.4)
The strict rule-engine that sandboxes the AI.

### Architecture
- **Strict Approval Parsing**: The AI must mechanically ask "Do you approve the plan?" before executing CRUD tasks. Any response containing questions or ambiguity must trigger an abort/re-clarification. The AI must parse for an explicit "yes" or "proceed".
- **Confidence Reporting**: The AI is programmed to prepend a "Confidence Level: [High/Medium/Low]" tag to proposals. If not High, the AI must explicitly pause execution until the user signs off on the risk.
- **System-Enforced Hard Write Block**: The AI is blocked from writing to global files not only by prompt rules but by hardcoded backend system boundaries (e.g., the system explicitly returns "Permission denied" if the AI attempts to read/write `~/.gemini/antigravity`). 
- **Global Config Patch Generation**:
  - When updating global rules, the AI generates standard unified diff format (`.patch` files) and saves them to `${AG_WORKSPACE}/skills/patches/`.
  - The AI outputs the command: `git apply ${AG_WORKSPACE}/skills/patches/update.patch` for the user to manually execute.
- **Sanitization Pipeline**:
  - During the "Insights Report" generation, the AI performs comprehensive heuristic and AST analysis against its own proposed skills. If it detects obfuscated shell execution, network calls, or malicious aliases, it prepends a `[SECURITY REVIEW REQUIRED]` tag to the UI output.

---

## Verification Plan

### Global System Instructions Update (Manual Human Execution)
Because the AI is sandboxed from modifying the IDE's core configuration, the human user MUST perform the following steps:
1. Open the `system_instructions.md` artifact and copy the entire markdown text.
2. Navigate to the IDE's AI configuration panel (typically found at **Settings / Preferences -> Antigravity / AI Assistant -> Custom Instructions** or within the **User Global Rules** text box).
3. Paste the markdown block directly into the text box and save the configuration. This ensures the rules are injected into the `<RULE[user_global]>` block for all future sessions.
4. The AI must explicitly ask the user: "Have you pasted the updated system instructions into your global settings?" and wait for a "yes" confirmation before proceeding to the automated setup.

### Automated Setup
**1. Setup Documentation Repository (`${AG_SETUP_DIR}/`)**
- Initialize `git init` in `${AG_SETUP_DIR}/`.
- Export the `system_instructions.md` and `implementation_plan.md` artifacts into this directory so the PRD, TDD, and Rules are permanently tracked in Git.

**2. Skills Repository (`${AG_WORKSPACE}/skills/`)**
- Create the structural directories (`bin`, `summaries`, `templates`, `patches`) in `${AG_WORKSPACE}/skills`.
- Initialize `git init` and create `.gitignore`.
- Create empty `memories.md` and `INDEX.md`.

**3. Skill Extraction: Global Environment Scaffolder**
- Create an executable skill `ag-scaffold-brain.sh` in `${AG_WORKSPACE}/skills/bin/` that automatically replicates this entire folder structure, git configurations, and base files so it can be easily shared or executed on a fresh machine.
- Add an entry for `ag-scaffold-brain.sh` into the `INDEX.md` manifest.

### Manual Validation
- User confirms the folder architecture matches their expectations.
