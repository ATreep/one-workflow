---
description: Project-agnostic end-to-end refactor workflow spanning dead code cleanup, architecture redesign, spec regeneration, full reimplementation, and optional simulated testing
---

# Overall Refactor

Run a full-project refactor sequentially across cleanup, understanding, redesign, documentation, and implementation.

## Prerequisites

This skill depends on the following plugins. Install them before using:

**everything-claude-code** (provides `refactor-clean`, `plan`):
```
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

**superpowers** (provides `brainstorming`):
```
/plugin install superpowers@claude-plugins-official
```

## Mandatory first action (before doing anything else)

First, determine if the project is a web application (e.g., uses a browser-based frontend with HTML/CSS/JS, React, Vue, Next.js, SvelteKit, etc.).

- **If the project is NOT a web app:** Set `runSimTest = false` automatically and proceed. Do not ask the user.
- **If the project IS a web app:** Ask the user this exact question and wait for the answer:

> "Should I run a simulated test at the end using `sim-test`? (yes/no)"

Store the response as `runSimTest`:
- `yes` -> `runSimTest = true`
- `no` -> `runSimTest = false`

Do not execute any other step before this determination (and prompt, if applicable) is resolved.

## Sequential workflow (strict order)

Complete each step fully before moving to the next.

1. **Clean dead code**
   - Use the `everything-claude-code:refactor-clean` skill to remove unused code, dead imports, and unreachable logic across the project.

2. **Read and understand the project**
   - Read all existing documentation and code in scope.
   - Build a full picture of current architecture, module responsibilities, dependency flow, and patterns.

3. **Redesign the architecture**
   - Propose a new architecture focused on:
     - high modularity
     - low coupling
     - clear separation of concerns
     - well-defined interfaces between modules
     - minimal cross-module dependencies
   - Use the redesign as the blueprint for all downstream steps.

4. **Regenerate documentation**
   - Delete all existing specification files first.
   - Then follow the `one-workflow:specify` skill to generate a fresh `spec` set that reflects the new architecture.

5. **Reimplement the code**
   - Using the `one-workflow:sdd` skill, rewrite the codebase to conform to the new architecture and newly generated specs.
   - Treat new specs as the source of truth.

6. **Run simulated tests (conditional)**
   - If `runSimTest` is `true`, run the `one-workflow:sim-test` skill to validate the refactored implementation.
   - If `runSimTest` is `false`, skip this step.

7. **Brainstorming**
  - Use `superpowers:brainstorming` during development.

## Rules

- Use `everything-claude-code:plan` to draft a plan before your actions.
- This command is project-agnostic and reusable across repositories.
- Execute strictly in sequence; no parallelization of these six phases.
- Do not skip any required phase except the conditional simulated test when `runSimTest = false`.
- Keep architecture, specs, and implementation in parity at the end.

## Output

Provide a concise completion summary including:
- user decision for simulated testing (`yes`/`no`)
- results from each sequential phase
- regenerated specs footprint
- implementation status
- simulated test result (run/skipped, with outcome if run)
