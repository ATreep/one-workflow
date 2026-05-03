---
description: Check what source code has been modified (vs last commit), then update `spec/` documents incrementally
---

# Update Spec

Incrementally update `spec/` documents based on what has actually changed in the codebase since the last git commit.

This is a delta-oriented alternative to `/specify` — instead of regenerating all specs from scratch, it detects changes and updates only the affected documents.

## Hard Rules (non-negotiable)

1. **Delta-only updates.** Only update spec files that are affected by the detected source changes. Do not regenerate unrelated documents.

2. **AI-reimplementable fidelity.** Every updated spec must remain detailed enough that another AI agent, given *only* the spec folder, can re-implement the entire project from scratch without reading the original source code. If a detail is needed to reproduce behavior, include it.

3. **Output directory is `spec`.** All generated documents live under `spec/` — not `docs/`, not `specs/`, not any other name. If a `spec/` directory does not exist, create it (this effectively becomes a full `/specify` run).

4. **Preserve manual edits.** When updating an existing spec file, preserve manually written sections. Only update content that corresponds to changed source code.

## Workflow

### 1. Detect Changes

Determine what source code has been modified since the last commit:

- First, check for uncommitted changes: `git diff HEAD` (covers both staged and unstaged).
- If no uncommitted changes exist, compare the latest commit with its parent: `git diff HEAD~1 HEAD`.
- Identify all modified, added, and deleted source files. Ignore non-source files (e.g., `.md`, `.gitignore`, lock files, config files that don't affect behavior).

If no meaningful source changes are found, report this and stop.

### 2. Map Changes to Spec Files

For each changed source file:

- Look up the corresponding spec file in `spec/modules/` (follows `<path-safe-module-name>.md` naming).
- Check whether the change affects other spec files such as `spec/architecture.md`, `spec/data-model.md`, `spec/integrations.md`, etc.
- If a changed module has no corresponding spec file yet, flag it as **new** — it needs to be created.
- If a spec file exists but the source module was deleted, flag it for removal or archival.

### 3. Read and Analyze Affected Specs

For each spec file that needs updating:

- Read the current spec content.
- Read the corresponding source code (current state).
- Identify what has changed and which sections of the spec are now stale.

### 4. Update Specs (delta only)

Apply targeted updates:

- **Modified modules** — update the relevant sections in their `spec/modules/*.md` file (interfaces, behavior, dependencies, etc.).
- **New modules** — create a new `spec/modules/<module-name>.md` following the per-module documentation requirements from `/specify`.
- **Deleted modules** — mark the spec as archived or remove if appropriate.
- **Cross-cutting changes** — if the change affects architecture, data model, integrations, or operations, update those top-level spec files accordingly.
- **Coverage matrix** — update `spec/modules.md` coverage table if modules were added or removed.
- **Index** — update `spec/README.md` navigation if new spec files were added or removed.
- **Provenance** — update scan metadata (date, scope, files scanned) in affected files.

### 5. Per-Module Documentation Requirements

When creating or significantly updating a module spec, include:

- File path(s) and responsibility.
- Public interfaces (functions/classes/endpoints/CLI commands).
- Inputs/outputs, side effects, and invariants.
- Internal dependencies and call relationships.
- Error handling and edge cases.
- Security considerations and validation boundaries.
- Reimplementation notes (what must be preserved for parity).

### 6. Final Summary

Report a clear change summary:

- List of source files detected as changed.
- List of spec files updated/created/archived.
- Any modules that were skipped (non-source, vendor, generated) and why.
- If no specs needed updating, say so explicitly.

## Output Quality Bar

The updated specification must maintain enough architectural, interface, and behavioral detail for full-project reconstruction without reading the original code — same bar as `/specify`, but applied incrementally.
