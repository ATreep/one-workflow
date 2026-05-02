# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`one-workflow` is a **Claude Code plugin**, not an application. It ships four skill prompts that orchestrate higher-order DevOps workflows on top of `everything-claude-code`. There is no compile/test/lint step — the deliverables are markdown skill files plus the plugin manifest.

## Repository layout (only the load-bearing parts)

- `.claude-plugin/plugin.json` — plugin manifest (name, description, version)
- `.claude-plugin/marketplace.json` — marketplace registration; **the `version` field here must always equal the one in `plugin.json`**
- `skills/<name>/SKILL.md` — the four skills exposed by this plugin (`specify`, `sdd`, `sim-test`, `overall-refactor`). Each starts with a `description:` frontmatter that the Claude Code skill loader reads.
- `commands/` — legacy mirror of the same prompts as standalone slash commands. Currently deleted on `main` (see `git status`); the plugin path under `skills/` is the canonical surface.

## Versioning rule (critical)

All version strings in this repo follow the `vYYMMDD` format (e.g. `v260427`).

When invoking `project-version-workflow:update-commit`, read the latest version produced by that skill and update the `version` field in **both** files to that exact value:

- `.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json` (the version inside the `plugins[0]` entry)

**The two files must always stay in lockstep.** Never update one without updating the other. 

## How the four skills relate (architectural big picture)

The skills are designed to compose, not to be used in isolation. Reading any single skill file in isolation hides this:

```
overall-refactor  ──▶  refactor-clean (cleanup phase)
                  ──▶  specify (regenerate specs phase)
                  ──▶  sdd (reimplement phase)
                  ──▶  sim-test (optional final phase, gated by user yes/no)

sdd               ──▶  reads spec/ → updates spec/ → delegates to Agent Team / Subagents
                  ──▶  each teammate uses tdd-workflow skill internally
                  ──▶  uses planner agent for the up-front plan

specify           ──▶  uses planner agent; writes everything under spec/

sim-test          ──▶  wraps e2e-testing skill + Playwright MCP screenshots
```

Two implications when editing these files:

1. **Cross-skill references must stay valid.** If you rename a phase or change a skill's expected output directory (`spec/`, `artifacts/screenshots/`), the calling skills break silently — the plugin doesn't validate this.
2. **Frontmatter `description:` is the routing signal.** Claude Code uses it to decide when to surface the skill. Keep descriptions specific and verb-led.

## Non-obvious invariants encoded in the skills

These are easy to break with "harmless cleanups" — preserve them:

- **`sdd` mandatory delegation**: the main thread must NEVER write production code or tests directly. All implementation goes through `TeamCreate` + `Agent` (Agent Team primary, Subagents fallback). Each teammate runs the `tdd-workflow` skill internally. Don't soften this language.
- **`overall-refactor` mandatory first action**: the skill must ask the user `"Should I run a simulated test at the end using sim-test? (yes/no)"` *before* any other step and store the answer as `runSimTest`. Don't reorder this.
- **`overall-refactor` strict sequential order**: the six phases (clean → understand → redesign → regen specs → reimplement → optional sim-test) must run sequentially, no parallelization. Phase 4 deletes existing specs before regenerating.
- **`specify` output contract**: specs go under `spec/`, prefer many small files over one large file, and the spec set must be detailed enough to rebuild the project from specs alone (it is the spec the `sdd` skill consumes).

## Installation surface (for context when editing the README/manifest)

Users install this plugin two ways — both must keep working:

```
/plugin install one-workflow@github.com/ATreep/one-workflow
```
or
```
/plugin marketplace add ATreep/one-workflow
/plugin install one-workflow
```

After install, skills are namespaced: `/one-workflow:specify`, `/one-workflow:sdd`, `/one-workflow:sim-test`, `/one-workflow:overall-refactor`.

## Working in this repo

- **Editing a skill** = editing the markdown in `skills/<name>/SKILL.md`. Keep frontmatter intact (`---\ndescription: ...\n---`).
- **There are no tests, no build, no linter.** Validation is "does the skill still load and behave as intended when invoked through Claude Code." Smoke-test by invoking `/one-workflow:<skill>` after non-trivial edits.
- **README.md** documents both install paths and the skill table; update it whenever skill names, descriptions, or install instructions change.
