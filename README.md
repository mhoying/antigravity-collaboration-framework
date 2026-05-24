# Antigravity Global Environment Setup

## Context
This repository contains the foundational documentation, technical design, and compiled system instructions for the Antigravity AI global environment framework. It is designed to safely enforce a strict, TPM-governed execution model for LLM interactions.

## Quick Start / Installation
To install this framework locally:
1. Clone this repository to your machine.
2. Create your local environment configuration:
   ```bash
   cp .env.example .env
   ```
3. Edit the `.env` file with your specific operating system, editor, and target directory variables.
4. Run the automated scaffolder script to compile the documentation templates and generate your AI "brain" repository:
   ```bash
   chmod +x ~/vibe/skills/bin/ag-scaffold-brain.sh  # If downloaded directly
   ./ag-scaffold-brain.sh
   ```
5. Follow the generated `system_instructions.md` to inject the rules into your IDE settings.

## Files
- `requirements.md`: The Product Requirements Document (PRD).
- `implementation_plan.md`: The Technical Design Document (TDD).
- `system_instructions.md`: The compiled system rules for the IDE.
