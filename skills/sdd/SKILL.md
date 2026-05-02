---
description: Spec-driven delivery flow that reads specs, updates specs first for module changes, then implements via tdd inside an Agent Team (primary) or Subagents
---

# SDD

Implement user demand using a documentation-first + TDD workflow.

## Prerequisites

This skill depends on the following plugins. Install them before using:

**everything-claude-code** (provides `tdd`, `plan`):
```
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

**superpowers** (provides `brainstorming`):
```
/plugin install superpowers@claude-plugins-official
```

## Required Workflow

1. **Understand demand first**
   - Parse the user request into concrete outcomes, constraints, and affected modules.
   - Identify whether the request is: add module, modify module, fix module, or non-code/documentation-only.

2. **Read relevant specs from `spec`**
   - Before coding, read the relevant existing specifications in `spec` (architecture, modules, runtime, integrations, and module-specific specs).
   - Use specs to map expected behavior, interfaces, and constraints for affected modules.

3. **Update or create specs before implementation (when module changes are requested)**
   - If the user wants to add/modify/fix any module, update specifications first.
   - Follow the `one-workflow:specify` skill as the specification-writing standard.
   - Keep specs in `spec` and prefer multiple focused files.
   - Ensure changed/new module specs are explicit enough to drive implementation.

4. **Implement via `everything-claude-code:tdd` skill — execute through Agent Team (primary) or Subagents**
   - **Mandatory delegation**: The main thread orchestrates only. Do NOT write production code or tests directly in the main thread.
   - **Primary mode — Agent Team**: Spin up a team via `TeamCreate`, spawn teammates with the `Agent` tool (`team_name` + `name`), break the work into tasks via `TaskCreate`, and assign tasks via `TaskUpdate` (`owner`).
     - Prefer Agent Team when the work has multiple parallelizable tracks, requires coordination across modules, or benefits from role specialization (e.g., planner, implementer, reviewer, test-runner).
     - Use `SendMessage` for inter-teammate communication; rely on the shared task list for status.
   - **Fallback mode — Subagents**: If a single isolated track is sufficient (small surface area, no coordination needed), dispatch one or more Subagents via the `Agent` tool instead of forming a team.
   - **Inside each teammate/subagent**: Use the `everything-claude-code:tdd` skill for development execution.
     - Stay strict on RED -> GREEN -> REFACTOR.
     - Ensure tests cover modified behavior and regressions.
   - **Trust but verify**: After teammates/subagents report completion, the main thread inspects the actual diff and test results before declaring the step done.

5. **Consistency check after implementation**
   - Verify code and specs match.
   - If implementation details changed during coding, update `spec` again to keep parity.

6. **Brainstorming**
  - Use `superpowers:brainstorming` during development.

## Rules

- Use `everything-claude-code:plan` skill to draft a plan before your actions.
- **Enforce Agent Team (primary) or Subagents for implementation**. The main thread orchestrates and verifies; all production code and tests are written by teammates/subagents.
- This command is reusable across projects; do not assume project-specific paths beyond `spec`.
- Do not skip specification read/update phases for module changes.
- Specifications are the implementation contract; code must follow them.
- Prefer small, targeted spec files over monolithic specs.

## Output

Provide a concise execution summary including:
- interpreted user demand
- specs read
- specs created/updated in `spec`
- TDD execution status (tests added/updated, pass/fail)
- implemented modules and final parity status between specs and code
