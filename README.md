# One Workflow

A high-layer DevOps workflow for beginners, based on:
- [Everything-Claude-Code](https://github.com/affaan-m/everything-claude-code).
- [Superpowers](https://github.com/obra/superpowers)

One Workflow allows you to build a mature software project with simply several commands.

*Continuously updating...*

---

## Installation

### As a Plugin (Recommended)

Install via Claude Code's plugin system:

```bash
/plugin install one-workflow@github.com/ATreep/one-workflow
```

Or add as a marketplace:

```bash
/plugin marketplace add ATreep/one-workflow
/plugin install one-workflow
```

After installation, invoke skills with the plugin namespace:

```
/one-workflow:new-docs
/one-workflow:sdd
/one-workflow:sim-test
/one-workflow:overall-refactor
```

### As Standalone Commands (Legacy)

Copy the files from `commands/` into your `~/.claude/commands/` directory:

```bash
cp commands/*.md ~/.claude/commands/
```

Then use the short-form commands:

```
/new-docs
/sdd
/sim-test
/overall-refactor
```

---

## Skills

| Skill | Description |
|-------|-------------|
| `/one-workflow:new-docs` | Generate implementation-grade project documentation from source code into `docs` |
| `/one-workflow:sdd` | Spec-driven delivery: read docs, update docs first, then implement via TDD with Agent Team |
| `/one-workflow:sim-test` | End-to-end simulation testing with Playwright MCP screenshots and agent-driven fixes |
| `/one-workflow:overall-refactor` | Full-project refactor: dead code cleanup, redesign, docs regeneration, reimplementation |

---

## Plugin Structure

```
one-workflow/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/                  # Plugin skills (namespaced)
│   ├── new-docs/
│   │   └── SKILL.md
│   ├── sdd/
│   │   └── SKILL.md
│   ├── sim-test/
│   │   └── SKILL.md
│   └── overall-refactor/
│       └── SKILL.md
├── commands/                # Standalone commands (legacy, optional)
│   ├── new-docs.md
│   ├── sdd.md
│   ├── sim-test.md
│   └── overall-refactor.md
└── README.md
```

---

## Plugin Recommendation

- [Superpowers](https://github.com/obra/superpowers): As soon as it sees that you're building something, it *doesn't* just jump into trying to write code. Instead, it steps back and asks you what you're really trying to do.
