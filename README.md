# OpenClaw Technical Design

Technical design documents for **OpenClaw** — an open-source, multi-channel AI agent platform that acts as a personal assistant gateway.

OpenClaw connects one or more AI agents to multiple messaging surfaces (WhatsApp, Telegram, Slack, Discord, Signal, iMessage, Web Chat) through a single long-lived daemon process.

## Documents

| # | Document | Description |
|---|----------|-------------|
| 00 | [Architecture Overview](00-overview.md) | High-level architecture, component summary, and design principles |
| 01 | [Gateway Architecture](01-gateway-architecture.md) | WebSocket gateway daemon, provider connections, and RPC layer |
| 02 | [Agent Loop](02-agent-loop.md) | LLM inference loop, streaming, tool execution, and compaction |
| 03 | [System Prompt](03-system-prompt.md) | System prompt assembly, runtime parameters, and persona injection |
| 04 | [Tool System](04-tool-system.md) | 25+ built-in tools, plugin tools, and policy-gated execution |
| 05 | [Sub-Agent System](05-sub-agent-system.md) | Hierarchical spawning, steering, and lifecycle management |
| 06 | [Multi-Agent Routing](06-multi-agent-routing.md) | Message routing, session keys, and agent bindings |
| 07 | [Memory System](07-memory-system.md) | Vector-indexed persistent memory with hybrid search (SQLite + embeddings) |
| 08 | [Skills System](08-skills-system.md) | Loadable instruction bundles (SKILL.md), discovery, and lazy loading |
| 09 | [Automation](09-automation.md) | Cron scheduling, event-driven hooks, Gmail watch, and webhooks |
| 10 | [Channels & Messaging](10-channels-and-messaging.md) | Channel abstractions, auto-reply pipeline, and group activation |
| 11 | [Context Management](11-context-management.md) | Context window enforcement, compaction algorithm, and transcript repair |
| 12 | [Sandbox & Security](12-sandbox-and-security.md) | Docker-based execution isolation and configurable access levels |
| 13 | [File Reference](13-file-reference.md) | Complete file reference index across the codebase |

## Key Design Principles

- **Single gateway, many channels** — One daemon process owns all provider connections
- **Per-session serialization** — Runs are queued per session key to prevent transcript races
- **Push-based sub-agents** — Sub-agents announce results when done; no polling needed
- **Lazy skill loading** — Skills are discovered at startup but only read on demand
- **Agent isolation** — Each agent has its own workspace, auth profiles, sessions, and persona
- **Policy-gated tools** — Every tool call passes through configurable allow/deny policies
- **Auto-compaction** — Context overflow triggers automatic summarization and retry

## Architecture

```
Channels (WhatsApp, Telegram, Slack, ...)
        │
        ▼
   Gateway Daemon (Node.js, WebSocket)
        │
   Auto-Reply Pipeline → Routing → Command Queue
        │
   Agent Runtime (System Prompt → LLM Loop → Tool Dispatch)
        │
   Persistence (Sessions, Memory, Config, Skills)
```

## License

These design documents describe the architecture of the OpenClaw project.
