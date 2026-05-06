---
description: End-to-end simulation test workflow using e2e-testing with Playwright MCP screenshots and agent-driven fixes
---

# Sim Test

Run end-to-end validation for the current project with screenshot-driven issue fixing.

## Prerequisites

This skill is **only compatible with web applications**. Before proceeding, verify the project is a web app (e.g., uses a browser-based frontend with HTML/CSS/JS, React, Vue, Next.js, SvelteKit, etc.). If the project is not a web app (CLI tool, library, backend service without a UI, mobile app, etc.), **refuse to run** and inform the user:

> "Simulation testing requires a web application with a browser-based UI. This project does not appear to be a web app, so sim-test cannot be used."

This skill depends on the **everything-claude-code** plugin. Install it before using:

```
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

## Canonical Surface

- Prefer the `everything-claude-code:e2e` skill directly.
- Use this command as an orchestration wrapper for screenshot-first E2E validation and repair loops.

## Workflow

1. **Define test scope**
   - Determine modules/flows to validate from user demand or project docs.
   - Build a module list (pages, routes, features, or service surfaces).

2. **Run E2E via skill**
   - Apply the `everything-claude-code:e2e` skill for test generation/execution.
   - Focus on critical user journeys first, then broader coverage in scope.

3. **Capture module screenshots with Playwright MCP**
   - During test execution, use Playwright MCP to capture screenshots for each module/surface in scope.
   - Save screenshots with stable names, for example:
     - `artifacts/screenshots/<module>/<state>.png`
   - Capture both normal and failure states when available.

4. **Agent-driven visual triage and fixes**
   - For every captured screenshot, pass the screenshot + relevant test/context to dev agents.
   - Agents should detect unexpected UI/UX/functional issues and propose/fix code changes.
   - Re-run targeted E2E tests after each fix batch.

5. **Iterate to green**
   - Continue screenshot -> agent triage -> fix -> re-test until scoped flows are stable.
   - Escalate only blockers that cannot be resolved automatically.

## Rules

- Reusable across projects; do not assume project-specific structure.
- Do not skip screenshot capture for in-scope modules.
- Do not skip agent triage for captured screenshots.
- Keep fixes traceable to failing or unexpected visual/behavioral evidence.

## Output

Return a concise summary including:
- scope tested
- E2E skill execution result
- screenshot inventory (by module)
- issues found from screenshot triage
- fixes applied
- final pass/fail status and remaining blockers
