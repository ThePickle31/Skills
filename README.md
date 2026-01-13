# Skills

A collection of custom Agent Skills for Claude Code and other AI coding assistants.

## Installation

```bash
/plugin marketplace add ThePickle31/Skills
```

Or install individual skills:

```bash
/plugin install ThePickle31/Skills/clawdbot-dev
```

## Available Skills

### clawdbot-dev

Development skill for [Clawdbot](https://github.com/clawdbot/clawdbot) - a multi-platform AI assistant.

**Features:**
- Complete source code structure map
- Runtime config schema (clawdbot.json)
- Agent workspace file documentation
- Development patterns for providers, tools, CLI commands
- Sub-agent system documentation

**Triggers:** `clawdbot`, `provider`, `telegram bot`, `discord bot`, `whatsapp bot`, `gateway`, `agent tools`, `sub-agent`

## Skill Format

Skills follow the open [SKILL.md](https://github.com/anthropics/skills) specification:

```
skill-name/
└── SKILL.md    # YAML frontmatter + markdown instructions
```

## License

MIT
