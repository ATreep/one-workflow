# One Workflow

One Workflow allows you to build a mature software project with simply one skill.

---

## Installation

### As a Plugin

Install via Claude Code's plugin system:

```bash
/plugin marketplace add ATreep/one-workflow
/plugin install one-workflow
```

After installation, invoke the skill with the plugin namespace:

```
/one-workflow:brainstorm-specify-implement
```

---

## Dependencies

This plugin requires the following external plugins to function:

| Plugin | Required Skills | Purpose |
|--------|----------------|---------|
| [Superpowers](https://github.com/obra/superpowers) | `brainstorming`, `writing-plans`, `test-driven-development`, `subagent-driven-development` | Brainstorming, implementation planning, TDD methodology, and parallel subagent execution |
| [SpecKit](https://github.com/obra/speckit) | `speckit-specify`, `speckit-clarify`, `speckit-plan`, `speckit-tasks`, `speckit-implement` | Optional — double spec confirmation pipeline (only if user chooses the SpecKit path) |

Install them before using this plugin:

```bash
/plugin install superpowers
/plugin install speckit
```

---

## Skills

| Skill | Description |
|-------|-------------|
| `/one-workflow:brainstorm-specify-implement` | Transform a rough idea or feature request into actionable implementation: validate prerequisites, brainstorm requirements, write an implementation plan, and optionally run through SpecKit for double spec confirmation before subagent-driven development with TDD |

---

## Plugin Structure

```
one-workflow/
├── .claude-plugin/
│   ├── plugin.json          # Plugin manifest
│   └── marketplace.json     # Marketplace listing
├── skills/                  # Plugin skills (namespaced)
│   └── brainstorm-specify-implement/
│       └── SKILL.md
└── README.md
```
