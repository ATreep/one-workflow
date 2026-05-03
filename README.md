# One Workflow

A high-layer DevOps workflow for beginners, based on:
- [Everything-Claude-Code](https://github.com/affaan-m/everything-claude-code).
- [Superpowers](https://github.com/obra/superpowers)

One Workflow allows you to build a mature software project with simply several commands.

---

## Installation

### As a Plugin 

Install via Claude Code's plugin system:

```bash
/plugin marketplace add ATreep/one-workflow
/plugin install one-workflow
```

After installation, invoke skills with the plugin namespace:

```
/one-workflow:specify
/one-workflow:update-spec
/one-workflow:sdd
/one-workflow:sdd-notest
/one-workflow:sim-test
/one-workflow:overall-refactor
```

---

## Skills

| Skill | Description |
|-------|-------------|
| `/one-workflow:specify` | Generate implementation-grade project specification documents from source code into `spec` |
| `/one-workflow:update-spec` | Check what source code has been modified (vs last commit), then update `spec/` documents incrementally |
| `/one-workflow:sdd` | Spec-driven delivery: read specs, update specs first, then implement via TDD with Agent Team |
| `/one-workflow:sdd-notest` | Same as `sdd`, but skips test writing — faster for development when tests aren't required |
| `/one-workflow:sim-test` | End-to-end simulation testing with Playwright MCP screenshots and agent-driven fixes |
| `/one-workflow:overall-refactor` | Full-project refactor: dead code cleanup, redesign, spec regeneration, reimplementation |

---

## Plugin Structure

```
one-workflow/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/                  # Plugin skills (namespaced)
│   ├── specify/
│   │   └── SKILL.md
│   ├── update-spec/
│   │   └── SKILL.md
│   ├── sdd/
│   │   └── SKILL.md
│   ├── sdd-notest/
│   │   └── SKILL.md
│   ├── sim-test/
│   │   └── SKILL.md
│   └── overall-refactor/
│       └── SKILL.md
└── README.md
```
