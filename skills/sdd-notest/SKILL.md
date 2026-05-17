---
description: Spec-driven delivery flow that reads specs, updates specs first for module changes, then implements inside an Agent Team (primary) or Subagents. Use when speed is a priority and the user demand is low-risk or well-understood.
---

## Prerequisites

This skill delegates to `one-workflow:sdd`, which depends on the following plugins. Install them before using:

**everything-claude-code** (provides `tdd`, `plan`):
```
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

**superpowers** (provides `brainstorming`):
```
/plugin install superpowers@claude-plugins-official
```

# SDD (No Test)

- Use `one-workflow:sdd` skill to implement user demand, but without the requirement for test creation or passing before implementation.
- Without test first development, this workflow may be faster during development but carries higher risk of implementation drift from specs and potential regressions. 
- Obey all rules (except test) defined in `one-workflow:sdd`.
- Merge all worktrees after implementation is complete.