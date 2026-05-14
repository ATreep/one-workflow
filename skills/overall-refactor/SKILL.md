---
name: overall-refactor
description: Use when running a full-project refactor spanning dead code cleanup, architecture redesign, spec regeneration, and reimplementation. Triggers on `/overall-refactor` or when the user wants an end-to-end codebase restructuring.
---

# Overall Refactor

Run a full-project refactor sequentially: cleanup, understanding, redesign, documentation, reimplementation.

## Prerequisites

**everything-claude-code** (provides `refactor-clean`, `plan`):
```
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

**superpowers** (provides `brainstorming`):
```
/plugin install superpowers@claude-plugins-official
```

## Mandatory first action

Determine if the project is a web application (browser-based frontend: HTML/CSS/JS, React, Vue, Next.js, SvelteKit, etc.).

- **NOT a web app:** Set `runSimTest = false` automatically. Proceed without asking.
- **IS a web app:** Ask the user:

> "Should I run a simulated test at the end using `sim-test`? (yes/no)"

Store as `runSimTest` (`yes` -> `true`, `no` -> `false`). Resolve before any other step.

## Sequential workflow (strict order)

Complete each step fully before the next.

### 1. Clean dead code

Use `everything-claude-code:refactor-clean` to remove unused code, dead imports, and unreachable logic.

### 2. Read and understand the project

Read all documentation and code in scope. Build a full picture of current architecture, module responsibilities, dependency flow, and patterns.

### 3. Redesign the architecture

Propose a new architecture that balances two competing goals:

**Goal A — Module quality:** Low coupling, clear separation of concerns, well-defined interfaces.
**Goal B — Architecture conciseness:** Few enough modules that a human can hold the structure in their head. Shallow file tree. No unnecessary indirection.

#### Anti-fragmentation rules (non-negotiable)

These prevent the common failure mode of over-splitting into tiny modules:

1. **Minimum viable module.** A module must have a distinct, nameable responsibility. If you cannot describe what it does in one sentence without referencing another module, it does not deserve to be its own module.

2. **Group by bounded context, not by layer.** Related code stays together. A `payments` module can contain its own models, routes, and utilities — do not split into `payments/models/`, `payments/routes/`, `payments/utils/` unless each subfolder is independently large (>500 lines) or independently testable.

3. **Max depth = 3.** No file should be more than 3 directories deep from the project root (e.g., `src/payments/processor.ts` is fine, `src/modules/payments/processing/strategies/card/stripe.ts` is not). If your design requires deeper nesting, flatten it.

4. **Module count heuristic.** For a project of N significant source files, aim for roughly sqrt(N) to N/3 modules. A 30-file project should have 5-10 modules, not 20.

5. **Merge before splitting.** When in doubt, keep things together. Splitting is easy; merging fragmented modules back is painful and rarely happens.

#### When to create a new module

Only split a module when ALL of these are true:
- It has a responsibility that no existing module covers
- It can be described without referencing another module's internals
- It would contain enough code to justify its existence (>200 lines or >3 files)
- It has a different change cadence than its neighbors (changes for different reasons)

#### When to keep modules together

Keep code in the same module when:
- They always change together
- One exists only to wrap or delegate to the other
- Splitting would create circular dependencies between the new modules
- There is no clear bounded context boundary

#### Architecture output format

Produce a module map with:
- Module name and one-sentence responsibility
- Public interface (what it exports/exposes)
- Dependencies (which other modules it uses)
- Estimated size (lines/files)

Use this map as the blueprint for all downstream steps.

### 4. Regenerate documentation

Delete all existing specification files. Then follow `one-workflow:specify` to generate a fresh `spec/` set reflecting the new architecture.

### 5. Reimplement the code

Using `one-workflow:sdd`, rewrite the codebase to conform to the new architecture and specs. Treat new specs as source of truth.

### 6. Run simulated tests (conditional)

If `runSimTest = true`, run `one-workflow:sim-test` to validate the refactored implementation. Otherwise skip.

### 7. Brainstorming

Use `superpowers:brainstorming` during development as needed.

## Rules

- Use `everything-claude-code:plan` to draft a plan before your actions.
- Execute strictly in sequence; no parallelization of phases.
- Do not skip any required phase (except conditional sim-test).
- Keep architecture, specs, and implementation in parity at the end.
- **Architecture conciseness is a hard constraint, not a nice-to-have.** If your redesign doubles the module count without a clear justification, simplify it.

## Output

Provide a concise completion summary:
- `runSimTest` decision (`yes`/`no`)
- Results from each sequential phase
- Architecture redesign: module count before vs. after, with justification for any increases
- Regenerated specs footprint
- Implementation status
- Simulated test result (run/skipped, with outcome if run)
