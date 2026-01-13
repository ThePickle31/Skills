---
name: clawdbot-dev
description: |
  Use when developing features for Clawdbot - the multi-platform AI assistant.
  Triggers: clawdbot development, provider implementation, telegram bot, discord bot,
  whatsapp bot, signal bot, slack bot, gateway, agent tools, pi-agent, sessions,
  chat routing, sub-agent, subagent
---

# Clawdbot Development

Clawdbot is a multi-platform AI assistant with providers for Discord, Telegram, WhatsApp, Slack, Signal, iMessage, and MS Teams.

**Repository:** https://github.com/nickclaw/clawdbot

## Getting Started

```bash
git clone https://github.com/nickclaw/clawdbot.git
cd clawdbot
pnpm install
```

## Ecosystem Overview

| Directory | Purpose |
|-----------|---------|
| `<clawdbot-repo>/` | Source code repo |
| `<clawdbot-repo>/docs/` | Documentation (Mintlify markdown files) |
| `~/.clawdbot/` | Runtime config (clawdbot.json, credentials, agents, cron) |
| `~/clawd/` | Default agent workspace (identity, memory, workspace files) |

## Source Code Structure

```
src/
├── agents/              # Agent framework
│   ├── tools/           # Agent tool definitions (TypeBox schemas)
│   ├── subagent-*.ts    # Sub-agent lifecycle & announcements
│   └── pi-*.ts          # Pi-agent integration
├── cli/                 # CLI framework (Commander.js)
│   ├── program.ts       # Main CLI builder
│   └── run-main.ts      # Entry bootstrap
├── commands/            # Individual CLI commands
├── config/              # Configuration loading/validation
│   ├── config.ts        # Config loader
│   └── zod-schema.ts    # Config validation schemas
├── gateway/             # Core message routing hub
│   ├── server.ts        # Gateway server
│   └── call.ts          # Gateway RPC client
├── providers/           # Provider abstraction layer
│   ├── plugins/         # Provider plugin implementations
│   │   ├── types.ts     # ProviderPlugin interface
│   │   ├── index.ts     # Plugin registry
│   │   └── {provider}.ts
│   └── registry.ts      # CHAT_PROVIDER_ORDER
├── discord/             # Discord provider
├── telegram/            # Telegram provider (grammy)
├── whatsapp/            # WhatsApp provider (baileys)
├── slack/               # Slack provider (@slack/bolt)
├── signal/              # Signal provider
├── imessage/            # iMessage provider
├── msteams/             # MS Teams provider
├── routing/             # Message routing logic
├── sessions/            # Session management
└── types/               # Shared TypeScript types
```

## Runtime Structure

### Config Directory (`~/.clawdbot/`)
```
~/.clawdbot/
├── clawdbot.json        # Main config
├── .env                 # Environment variables
├── agents/              # Per-agent runtime data
│   └── {agentId}/
│       ├── sessions/    # Session store (sessions.json)
│       └── agent/       # Agent state
├── credentials/         # Stored credentials (OAuth tokens, API keys)
├── cron/                # Scheduled tasks
├── browser/             # Browser automation data (Playwright state)
└── media/               # Media storage
```

#### clawdbot.json Schema
```json
{
  "wizard": {},           // Setup wizard state (lastRunAt, lastRunVersion)
  "auth": {
    "profiles": {}        // Auth profiles: provider → {mode: "oauth"|"token"|"api_key"}
  },
  "agents": {
    "defaults": {
      "model": {},        // Default model config
      "models": {},       // Available models with aliases
      "workspace": ""     // Default workspace path (e.g., "~/clawd")
    },
    "list": [{            // Agent definitions
      "id": "main",
      "identity": {},     // name, theme, emoji
      "workspace": "",    // Override workspace path
      "model": "",        // Override model
      "subagents": {
        "allowAgents": [] // Agents this agent can spawn as sub-agents
      }
    }]
  },
  "tools": {
    "deny": [],           // Tools to disable
    "agentToAgent": {}    // Cross-agent communication settings
  },
  "messages": {},         // Message handling config
  "commands": {},         // Command settings (native, restart)
  "gateway": {            // Gateway settings
    "port": 18789,
    "mode": "local",
    "bind": "loopback"
  },
  // Provider-specific configs:
  "discord": {},
  "telegram": {},
  "whatsapp": {},
  "slack": {},
  "signal": {}
}
```

### Agent Workspace (`~/clawd/` by default)

The workspace is the agent's persistent memory and identity. Path is configurable per-agent.

```
<workspace>/
├── AGENTS.md            # Workspace instructions (read on session start)
├── IDENTITY.md          # Agent identity: name, creature type, vibe, emoji
├── SOUL.md              # Persona & behavioral boundaries
├── USER.md              # User profile and preferences
├── TOOLS.md             # Custom tool documentation
├── HEARTBEAT.md         # Checklist for heartbeat runs (periodic tasks)
├── memory/              # Daily memory logs
│   └── YYYY-MM-DD.md    # Daily notes, facts, decisions (read today + yesterday)
└── canvas/              # Canvas rendering output
```

**Workspace File Purposes:**
| File | Read When | Purpose |
|------|-----------|---------|
| `AGENTS.md` | Session start | Safety defaults, backup tips, daily memory guidance |
| `IDENTITY.md` | Session start | Who the agent IS (name, personality, emoji) |
| `SOUL.md` | Session start | Tone, boundaries, behavioral rules |
| `USER.md` | Session start | User preferences and profile |
| `TOOLS.md` | On demand | Documentation for custom/external tools |
| `HEARTBEAT.md` | Heartbeat runs | Checklist of periodic tasks to perform |
| `memory/*.md` | Session start | Persistent facts, preferences, decisions |

## Key Files Reference

| File | Purpose |
|------|---------|
| `src/entry.ts` | CLI entry point |
| `src/cli/program.ts` | CLI builder |
| `src/providers/plugins/types.ts` | ProviderPlugin interface |
| `src/providers/registry.ts` | Provider list & metadata |
| `src/agents/tools/common.ts` | Tool utilities (jsonResult, etc.) |
| `src/config/config.ts` | Config loader |
| `src/config/zod-schema.ts` | Config validation |
| `src/gateway/server.ts` | Gateway server |
| `src/gateway/call.ts` | Gateway RPC client |

## Development Patterns

### Adding a New Provider

1. Create plugin: `src/providers/plugins/{provider}.ts`
```typescript
import type { ProviderPlugin } from "./types.js";

export const myProviderPlugin: ProviderPlugin<MyAccount> = {
  id: "myprovider",
  meta: { label: "My Provider", docsPath: "/providers/myprovider" },
  capabilities: { chatTypes: ["dm", "group"], polls: false, reactions: true },
  // ... implement full interface
};
```

2. Create provider directory: `src/{provider}/`
   - `bot.ts` - Provider-specific bot logic
   - `send.ts` - Message sending
   - `monitor.ts` - Message receiving
   - `accounts.ts` - Account management

3. Register in `src/providers/plugins/index.ts`:
```typescript
import { myProviderPlugin } from "./myprovider.js";
// Add to resolveProviders() return array
```

4. Add to `src/providers/registry.ts`:
```typescript
export const CHAT_PROVIDER_ORDER = [..., "myprovider"];
```

### Adding a New Agent Tool

1. Create tool: `src/agents/tools/{tool-name}.ts`
```typescript
import { Type } from "@sinclair/typebox";
import type { AnyAgentTool } from "./common.js";
import { jsonResult } from "./common.js";

const Schema = Type.Object({
  action: Type.String(),
  // ... parameters
});

export function createMyTool(): AnyAgentTool {
  return {
    label: "Category",
    name: "my_tool",
    description: "What this tool does",
    parameters: Schema,
    execute: async (toolCallId, args) => {
      // Implementation
      return jsonResult({ status: "ok" });
    },
  };
}
```

2. Register in the appropriate tool factory or pi-tools.ts

### Adding a CLI Command

1. Create command: `src/commands/{command}.ts`
```typescript
import { Command } from "commander";

export function myCommand(): Command {
  return new Command("mycommand")
    .description("What this command does")
    .option("-f, --flag <value>", "Option description")
    .action(async (options) => {
      // Implementation
    });
}
```

2. Register in `src/cli/program.ts`

### Sub-agent System

Sub-agents are isolated agent runs spawned for specific tasks.

**Key Files:**
- `src/agents/subagent-registry.ts` - Tracks running sub-agents, lifecycle
- `src/agents/subagent-announce.ts` - Announces results back to requester
- `src/agents/tools/sessions-spawn-tool.ts` - The `sessions_spawn` tool

**Session Key Format:** `agent:{agentId}:subagent:{uuid}`

**Rules:**
- Sub-agents cannot spawn nested sub-agents (forbidden)
- Cross-agent spawning requires allowlist: `agents.list[].subagents.allowAgents`
- Sub-agents get a special system prompt defining their ephemeral role
- Results are announced back to the requester chat

**Config Example:**
```json
{
  "agents": {
    "list": [{
      "id": "main",
      "subagents": {
        "allowAgents": ["gemini-flash", "browser-agent"]
      }
    }]
  }
}
```

## Tech Stack

- **Language:** TypeScript (ES2022, CommonJS output)
- **Package Manager:** pnpm (Bun also supported)
- **CLI:** Commander.js
- **Agent Framework:** @mariozechner/pi-agent-core
- **Validation:** Zod + TypeBox (for tool schemas)
- **Providers:** grammy (Telegram), baileys (WhatsApp), @slack/bolt (Slack), discord-api-types
- **Build:** tsc → /dist
- **Testing:** Vitest

## Documentation

Documentation is in the `docs/` directory (Mintlify). Also available at https://docs.clawd.bot

| Path | Content |
|------|---------|
| `docs/start/` | Getting started |
| `docs/concepts/` | Architecture, agents, sessions |
| `docs/providers/` | Provider setup guides |
| `docs/tools/` | Agent tools |
| `docs/gateway/` | Gateway config |

## Quick Search

```bash
# Find in source (run from repo root)
grep -r "pattern" src/

# Find in docs
grep -r "pattern" docs/

# List all tools
ls src/agents/tools/

# List all providers
ls src/providers/plugins/
```

## Common Commands

```bash
pnpm install          # Install dependencies
pnpm build            # Build TypeScript
pnpm test             # Run tests
pnpm docs:dev         # Run local docs server
clawdbot gateway      # Start the gateway
clawdbot doctor       # Health check
```
