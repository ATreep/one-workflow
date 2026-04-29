---
description: Spec-driven delivery flow that reads docs, updates docs first for module changes, then implements via tdd inside an Agent Team (primary) or Subagents
---

# SDD

Implement user demand using a documentation-first + TDD workflow.

## Arguments

`$ARGUMENTS`

Treat arguments as the user demand and execution scope. If empty, request the user demand before proceeding.

## Required Workflow

1. **Understand demand first**
   - Parse the user request into concrete outcomes, constraints, and affected modules.
   - Identify whether the request is: add module, modify module, fix module, or non-code/documentation-only.

2. **Read relevant docs from `docs`**
   - Before coding, read the relevant existing documentation in `docs` (architecture, modules, runtime, integrations, and module-specific docs).
   - Use docs to map expected behavior, interfaces, and constraints for affected modules.

3. **Update or create docs before implementation (when module changes are requested)**
   - If the user wants to add/modify/fix any module, update documentation first.
   - Follow the `one-workflow:new-docs` skill as the documentation-writing standard.
   - Keep docs in `docs` and prefer multiple focused files.
   - Ensure changed/new module specs are explicit enough to drive implementation.

4. **Implement via `tdd` skill — execute through Agent Team (primary) or Subagents**
   - **Mandatory delegation**: The main thread orchestrates only. Do NOT write production code or tests directly in the main thread.
   - **Primary mode — Agent Team**: Spin up a team via `TeamCreate`, spawn teammates with the `Agent` tool (`team_name` + `name`), break the work into tasks via `TaskCreate`, and assign tasks via `TaskUpdate` (`owner`).
     - Prefer Agent Team when the work has multiple parallelizable tracks, requires coordination across modules, or benefits from role specialization (e.g., planner, implementer, reviewer, test-runner).
     - Use `SendMessage` for inter-teammate communication; rely on the shared task list for status.
   - **Fallback mode — Subagents**: If a single isolated track is sufficient (small surface area, no coordination needed), dispatch one or more Subagents via the `Agent` tool instead of forming a team.
   - **Inside each teammate/subagent**: Use the `tdd` skill for development execution.
     - Stay strict on RED -> GREEN -> REFACTOR.
     - Ensure tests cover modified behavior and regressions.
   - **Trust but verify**: After teammates/subagents report completion, the main thread inspects the actual diff and test results before declaring the step done.

5. **Consistency check after implementation**
   - Verify code and docs match.
   - If implementation details changed during coding, update `docs` again to keep parity.

6. **Brainstorming**
  - Use `superpowers:brainstorming` during development.

## Rules

- Use `plan` skill to draft a plan before your actions.
- **Enforce Agent Team (primary) or Subagents for implementation**. The main thread orchestrates and verifies; all production code and tests are written by teammates/subagents.
- This command is reusable across projects; do not assume project-specific paths beyond `docs`.
- Do not skip documentation read/update phases for module changes.
- Documentation is the implementation contract; code must follow it.
- Prefer small, targeted doc files over monolithic docs.

## Output

Provide a concise execution summary including:
- interpreted user demand
- docs read
- docs created/updated in `docs`
- TDD execution status (tests added/updated, pass/fail)
- implemented modules and final parity status between docs and code
