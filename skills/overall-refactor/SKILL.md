---
description: Project-agnostic end-to-end refactor workflow spanning dead code cleanup, architecture redesign, docs regeneration, full reimplementation, and optional simulated testing
---

# Overall Refactor

Run a full-project refactor sequentially across cleanup, understanding, redesign, documentation, and implementation.

## Arguments

`$ARGUMENTS`

Treat arguments as optional scope constraints. If omitted, operate on the full repository.

## Mandatory first action (before doing anything else)

Immediately ask the user this exact question and wait for the answer:

> "Should I run a simulated test at the end using `sim-test`? (yes/no)"

Store the response as `runSimTest`:
- `yes` -> `runSimTest = true`
- `no` -> `runSimTest = false`

Do not execute any other step before this prompt is answered.

## Sequential workflow (strict order)

Complete each step fully before moving to the next.

1. **Clean dead code**
   - Use the `refactor-clean` skill to remove unused code, dead imports, and unreachable logic across the project.

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
   - Delete all existing documentation files first.
   - Then follow the `one-workflow:new-docs` skill to generate a fresh `docs` set that reflects the new architecture.

5. **Reimplement the code**
   - Using the `one-workflow:sdd` skill, rewrite the codebase to conform to the new architecture and newly generated docs.
   - Treat new docs as the source of truth.

6. **Run simulated tests (conditional)**
   - If `runSimTest` is `true`, run the `one-workflow:sim-test` skill to validate the refactored implementation.
   - If `runSimTest` is `false`, skip this step.

7. **Brainstorming**
  - Use `superpowers:brainstorming` during development.

## Rules

- Use `planner` agent to draft a plan before your actions.
- This command is project-agnostic and reusable across repositories.
- Execute strictly in sequence; no parallelization of these six phases.
- Do not skip any required phase except the conditional simulated test when `runSimTest = false`.
- Keep architecture, docs, and implementation in parity at the end.

## Output

Provide a concise completion summary including:
- user decision for simulated testing (`yes`/`no`)
- results from each sequential phase
- regenerated docs footprint
- implementation status
- simulated test result (run/skipped, with outcome if run)
