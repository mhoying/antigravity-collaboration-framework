# Requirements: Antigravity Project Defaults (v4.0)

This document outlines the requirements and specifications for setting up global defaults and behaviors for all projects developed with Antigravity, synthesized from a rigorous multi-persona adversarial review.

## 1. Objectives
- Establish an efficient, frictionless project initialization workflow driven by deterministic scaffolding rather than slow, error-prone LLM text generations.
- Ensure all new projects are automatically tracked with standard best practices (Git, CI/CD, isolated dependencies).
- Iteratively build and refine defaults as patterns emerge, prioritizing on-demand abstraction with strict security sandboxing.

---

## 2. Environment & User Context
- **OS**: ${AG_OS}
- **Editor**: ${AG_EDITOR}
- **Shell**: ${AG_SHELL}
- **Dotfiles**: Managed via a bare git repository:
  - Alias: `dotfiles` (`/usr/bin/git --git-dir=${AG_DOTFILES_DIR} --work-tree=${AG_HOME}`)
- **Git Context**: `gh` CLI for auth, ${AG_EDITOR} for commit editor.

---

## 3. Core Functional Requirements

### 3.1. Phase 0: Zero-Friction Project Initialization
Project initialization must favor extreme velocity and determinism. It MUST occur at Phase 0, before the PRD is drafted, to ensure all architectural provenance is tracked.
1. **Trigger**: User asks Antigravity to "start a new project".
2. **Instant Setup**: Antigravity skips the confirmation prompt. It immediately creates a subdirectory in `$ANTIGRAVITY_WORKSPACE` (default: `${AG_WORKSPACE}`).
3. **Path Jail & Collision Protection**: Antigravity is explicitly forbidden from path traversal (`../`). Path resolution must use canonicalized absolute paths to strictly enforce the jail. If the directory exists, it must safely abort.
4. **Native Scaffolding**: Antigravity MUST use native language scaffolders (e.g., `npm create`, `cargo new`, `uv init`) rather than attempting to dynamically write boilerplate config files via LLM. 
5. **Standardization**:
   - Run `git init`.
   - Run `gh repo create` (if remote syncing is appropriate).
   - Drop a basic `.github/workflows/ci.yml` file to eliminate CI tech debt on day zero.
   - *Security Rule*: Antigravity MUST NOT generate or modify `.git/hooks` without unambiguous, multi-step explicit user approval.
6. **Handoff**: Antigravity responds with the exact terminal command to enter the new environment.

### 3.2. Execution Orchestration Layer
To maintain velocity without drowning the context window, Antigravity must orchestrate complex features rather than writing everything sequentially.
- **State Machine Architecture**: Antigravity operates in two execution modes:
  1. *Sequential Mode*: For simple scripts/components, the AI executes tasks step-by-step in the main chat.
  2. *Lead Orchestrator Mode*: For multi-component features, the primary AI transitions into a TPM role, delegating tasks to specialized subagents operating in parallel.
- **Execution Receipts**: At the end of every project or major task, the AI must copy the `implementation_plan.md` (Design), `task.md` (Execution Log), and `walkthrough.md` (Summary) artifacts into the target project's Git repository to serve as a permanent historical receipt.
- **Parallel Subagent Spawning**: For multi-component tasks (e.g., building a frontend and an API), Antigravity must leverage `invoke_subagent` to spawn specialized execution agents (e.g., Frontend SWE, Database Admin) in parallel.
- **TPM Oversight**: The primary agent must monitor the subagents' progress against strict `task.md` checklists and synthesize their outputs safely into the main branch.
- **Automated Milestone Commits**: The primary agent MUST automatically execute semantic git commits at the end of Phase 1 (PRD Approval) and Phase 2 (TDD Approval) to preserve architectural history without polluting the git log with brainstorming drafts.

### 3.3. Cross-Project Continuous Learning & Skill Extraction
Skill abstraction must never interrupt the user's flow or rely on privacy-violating background surveillance.
- **Correction Memory (`memories.md`)**: To prevent repeating mistakes, the system will maintain a `memories.md` file. Whenever the user corrects an error or establishes a preference (e.g., "always query SQL before the API"), the AI must explicitly ask if this should be saved as a permanent directive.
- **Safe Session Summaries**: To watch for patterns across multiple projects without scraping raw system histories (`~/.zsh_history`), Antigravity will generate anonymized, secret-free "Safe Session Summaries" at the end of tasks, tracking abstract workflows and friction points.
- **Local Metadata Indexing**: Rather than heavy RAG/Vector databases (over-engineering), the `memories.md` directives, session summaries, and `${AG_WORKSPACE}/skills` repository are tracked using a fast, simple local text/metadata index.
- **On-Demand Extraction**: The primary method for skill extraction is an explicit user command (e.g., "Extract what we just did into a skill").
- **Asynchronous Insights & Global Updates**: At the end of a session, Antigravity may provide a non-blocking "Insights Report". If it detects cross-project repetition in the summaries, it will proactively propose creating a new skill OR propose updates to global configurations and `gemini.md` rules.
- **Proposal-Only Mutability**: Any proposed updates to global configs or `gemini.md` MUST be presented as a patch file or manual command. Antigravity is strictly forbidden from silently mutating global state.
- **Binary Shadowing Protection**: Extracted skills MUST use a strict prefix (e.g., `ag-`) to prevent shadowing essential system binaries like `ls` or `git`.

### 3.4. Strict Governance, Efficiency & System Safety
- **Strict Approval Parsing**: For Human-in-the-Loop tasks, the AI must explicitly ask "Do you approve the plan?". It must NEVER interpret user responses containing a question or ambiguity as an approval. The user must explicitly say "yes" or "proceed" to authorize execution.
- **Confidence Reporting**: Whenever the AI lacks high confidence in an answer, proposal, or suggestion, it must explicitly state its confidence level and ask if the user is comfortable proceeding before taking any action.
- **Token Economy & Caching**: The AI must actively cache intermediate data to temporary scratch files and filter context strictly to prevent token bloat and redundant API/tool calls.
- **Config Pruning & Deduplication**: To maintain system efficiency and coherence, the AI must periodically evaluate core instruction files (`gemini.md`, `memories.md`) for conflicting directives or bloated redundancies. Any proposed consolidations must be presented as a patch file to keep the rules lean.
- **Global Config Protection & Hard Write-Block**: Antigravity is strictly forbidden from executing direct file writes (via tools, `echo`, or `tee`) to any file outside of `$ANTIGRAVITY_WORKSPACE`. All updates to global dotfiles or core memory (`gemini.md`) MUST only be presented as patch files for explicit, manual human execution.
- **Sanitization of Extracted Patterns**: To prevent prompt injection from untrusted repos, any proposed skills or config updates in the "Insights Report" MUST undergo comprehensive heuristic and AST analysis to detect obfuscated shell execution, network calls, or malicious aliases. Any flagged operation MUST require a secondary explicit security review before application.
- **Project Isolation**: Generic `git` commands are strictly confined to `${AG_WORKSPACE}/*` directories.
- **Dependency Isolation**: Every project must explicitly manage its dependencies (e.g., virtual environments, containerization). Global package bleeding is prohibited.
- **Terms of Service**: Antigravity must proactively enforce adherence to API ToS and flag questionable data acquisition methods (e.g., scraping).
