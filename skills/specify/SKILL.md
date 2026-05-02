---
description: Generate implementation-grade project specification documents from source code into `spec`
---

# Specify

Generate project specification documents directly from the current codebase.

## Prerequisites

This skill depends on the **everything-claude-code** plugin. Install it before using:

```
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

## Core Rules

- Follow drafting guidance from `specify` skill (this skill) for document writing standards.
- Store all generated specification documents in `spec`.
- Prefer multiple focused files over a single large file.
- Cover **all code modules** in scope, including (but not limited to): `.py`, `.html`, `.js`, `.ts`, `.tsx`, `.jsx`, `.java`, `.sh`, `.go`, `.rs`, `.php`, `.rb`, `.cs`, `.kt`, `.swift`, `.sql`, `.yaml`, `.yml`.
- Documentation must be detailed enough that Claude Code can implement the full project from specs alone.
- Derive specs from source-of-truth files and code; avoid inventing behavior.
- Use `everything-claude-code:plan` to draft a plan before your actions.

## Workflow

1. **Inventory the codebase**
   - Identify project type(s), runtimes, entry points, module boundaries, and infra files.
   - Build a complete module index for all relevant source files in scope.

2. **Map architecture and behavior**
   - Trace request/data/control flow across layers.
   - Capture dependencies, external services, config/env requirements, scripts/commands, and operational behavior.

3. **Generate `spec/` set (small, focused files)**
   - `spec/README.md` — navigation index for all generated specs.
   - `spec/architecture.md` — system structure, boundaries, and flow.
   - `spec/modules.md` — module catalog with purpose and ownership by path.
   - `spec/runtime.md` — setup, scripts, execution model, env/config.
   - `spec/data-model.md` — storage schemas, entities, and relationships.
   - `spec/integrations.md` — third-party APIs/services and interaction contracts.
   - `spec/operations.md` — deploy/runbook, health checks, failure/rollback paths.
   - `spec/implementation-guide.md` — end-to-end rebuild blueprint.
   - `spec/modules/<path-safe-module-name>.md` — one file per significant module or tightly related module group.

4. **Per-module documentation requirements**
   For each documented module, include:
   - File path(s) and responsibility.
   - Public interfaces (functions/classes/endpoints/CLI commands).
   - Inputs/outputs, side effects, and invariants.
   - Internal dependencies and call relationships.
   - Error handling and edge cases.
   - Security considerations and validation boundaries.
   - Reimplementation notes (what must be preserved for parity).

5. **Coverage validation**
   - Produce a coverage matrix in `spec/modules.md` mapping every discovered source module to a documentation target.
   - Explicitly list any skipped/generated/vendor files and the reason.
   - If coverage is incomplete, continue until all in-scope modules are documented.

6. **Staleness and provenance**
   - Add generated markers and scan metadata (date, scope, files scanned).
   - Preserve manually written sections when updating existing specs.

7. **Final summary**
   - Report created/updated files in `spec`.
   - Report module coverage totals and any intentional exclusions.

## Output Quality Bar

The generated specification must provide enough architectural, interface, and behavioral detail for full-project reconstruction without reading the original code.
