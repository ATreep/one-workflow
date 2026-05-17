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

4. **Implement via `everything-claude-code:tdd` skill — execute through Agent Team + Parallel Worktrees (primary) or Subagents (fallback)**
   - **Mandatory delegation**: The main thread orchestrates only. Do NOT write production code or tests directly in the main thread.
   - **Primary mode — Agent Team + Parallel Worktrees**: Maximize development speed by parallelizing across worktrees.
     - Spin up a team via `TeamCreate`, spawn teammates with the `Agent` tool (`team_name` + `name`), break the work into tasks via `TaskCreate`, and assign tasks via `TaskUpdate` (`owner`).
     - **Always prefer parallel worktrees**: Analyze the demand and identify every independent track. Create one worktree per track so all tracks develop simultaneously. Only consolidate into fewer worktrees when tracks share the same files and cannot be separated.
     - Use `SendMessage` for inter-teammate communication; rely on the shared task list for status.
   - **Fallback mode — Subagents (linear, single worktree)**: Use only when the demand **cannot** be decomposed into parallel tracks — e.g., a single-file change, a tightly coupled refactor, or an explicitly sequential dependency chain. In this mode, dispatch one or more sequential Subagents via the `Agent` tool on a single worktree.
   - **Inside each teammate/subagent**: Use the `everything-claude-code:tdd` skill for development execution.
     - Stay strict on RED -> GREEN -> REFACTOR.
     - Ensure tests cover modified behavior and regressions.
   - **Trust but verify**: After teammates/subagents report completion, the main thread inspects the actual diff and test results before declaring the step done.
   - **Mandatory worktree lifecycle**: Worktrees are temporary development sandboxes. Every worktree created **must** be merged back into the original local branch before the skill invocation is considered complete. A worktree that is created but never merged is an **incomplete delivery** — the skill is not done until all worktree branches are integrated into the working branch and cleaned up. **Merge automatically — do not wait for user confirmation.**

#### Parallel Worktree Strategy for Multi-Module Demands

When the user proposes **multiple independent demands on multiple modules**, use isolated git worktrees so each demand can be developed in parallel without merge conflicts during development:

1. **Classify demands**: Group the user's requests into independent work items. Each item must target a distinct module or non-overlapping file surface. If two demands touch the same files, they are NOT independent — sequence them instead.

2. **Create worktrees**: For each independent demand, create a worktree with its own branch:
   ```
   git worktree add ../<project>-<demand-slug> -b <demand-slug>
   ```
   Use the `Agent` tool with `isolation: "worktree"` when spawning subagents, or manually manage worktrees when using Agent Team.

3. **Assign one teammate/subagent per worktree**: Each agent works in its own worktree directory. Inside each worktree, the agent follows the full TDD flow (`everything-claude-code:tdd`).

4. **Merge as each worktree completes** (automatic, no user confirmation needed): When an agent finishes its worktree (tests pass, code reviewed):
   - Switch back to the main working branch.
   - Review the diff per file before merging: `git diff main...<demand-slug>`.
   - Handle each conflict carefully — resolve explicitly by reading both sides. Do not overlay files blindly or accept theirs/ours without understanding the change.
   - Merge the worktree branch: `git merge --no-ff <demand-slug>`.
   - Clean up the worktree: `git worktree remove ../<project>-<demand-slug>`.
   - Delete the feature branch after merge: `git branch -d <demand-slug>`.
   - **Do not ask the user before merging.** The worktree was created for this purpose — merge it back immediately upon completion.

5. **Repeat until all worktrees are integrated**: Process merges sequentially as agents complete. If a merge introduces conflicts with a still-running worktree, note the conflict for the remaining agent to rebase against once it completes.

6. **Final verification**: After all worktrees are merged, run the full test suite on the integrated branch to catch cross-module regressions.

7. **Confirm all worktrees merged to local**: Before reporting completion, verify that every worktree created during this invocation has been merged back into the original local branch and that no orphaned worktrees remain (`git worktree list`). This is a hard gate — do not report the skill as complete until all worktrees are merged and cleaned up.

**When to use linear subagents instead** (fallback, single worktree):
- Demand is a single non-decomposable task (one file, tightly coupled changes, or explicitly sequential dependency).
- The overhead of worktree management exceeds the parallelism benefit (e.g., trivial one-line fixes).

**When to use Agent Team on a single branch instead** (no worktrees):
- Demands are tightly coupled, touch the same files, but still benefit from role specialization (e.g., planner + implementer + reviewer coordinating on shared files).

5. **Consistency check after implementation**
   - Verify code and specs match.
   - If implementation details changed during coding, update `spec` again to keep parity.

6. **Brainstorming**
   - Use `superpowers:brainstorming` during development.

## Rules

- Use `everything-claude-code:plan` skill to draft a plan before your actions.
- **Enforce Agent Team + Parallel Worktrees as the default implementation strategy**. The main thread orchestrates and verifies; all production code and tests are written by teammates/subagents. Always decompose the demand into parallel tracks and create one worktree per track. Only fall back to linear subagent mode when the demand is strictly non-parallelizable.
- **Worktrees are mandatory**. When this skill is invoked, worktree(s) must be created for implementation. Each completed worktree **must** be merged back into the original local branch before finishing. Leaving worktrees unmerged is a violation — the delivery is not complete until all branches are integrated. **Merge automatically without user confirmation — never leave a worktree unmerged while waiting for approval.**
- **Handle merge conflicts carefully**. When merging worktree branches, review each file's diff and resolve conflicts explicitly. Do not overlay files blindly or accept default merge strategies without understanding the impact.
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
- **worktree merge status**: confirm all worktrees merged back to local branch (or list any that remain)
