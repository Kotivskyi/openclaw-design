# OpenClaw — Comprehensive Technical Design

> Agentic Workflow, Features & Use Cases
>
> Source Repository: [github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
>
> Date: February 2026 | Prepared for: Vitalii Kotivskyi

## 1. Executive Summary

OpenClaw is an open-source, multi-channel AI agent platform that acts as a personal assistant gateway. It connects a single AI brain to multiple messaging surfaces (WhatsApp, Telegram, Slack, Discord, Signal, iMessage, web chat, and more) through a long-lived WebSocket gateway daemon. The platform supports multi-agent routing, hierarchical sub-agent orchestration, persistent memory, scheduled automation, browser control, sandboxed execution, and an extensible plugin/skill system.

This document provides a comprehensive technical design of the agentic workflow: how messages flow from inbound channels through the gateway to the AI model, how tools are executed, how sub-agents are spawned and managed, and how results are delivered back to users. Each agent and sub-agent section includes references to the source files containing prompts and evaluation logic.

## 2. High-Level Architecture

### 2.1 Core Components

**Gateway Daemon** — A single long-lived Node.js process that owns all messaging provider connections. It exposes a typed WebSocket API for control-plane clients (macOS app, CLI, web UI). All inbound messages from every channel converge here.

- File: `src/gateway/` (gateway server, protocol, config)
- File: `docs/concepts/architecture.md`

**Agent Runtime (pi-embedded)** — The embedded LLM execution engine built on pi-agent-core and pi-coding-agent. It manages session state, tool dispatch, streaming, compaction, and model failover. Each agent run is serialized per session key to prevent race conditions.

- File: `src/agents/pi-embedded-runner/run.ts`
- File: `src/agents/pi-embedded-subscribe.ts`

**Session Manager** — Maintains per-agent chat history as JSONL transcripts under `~/.openclaw/agents/<agentId>/sessions/`. Sessions are keyed as `agent:<agentId>:<mainKey>` for main sessions and `agent:<agentId>:subagent:<uuid>` for sub-agents.

- File: `src/agents/pi-embedded-runner/session-manager-init.ts`
- File: `src/config/sessions.ts`

**Auto-Reply Pipeline** — The inbound message dispatch pipeline that receives messages from channels, applies command detection, debouncing, group activation checks, and triggers the agent loop.

- File: `src/auto-reply/dispatch.ts`
- File: `src/auto-reply/reply/` (full reply pipeline)

**Tool System** — A rich set of built-in tools (file I/O, shell exec, browser, web search/fetch, cron, messaging, sessions/sub-agents, memory, canvas, nodes, image analysis) plus plugin-provided tools. Each tool is policy-gated.

- File: `src/agents/openclaw-tools.ts`
- File: `src/agents/tools/` (individual tool implementations)

**Skills System** — Loadable instruction bundles (SKILL.md files) that provide domain expertise. Skills are discovered from workspace, bundled, and plugin directories, then injected into the system prompt as available options for the model to read on demand.

- File: `src/agents/skills.ts`
- File: `src/agents/skills/` (workspace, bundled, plugin skills)
- File: `skills/` (50+ bundled skills)

**Memory System** — Persistent vector-indexed memory using SQLite with embedding search (OpenAI, Gemini, Voyage, or local). Supports MEMORY.md bootstrap injection plus on-demand memory_search / memory_get tools.

- File: `src/memory/` (embedding, search, sync)
- File: `src/agents/memory-search.ts`
- File: `src/agents/tools/memory-tool.ts`

### 2.2 Component Interaction Flow

The following table summarizes how the major components interact during a single message-to-reply cycle:

| Step | Component | Action |
|------|-----------|--------|
| 1 | Channel Provider | Receives raw message from WhatsApp/Telegram/Slack/etc. |
| 2 | Auto-Reply Pipeline | Validates, debounces, detects commands, resolves agent routing via bindings |
| 3 | Gateway RPC | Dispatches agent method with session key, message, and context |
| 4 | Session Lane Queue | Serializes the run per session key (prevents concurrent runs) |
| 5 | Agent Runtime | Resolves model/auth, builds system prompt, loads skills snapshot |
| 6 | Pi-Embedded Runner | Executes LLM inference loop with tool calls, streaming, and compaction |
| 7 | Tool Execution | Tools are called in-loop; sub-agents may be spawned asynchronously |
| 8 | Reply Dispatcher | Assembles final payloads, suppresses silent tokens, applies formatting |
| 9 | Channel Provider | Delivers reply to the originating messaging surface |

## 3. The Agent Workflow (Core Loop)

### 3.1 Entry Points

The agent loop can be triggered from two entry points:

- **Gateway RPC:** The agent method is called via WebSocket from any channel adapter, CLI, or automation. It validates params, resolves the session, persists metadata, and returns `{ runId, acceptedAt }` immediately (async fire-and-forget).
- **CLI:** The `openclaw agent --message` command triggers the same pipeline directly.

File: `docs/concepts/agent-loop.md`

### 3.2 Run Lifecycle

Each agent run follows a deterministic lifecycle emitting events on three streams:

| Phase | Stream | Description |
|-------|--------|-------------|
| Accepted | lifecycle | Run ID assigned, queued for execution |
| Start | lifecycle | Session lock acquired, model resolved, system prompt built |
| Inference | assistant | LLM generates text/reasoning tokens streamed as deltas |
| Tool Call | tool | Model requests a tool; tool executes and returns result |
| Compaction | compaction | Context overflow triggers summarize-and-retry |
| End / Error | lifecycle | Run completes or errors; payloads assembled |

### 3.3 Detailed Agent Loop Sequence

Inside the agent runtime, the following steps execute in order:

1. Session lane acquired (per-session serialization queue)
2. Global lane optionally acquired (cross-session throttling)
3. Model resolved from config, per-agent overrides, or per-session overrides
4. Auth profile resolved (API key selection with failover and cooldown)
5. Skills snapshot loaded and environment overrides applied
6. System prompt assembled (see Section 4)
7. Bootstrap context files injected (AGENTS.md, SOUL.md, TOOLS.md, etc.)
8. Pi-agent-core session opened with tool definitions
9. LLM inference loop begins (model generates text and/or tool calls)
10. For each tool call: policy check, execute, return result to model
11. On context overflow: auto-compaction (summarize history, retry)
12. On auth failure: failover to next auth profile (with cooldown)
13. On timeout (default 600s): abort signal sent
14. Final payloads assembled: text, tool summaries, error fallbacks
15. Reply dispatcher delivers to originating channel

- File: `src/agents/pi-embedded-runner/run.ts` (main run orchestration)
- File: `src/agents/pi-embedded-subscribe.ts` (event bridging)
- File: `src/agents/pi-embedded-subscribe.handlers.ts` (stream handlers)

### 3.4 Queueing and Concurrency

Runs are serialized via a two-tier lane system:

- **Session Lane:** One run at a time per session key. This prevents tool/session transcript races and keeps history consistent.
- **Global Lane:** Optional cross-session throttle for resource management.
- **Sub-agent Lane:** Sub-agents run on the `CommandLane.Subagent` lane, isolating them from the main session queue.

- File: `src/agents/lanes.ts`
- File: `src/process/lanes.ts`

## 4. System Prompt Architecture

### 4.1 Prompt Assembly

OpenClaw builds a custom system prompt for every agent run. It does NOT use the default pi-coding-agent prompt. The prompt is assembled by the `buildAgentSystemPrompt` function and includes the following sections:

| Section | Content | Mode |
|---------|---------|------|
| Identity | "You are a personal assistant running inside OpenClaw." | all |
| Tooling | Filtered tool list with short descriptions (case-sensitive names) | all |
| Tool Call Style | Narration rules: silent by default, narrate only complex/sensitive ops | all |
| Safety | No power-seeking, no self-preservation, pause-and-ask on conflicts | all |
| CLI Quick Reference | OpenClaw CLI subcommand reference | full |
| Skills | Available skill descriptions with SKILL.md loading instructions | full |
| Memory Recall | Instructions to search MEMORY.md + memory/*.md before answering | full |
| Self-Update | Gateway config.apply and update.run rules | full |
| Model Aliases | Alias-to-model mappings for overrides | full |
| Workspace | Working directory path and guidance | all |
| Documentation | Local docs path + web mirror URLs | full |
| Sandbox | Container paths, elevated exec, browser bridge info | when sandboxed |
| User Identity | Owner phone numbers for trust verification | full |
| Date & Time | User timezone and current time | all |
| Workspace Files | Bootstrap context files (AGENTS.md, SOUL.md, etc.) | all |
| Reply Tags | Native reply/quote syntax for messaging surfaces | full |
| Messaging | Channel routing, message tool usage, inline buttons | full |
| Voice (TTS) | Text-to-speech hint when configured | full |
| Reactions | Emoji reaction guidance (minimal/extensive) | when enabled |
| Reasoning | Think/final tag format when reasoning tags enabled | when enabled |
| Project Context | Injected bootstrap files + SOUL.md persona directive | all |
| Silent Replies | NO_REPLY token usage rules | full |
| Heartbeats | Heartbeat prompt + HEARTBEAT_OK ack protocol | full |
| Runtime | Agent ID, host, OS, model, channel, thinking level | all |

- File: `src/agents/system-prompt.ts` (prompt builder)
- File: `src/agents/pi-embedded-runner/system-prompt.ts` (embedded wrapper)
- File: `docs/concepts/system-prompt.md`

### 4.2 Prompt Modes

Three prompt modes control which sections are included:

- **full** (default) — All sections included. Used for the main agent session.
- **minimal** — Reduced set for sub-agents. Omits Skills, Memory, Self-Update, Model Aliases, User Identity, Reply Tags, Messaging, Silent Replies, Heartbeats. Keeps Tooling, Safety, Workspace, Sandbox, Date/Time, Runtime, and injected context.
- **none** — Returns only the base identity line. Used for lightweight/utility contexts.

When in minimal mode, the extra system prompt section is labeled "Subagent Context" instead of "Group Chat Context."

### 4.3 Bootstrap Context Files

The following user-editable files are injected into the Project Context section of every prompt:

- **AGENTS.md** — Repository/project guidelines and agent-specific notes
- **SOUL.md** — Persona definition (tone, mannerisms, mood). When present, agent embodies this persona.
- **TOOLS.md** — User guidance for how to use external tools
- **IDENTITY.md** — Agent identity configuration
- **USER.md** — User profile and preferences
- **HEARTBEAT.md** — Heartbeat behavior configuration
- **MEMORY.md / memory.md** — Persistent notes (injected into context window every turn)

*Sub-agent sessions only inject AGENTS.md and TOOLS.md to keep context small. Per-file max is 20,000 chars; total cap is 24,000 chars.*

- File: `src/agents/bootstrap-files.ts`
- File: `src/agents/pi-embedded-helpers.ts`

## 5. Tool System

### 5.1 Built-In Tools

OpenClaw provides a comprehensive set of built-in tools, organized into functional groups:

| Tool | Group | Description |
|------|-------|-------------|
| read | group:fs | Read file contents |
| write | group:fs | Create or overwrite files |
| edit | group:fs | Make precise edits to files |
| apply_patch | group:fs | Apply multi-file patches |
| grep | (core) | Search file contents for patterns |
| find | (core) | Find files by glob pattern |
| ls | (core) | List directory contents |
| exec | group:runtime | Run shell commands (PTY available for TTY-required CLIs) |
| process | group:runtime | Manage background exec sessions |
| web_search | group:web | Search the web (Brave API) |
| web_fetch | group:web | Fetch and extract readable content from a URL |
| browser | group:ui | Control web browser (Playwright/CDP) |
| canvas | group:ui | Present/evaluate/snapshot the Canvas (A2UI host) |
| nodes | group:nodes | List/describe/notify/camera/screen on paired nodes |
| cron | group:automation | Manage cron jobs and wake events (used for reminders) |
| message | group:messaging | Send messages and channel actions (polls, reactions) |
| gateway | group:automation | Restart, apply config, or run updates |
| agents_list | group:sessions | List agent IDs allowed for sessions_spawn |
| sessions_list | group:sessions | List other sessions including sub-agents |
| sessions_history | group:sessions | Fetch history for another session |
| sessions_send | group:sessions | Send a message to another session |
| sessions_spawn | group:sessions | Spawn a sub-agent in an isolated session |
| subagents | group:sessions | List, steer, or kill sub-agent runs |
| session_status | group:sessions | Show usage, time, model state |
| memory_search | group:memory | Search memory via vector embeddings |
| memory_get | group:memory | Get specific memory lines by path |
| image | (core) | Analyze an image with the configured image model |

- File: `src/agents/openclaw-tools.ts` (tool factory)
- File: `src/agents/tools/` (individual implementations)

### 5.2 Tool Policy and Gating

Tools are controlled via a layered policy system:

- **Tool Profiles:** Predefined sets — minimal (session_status only), coding (fs + runtime + sessions + memory), messaging (message + sessions), full (everything)
- **Allow/Deny Lists:** Per-agent configuration via `agents.list[].tools.allow/deny`
- **Tool Groups:** Shorthand references like `group:fs`, `group:web`, `group:sessions` that expand to their member tools
- **Owner-Only Tools:** Certain tools (e.g., whatsapp_login) are restricted to owner numbers only
- **Plugin Tools:** Extensions can provide additional tools, subject to an optional allowlist

- File: `src/agents/tool-policy.ts`
- File: `src/agents/tool-policy-pipeline.ts`

## 6. Sub-Agent System

### 6.1 Overview

OpenClaw implements a hierarchical sub-agent orchestration system. A main agent can spawn background sub-agents in isolated sessions, each with their own session key, model, thinking level, and timeout. Sub-agents run asynchronously and announce their results back to the requester when complete.

### 6.2 Sub-Agent Spawning

The `sessions_spawn` tool creates a new sub-agent:

- **Task:** A natural-language description of what the sub-agent should do
- **Label:** Optional human-readable name for identification
- **Agent ID:** Can target a different agent (e.g., spawn a coding agent from a chat agent), subject to allowlist
- **Model:** Override model for the sub-agent (resolved from config cascade)
- **Thinking:** Override thinking/reasoning level
- **Timeout:** Per-run timeout (default: no timeout for sub-agents)
- **Cleanup:** "delete" (auto-cleanup session after completion) or "keep"

File: `src/agents/tools/sessions-spawn-tool.ts`

### 6.3 Sub-Agent Lifecycle

The sub-agent lifecycle is managed through several coordinated components:

| Phase | Component | Action |
|-------|-----------|--------|
| Spawn | sessions_spawn tool | Validates depth/child limits, creates child session key, patches session store, builds subagent system prompt, calls gateway agent RPC |
| Register | subagent-registry | Registers run in in-memory + disk-persisted registry, starts lifecycle listener, begins wait-for-completion via gateway RPC |
| Execute | pi-embedded runner | Sub-agent runs on the Subagent lane with its own session, model, and system prompt (minimal mode) |
| Monitor | subagents tool (list) | Parent can list active/recent sub-agents with status, model, runtime, token usage |
| Steer | subagents tool (steer) | Parent can send a new message to an active sub-agent (aborts current run, injects message, restarts) |
| Kill | subagents tool (kill) | Parent can terminate a sub-agent (cascading to all descendants) |
| Complete | lifecycle listener | On lifecycle end/error event, outcome is recorded (ok/error/timeout) |
| Announce | subagent-announce | Reads the sub-agent's final reply from session transcript, builds a summary, sends it to the requester session as a system message |
| Cleanup | subagent-registry | Optionally deletes child session and transcript; archives run after configurable timeout |

- File: `src/agents/subagent-registry.ts` (registry + lifecycle)
- File: `src/agents/subagent-announce.ts` (announce flow)
- File: `src/agents/subagent-announce-queue.ts`
- File: `src/agents/subagent-depth.ts`

### 6.4 Depth and Concurrency Limits

Sub-agent spawning is controlled by configurable limits:

- **Max Spawn Depth:** Default 1 (main agent can spawn sub-agents, but sub-agents cannot spawn further sub-agents). Configurable via `agents.defaults.subagents.maxSpawnDepth`
- **Max Children Per Agent:** Default 5 concurrent sub-agents per requester session. Configurable via `agents.defaults.subagents.maxChildrenPerAgent`
- **Allowed Agents:** Cross-agent spawning requires explicit allowlist via `agents.list[].subagents.allowAgents`
- **Archive Timeout:** Completed sub-agent runs are archived after 60 minutes by default

### 6.5 Sub-Agent Prompt and Context

Sub-agents receive a minimal system prompt (`promptMode="minimal"`) plus a Subagent Context section that includes:

- The task description
- The requester session key (for context about who spawned them)
- Depth information and spawn limits
- Only AGENTS.md and TOOLS.md from bootstrap files (other files filtered out)

File: `src/agents/subagent-announce.ts` (`buildSubagentSystemPrompt`)

### 6.6 Steer Mechanism

The steer action allows a parent agent to redirect an active sub-agent mid-execution:

- The current sub-agent run is aborted (via AbortSignal)
- Pending queue items for that session are cleared
- A brief settle period allows the interrupted run to finalize
- A new agent run is dispatched with the steer message, continuing the conversation context
- The sub-agent registry replaces the old run record with the new one

Rate limiting (2 seconds between steers) prevents rapid-fire steer abuse.

File: `src/agents/tools/subagents-tool.ts` (steer action)

## 7. Multi-Agent Routing

### 7.1 Agent Isolation

Each agent is a fully isolated brain with its own workspace, state directory (agentDir), session store, auth profiles, and persona files. The gateway can host one or many agents side-by-side.

- **Workspace:** Separate files, AGENTS.md, SOUL.md, USER.md per agent
- **Auth Profiles:** Per-agent API keys under `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- **Sessions:** Per-agent session store under `~/.openclaw/agents/<agentId>/sessions/`
- **Skills:** Per-agent workspace skills + shared skills from `~/.openclaw/skills`

- File: `docs/concepts/multi-agent.md`
- File: `src/agents/agent-scope.ts`

### 7.2 Routing Rules (Bindings)

Inbound messages are routed to agents via deterministic binding rules (most-specific wins):

1. Peer match (exact DM/group/channel ID)
2. Parent peer match (thread inheritance)
3. Guild ID + roles (Discord role routing)
4. Guild ID (Discord)
5. Team ID (Slack)
6. Account ID match for a channel
7. Channel-level match (`accountId: "*"`)
8. Fallback to default agent

## 8. Channels and Messaging

### 8.1 Supported Channels

OpenClaw supports a wide range of messaging channels, both built-in and via extensions:

| Channel | Type | Location |
|---------|------|----------|
| WhatsApp | Built-in (Baileys) | `src/whatsapp/` |
| Telegram | Built-in (grammY) | `src/telegram/` |
| Slack | Built-in | `src/slack/` |
| Discord | Built-in | `src/discord/` |
| Signal | Built-in | `src/signal/` |
| iMessage | Built-in | `src/imessage/` |
| Web Chat | Built-in | `src/web/` |
| LINE | Built-in | `src/line/` |
| MS Teams | Extension | `extensions/msteams/` |
| Matrix | Extension | `extensions/matrix/` |
| Google Chat | Extension | `extensions/googlechat/` |
| IRC | Extension | `extensions/irc/` |
| Nostr | Extension | `extensions/nostr/` |
| Twitch | Extension | `extensions/twitch/` |
| Mattermost | Extension | `extensions/mattermost/` |
| Feishu/Lark | Extension | `extensions/feishu/` |
| Zalo | Extension | `extensions/zalo/` |
| Voice Call | Extension | `extensions/voice-call/` |

### 8.2 Message Flow

The auto-reply pipeline processes inbound messages through these stages:

- **Inbound Context:** Raw message is contextualized with sender, channel, group info, media attachments
- **Command Detection:** Messages starting with `/` are detected as commands (e.g., `/new`, `/reset`, `/stop`, `/status`)
- **Group Activation:** In group chats, the agent only responds when mentioned or triggered by configured patterns
- **Debouncing:** Multiple rapid messages may be collected before triggering a single agent run
- **Queue Mode:** Channels can use collect (batch), steer (redirect), or followup (append) queueing
- **Dispatch:** The message is dispatched to the agent runtime via the gateway agent RPC

- File: `src/auto-reply/dispatch.ts`
- File: `src/auto-reply/command-detection.ts`
- File: `src/auto-reply/group-activation.ts`

## 9. Memory System

### 9.1 Architecture

OpenClaw provides a persistent, vector-indexed memory system with two tiers:

- **Bootstrap Memory (MEMORY.md):** Injected into the context window on every turn. Consumes tokens. Should be kept concise.
- **Searchable Memory (memory/*.md + sessions):** Accessed on-demand via memory_search and memory_get tools. Does not consume context tokens unless explicitly read.

### 9.2 Embedding Providers

Memory search supports multiple embedding backends:

- **openai** — text-embedding-3-small (default)
- **gemini** — gemini-embedding-001
- **voyage** — voyage-4-large
- **local** — Local ONNX model (no API key required)
- **auto** — Auto-select based on available credentials

### 9.3 Search Pipeline

The memory search pipeline combines vector similarity with text search:

- Hybrid search: weighted combination of vector similarity and BM25 text matching
- Configurable weights, minimum scores, and max results
- Chunking: documents are split into ~400-token chunks with 80-token overlap
- Results include source path and line numbers for citation

- File: `src/memory/` (full memory subsystem)
- File: `src/agents/tools/memory-tool.ts`
- File: `src/agents/memory-search.ts` (config resolution)

## 10. Automation and Scheduling

### 10.1 Cron System

OpenClaw includes a full cron scheduling system for recurring and one-shot tasks:

- **Cron Jobs:** Defined via the cron tool or config. Support standard cron expressions.
- **Wake Events:** One-shot scheduled events (reminders) that fire once and auto-disable.
- **Isolated Agent:** Cron runs execute in an isolated agent session to avoid polluting the main conversation context.
- **Delivery:** Cron results are delivered to the channel that the cron job was created from.

- File: `src/cron/service.ts`
- File: `src/cron/isolated-agent.ts`
- File: `src/agents/tools/cron-tool.ts`

### 10.2 Hooks System

Hooks provide event-driven automation triggered by gateway lifecycle events:

- **Internal Hooks:** `agent:bootstrap`, command events (`/new`, `/reset`, `/stop`)
- **Plugin Hooks:** `before_agent_start`, `agent_end`, `before/after_compaction`, `before/after_tool_call`, `message_received/sending/sent`, `session_start/end`, `gateway_start/stop`
- **Gmail Hooks:** Watch for incoming emails and trigger agent actions via Google Pub/Sub

- File: `src/hooks/hooks.ts`
- File: `src/hooks/internal-hooks.ts`
- File: `src/hooks/gmail.ts`
- File: `docs/automation/hooks.md`

### 10.3 Heartbeat System

The heartbeat mechanism allows the agent to perform periodic self-checks:

- A configurable heartbeat prompt is sent to the agent at regular intervals
- If the agent has nothing to report, it responds with `HEARTBEAT_OK` (silently discarded)
- If something needs attention, the agent replies with an alert instead

File: `src/auto-reply/heartbeat.ts`

## 11. Skills System

### 11.1 Skill Discovery

Skills are instruction bundles discovered from three sources:

- **Workspace Skills:** Located in the agent's workspace `skills/` directory
- **Bundled Skills:** 50+ built-in skills under the `skills/` top-level directory (1password, apple-notes, github, slack, obsidian, weather, etc.)
- **Plugin Skills:** Provided by extensions (e.g., `extensions/open-prose/skills/prose/`)

### 11.2 Skill Loading

Skills are loaded lazily on demand:

- The system prompt includes a Skills section listing available skill names and descriptions
- When the model determines a skill applies, it reads the SKILL.md file using the read tool
- Only one skill is read upfront; the model selects the most specific match
- Skills can define frontmatter metadata for eligibility filtering, platform requirements, and invocation policies

- File: `src/agents/skills/workspace.ts`
- File: `src/agents/skills/frontmatter.ts`
- File: `src/agents/skills/config.ts`

### 11.3 Skill Examples (Bundled)

| Skill | Description | SKILL.md Location |
|-------|-------------|-------------------|
| coding-agent | Coding assistance | `skills/coding-agent/SKILL.md` |
| github | GitHub integration | `skills/github/SKILL.md` |
| slack | Slack workspace operations | `skills/slack/SKILL.md` |
| obsidian | Obsidian vault management | `skills/obsidian/SKILL.md` |
| apple-notes | Apple Notes integration | `skills/apple-notes/SKILL.md` |
| weather | Weather information | `skills/weather/SKILL.md` |
| peekaboo | Camera/screen capture | `skills/peekaboo/SKILL.md` |
| 1password | 1Password vault access | `skills/1password/SKILL.md` |
| spotify-player | Spotify playback control | `skills/spotify-player/SKILL.md` |
| voice-call | Voice call handling | `skills/voice-call/SKILL.md` |
| skill-creator | Create and test new skills | `skills/skill-creator/SKILL.md` |

## 12. Browser and Canvas

### 12.1 Browser Control

OpenClaw provides a dedicated browser tool backed by Playwright/CDP:

- Full web browser automation (navigate, click, type, screenshot, evaluate)
- Bridge server for sandboxed execution (browser runs outside sandbox, agent controls via bridge)
- Support for both host browser and sandbox browser modes

- File: `src/browser/cdp.ts`
- File: `src/browser/chrome.ts`
- File: `src/browser/bridge-server.ts`
- File: `src/agents/tools/browser-tool.ts`

### 12.2 Canvas (A2UI)

The canvas system provides a presentation and evaluation surface:

- Agent can generate interactive HTML/CSS/JS and present it via the canvas
- A2UI specification (vendor) provides structured UI generation and evaluation
- Canvas is served by the gateway HTTP server

- File: `src/canvas-host/`
- File: `src/agents/tools/canvas-tool.ts`
- File: `vendor/a2ui/`

## 13. Sandboxing

OpenClaw supports Docker-based sandboxing for secure agent execution:

- **Modes:** off (no sandbox), all (always sandbox), untrusted (sandbox for non-owner messages)
- **Scope:** shared (one container for all agents) or agent (one container per agent)
- **Workspace Access:** none, read-only (ro), or read-write (rw) host workspace mount
- **Elevated Exec:** Optional host-side execution with approval levels (on/off/ask/full)
- **Browser Bridge:** Allows sandboxed agents to control the host browser via HTTP bridge

- File: `src/agents/sandbox.ts`
- File: `src/agents/sandbox-paths.ts`
- File: `src/agents/pi-embedded-runner/sandbox-info.ts`
- File: `Dockerfile.sandbox`, `Dockerfile.sandbox-browser`

## 14. Context Management

### 14.1 Compaction

When the conversation context approaches the model's token limit, OpenClaw auto-compacts:

- History is split into chunks by token share
- Each chunk is summarized by the LLM (using `generateSummary`)
- Summaries are merged into a single cohesive summary preserving decisions, TODOs, and constraints
- The session transcript is replaced with the compacted summary + recent turns
- A compaction stream event is emitted and a retry is triggered

- File: `src/agents/compaction.ts`
- File: `src/agents/context-window-guard.ts`
- File: `src/agents/pi-extensions/context-pruning/`
- File: `docs/concepts/compaction.md`

### 14.2 Model Failover

Auth profiles support automatic failover:

- Multiple API keys can be configured per provider
- On auth failure, billing error, or rate limit, the system tries the next profile
- Failed profiles enter cooldown periods before retry
- Thinking level can be adjusted as a fallback strategy

- File: `src/agents/auth-profiles.ts`
- File: `src/agents/auth-health.ts`
- File: `src/agents/model-auth.ts`
- File: `src/agents/failover-error.ts`

## 15. Extension/Plugin System

OpenClaw supports a rich extension system for adding channels, tools, skills, and hooks:

| Extension | Type | Description |
|-----------|------|-------------|
| msteams | Channel | Microsoft Teams integration |
| matrix | Channel | Matrix protocol messaging |
| googlechat | Channel | Google Chat workspace integration |
| feishu | Channel + Skills | Feishu/Lark (docs, wiki, drive, permissions) |
| slack | Channel + Skills | Extended Slack operations |
| discord | Channel + Skills | Extended Discord operations |
| open-prose | Skills | Long-form writing guidance |
| lobster | UI | CLI styling and theming |
| memory-core | Extension | Core memory system |
| memory-lancedb | Extension | LanceDB memory backend |
| llm-task | Extension | LLM task orchestration |
| voice-call | Extension | Voice call handling |
| phone-control | Extension | Phone device control |
| diagnostics-otel | Extension | OpenTelemetry diagnostics |

- File: `extensions/` (36 extensions)
- File: `docs/plugins/`

## 16. Evaluation and Testing

### 16.1 Testing Framework

OpenClaw uses Vitest with V8 coverage thresholds (70% lines/branches/functions/statements):

- **Unit Tests:** Co-located `*.test.ts` files alongside source
- **E2E Tests:** Co-located `*.e2e.test.ts` files for integration testing
- **Live Tests:** Require real API keys via `CLAWDBOT_LIVE_TEST=1` or `LIVE=1`
- **Docker Tests:** Containerized tests for gateway and onboarding flows

### 16.2 Test Configuration Files

| Config File | Purpose |
|-------------|---------|
| `vitest.config.ts` | Default test configuration |
| `vitest.e2e.config.ts` | End-to-end test configuration |
| `vitest.unit.config.ts` | Unit test configuration |
| `vitest.gateway.config.ts` | Gateway-specific tests |
| `vitest.extensions.config.ts` | Extension test configuration |
| `vitest.live.config.ts` | Live (real API key) test configuration |

### 16.3 Agent-Related Test Files

Key test files related to agent behavior and prompts:

| Test File | What It Tests |
|-----------|---------------|
| `src/agents/system-prompt.e2e.test.ts` | System prompt assembly and modes |
| `src/agents/system-prompt-params.e2e.test.ts` | System prompt parameter resolution |
| `src/agents/system-prompt-report.test.ts` | System prompt report generation |
| `src/agents/agent-scope.e2e.test.ts` | Agent scope and config resolution |
| `src/agents/tool-policy.e2e.test.ts` | Tool policy enforcement |
| `src/agents/tool-policy.conformance.e2e.test.ts` | Tool policy conformance |
| `src/agents/skills.e2e.test.ts` | Skills loading and filtering |
| `src/agents/skills.resolveskillspromptforrun.e2e.test.ts` | Skills prompt resolution |
| `src/agents/memory-search.e2e.test.ts` | Memory search config |
| `src/agents/compaction.ts` (+ test) | Context compaction logic |
| `src/agents/context-window-guard.e2e.test.ts` | Context window limits |
| `src/agents/pi-embedded-runner/run.overflow-compaction.test.ts` | Overflow compaction behavior |
| `src/agents/tools/subagents-tool.ts` (+ tests) | Sub-agent list/kill/steer |
| `src/agents/tools/sessions-spawn-tool.ts` (+ tests) | Sub-agent spawning |
| `src/auto-reply/dispatch.test.ts` | Message dispatch pipeline |
| `src/cron/service.ts` (multiple .test.ts) | Cron scheduling and delivery |

### 16.4 A2UI Evaluation (Canvas)

The vendor A2UI specification includes a dedicated evaluation framework:

- **evaluation_flow.ts:** Defines a Genkit-based evaluation flow that checks generated UI JSON against user requirements and schemas
- **evaluator.ts:** Orchestrates generation -> analysis -> evaluation pipeline
- **prompts.ts:** Contains LLM prompts for UI generation and evaluation
- **Scoring:** Pass/fail with issue severity levels (minor, significant, critical)

- File: `vendor/a2ui/specification/0.9/eval/src/evaluation_flow.ts`
- File: `vendor/a2ui/specification/0.9/eval/src/evaluator.ts`
- File: `vendor/a2ui/specification/0.9/eval/src/prompts.ts`

## 17. Key Use Cases

### 17.1 Personal AI Assistant

A user deploys OpenClaw on their machine or VPS, connects WhatsApp/Telegram, and gets a persistent AI assistant that remembers context across conversations, manages files, runs code, and controls smart home devices via paired nodes.

### 17.2 Multi-Persona Deployment

A family or team shares one gateway server with multiple isolated agents: each person gets their own workspace, memory, persona (SOUL.md), and messaging accounts, routed via deterministic bindings.

### 17.3 Automated Workflows

Using cron jobs, hooks, and heartbeats, OpenClaw can monitor email inboxes (Gmail hooks), check systems on schedules, send daily summaries, and trigger complex multi-step automations that span sub-agents.

### 17.4 Coding Agent

With the full tool suite (read, write, edit, exec, grep, find, apply_patch), OpenClaw acts as a coding agent that can navigate codebases, run tests, manage git operations, and coordinate multiple coding sub-agents for parallel task execution.

### 17.5 Group Chat Bot

Deployed in WhatsApp or Telegram groups with mention gating, the agent responds when @mentioned, can moderate conversations, answer questions from its memory, and perform actions on behalf of group members.

### 17.6 Browser Automation

Using the browser tool with Playwright/CDP, the agent can navigate websites, fill forms, take screenshots, and perform web-based research tasks, either on the host or within a sandboxed browser container.

## 18. Complete File Reference Index

### 18.1 Agent Prompts

| File | Purpose |
|------|---------|
| `src/agents/system-prompt.ts` | Main system prompt builder (`buildAgentSystemPrompt`) |
| `src/agents/system-prompt-params.ts` | Runtime parameter resolution for system prompt |
| `src/agents/pi-embedded-runner/system-prompt.ts` | Embedded runner system prompt wrapper |
| `src/agents/subagent-announce.ts` | Sub-agent system prompt builder (`buildSubagentSystemPrompt`) |
| `src/auto-reply/reply/session-reset-prompt.ts` | Session reset greeting prompt |
| `extensions/open-prose/skills/prose/guidance/system-prompt.md` | Prose writing system prompt |
| `vendor/a2ui/specification/0.9/eval/src/prompts.ts` | A2UI eval/generation prompts |
| `docs/concepts/system-prompt.md` | System prompt documentation |

### 18.2 Agent Core

| File | Purpose |
|------|---------|
| `src/agents/pi-embedded-runner/run.ts` | Main agent run orchestration |
| `src/agents/pi-embedded-subscribe.ts` | Pi event subscription and bridging |
| `src/agents/pi-embedded-subscribe.handlers.ts` | Stream event handlers |
| `src/agents/pi-embedded-runner/runs.ts` | Run state management (abort, queue, wait) |
| `src/agents/pi-embedded-runner/model.ts` | Model resolution for embedded runs |
| `src/agents/pi-embedded-runner/history.ts` | Session history management |
| `src/agents/pi-embedded-runner/compact.ts` | Compaction execution |
| `src/agents/pi-embedded-runner/lanes.ts` | Lane resolution for embedded runs |
| `src/agents/compaction.ts` | Compaction algorithm (split, summarize, merge) |
| `src/agents/context-window-guard.ts` | Context window size enforcement |

### 18.3 Sub-Agent System

| File | Purpose |
|------|---------|
| `src/agents/tools/sessions-spawn-tool.ts` | Sub-agent spawn tool implementation |
| `src/agents/tools/subagents-tool.ts` | Sub-agent list/kill/steer tool |
| `src/agents/subagent-registry.ts` | In-memory + disk registry of active/recent runs |
| `src/agents/subagent-registry.store.ts` | Disk persistence for subagent registry |
| `src/agents/subagent-announce.ts` | Announce flow (read result, summarize, notify parent) |
| `src/agents/subagent-announce-queue.ts` | Queue for serializing announce operations |
| `src/agents/subagent-depth.ts` | Spawn depth resolution from session store |
| `src/agents/lanes.ts` | Lane constants (Nested, Subagent) |

### 18.4 Tool Implementations

| File | Purpose |
|------|---------|
| `src/agents/openclaw-tools.ts` | Tool factory (creates all tool instances) |
| `src/agents/tool-policy.ts` | Tool profiles and allow/deny policy |
| `src/agents/tool-policy-pipeline.ts` | Tool policy evaluation pipeline |
| `src/agents/tools/browser-tool.ts` | Browser automation tool |
| `src/agents/tools/canvas-tool.ts` | Canvas presentation tool |
| `src/agents/tools/cron-tool.ts` | Cron/reminder scheduling tool |
| `src/agents/tools/memory-tool.ts` | Memory search and retrieval tool |
| `src/agents/tools/message-tool.ts` | Cross-channel messaging tool |
| `src/agents/tools/web-tools.ts` | Web search and fetch tools |
| `src/agents/tools/image-tool.ts` | Image analysis tool |
| `src/agents/tools/gateway-tool.ts` | Gateway management tool |
| `src/agents/tools/nodes-tool.ts` | Paired device control tool |
| `src/agents/tools/session-status-tool.ts` | Session status display tool |

### 18.5 Eval and Test Files

| File | Purpose |
|------|---------|
| `src/agents/system-prompt.e2e.test.ts` | System prompt assembly tests |
| `src/agents/system-prompt-params.e2e.test.ts` | Prompt param resolution tests |
| `src/agents/tool-policy.e2e.test.ts` | Tool policy enforcement tests |
| `src/agents/tool-policy.conformance.e2e.test.ts` | Tool policy conformance tests |
| `src/agents/skills.e2e.test.ts` | Skills loading and filtering tests |
| `src/agents/agent-scope.e2e.test.ts` | Agent scope resolution tests |
| `src/agents/context-window-guard.e2e.test.ts` | Context window guard tests |
| `src/agents/pi-embedded-runner/run.overflow-compaction.test.ts` | Overflow compaction tests |
| `src/agents/tools/memory-tool.citations.e2e.test.ts` | Memory citation tests |
| `src/agents/tools/web-search.e2e.test.ts` | Web search tests |
| `src/auto-reply/dispatch.test.ts` | Auto-reply dispatch tests |
| `src/cron/service.ts` (12+ test files) | Cron service comprehensive tests |
| `vendor/a2ui/specification/0.9/eval/src/evaluation_flow.ts` | A2UI eval flow |
| `vendor/a2ui/specification/0.9/eval/src/evaluator.ts` | A2UI evaluator orchestrator |
| `vendor/a2ui/specification/0.9/eval/src/prompts.ts` | A2UI eval prompts |

## 19. Configuration Reference

Key configuration paths that control agent behavior:

| Config Path | Purpose | Default |
|-------------|---------|---------|
| `agents.defaults.model.primary` | Default LLM model | Provider-dependent |
| `agents.defaults.workspace` | Working directory | `~/.openclaw/workspace` |
| `agents.defaults.timeoutSeconds` | Agent run timeout | 600 |
| `agents.defaults.subagents.maxSpawnDepth` | Max sub-agent nesting | 1 |
| `agents.defaults.subagents.maxChildrenPerAgent` | Max concurrent sub-agents | 5 |
| `agents.defaults.subagents.model` | Default sub-agent model | Inherits parent |
| `agents.defaults.subagents.archiveAfterMinutes` | Archive completed runs after | 60 |
| `agents.defaults.bootstrapMaxChars` | Max per-file bootstrap size | 20000 |
| `agents.defaults.bootstrapTotalMaxChars` | Total bootstrap injection cap | 24000 |
| `agents.defaults.userTimezone` | User timezone | System default |
| `agents.list[].tools.allow/deny` | Per-agent tool restrictions | All allowed |
| `agents.list[].sandbox.mode` | Sandbox mode | off |
| `memory.enabled` | Enable memory system | true |
| `memory.provider` | Embedding provider | auto |

- File: `src/config/config.ts`
- File: `docs/` (comprehensive documentation)
