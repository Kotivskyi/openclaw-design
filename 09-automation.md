# Automation: Cron, Hooks, and Heartbeats

## Overview

OpenClaw provides three automation mechanisms: cron-based scheduling, event-driven hooks, and periodic heartbeat polling. These enable reminders, recurring tasks, email monitoring, and lifecycle event handling.

**Source:** `src/cron/`, `src/hooks/`, `src/auto-reply/heartbeat.ts`

## Cron System

### Architecture

```mermaid
flowchart TD
    subgraph CronService["Cron Service"]
        SCH[Scheduler<br/>cron expressions + one-shot]
        STORE[Cron Store<br/>persistent job definitions]
        TIMER[Timer Manager<br/>prevents duplicate timers]
    end

    subgraph Execution["Execution"]
        ISO[Isolated Agent Session<br/>separate from main conversation]
        DEL[Delivery Plan<br/>route to originating channel]
    end

    TOOL[cron tool<br/>create/list/delete jobs] --> STORE
    STORE --> SCH
    SCH -->|"Timer fires"| ISO
    ISO --> DEL
    DEL --> CH[Channel<br/>WhatsApp/Telegram/etc.]
```

### Cron Tool

The `cron` tool supports creating both recurring jobs and one-shot reminders:

| Action | Description |
|--------|-------------|
| `create` | Create a cron job with schedule expression or one-shot timestamp |
| `list` | List all active cron jobs |
| `delete` | Remove a cron job |
| `update` | Modify an existing job |

**System prompt guidance:** "Use for reminders; when scheduling a reminder, write the systemEvent text as something that will read like a reminder when it fires."

**Source:** `src/agents/tools/cron-tool.ts`, `src/cron/service.ts`

### Isolated Agent Execution

Cron jobs execute in an **isolated agent session** to avoid polluting the main conversation:

```mermaid
sequenceDiagram
    participant SCH as Scheduler
    participant ISO as Isolated Agent
    participant GW as Gateway
    participant CH as Channel

    SCH->>ISO: Fire job (systemEvent text)
    ISO->>ISO: Run agent with event context
    ISO->>ISO: Generate response
    ISO->>GW: Delivery plan (channel, recipient)
    GW->>CH: Deliver response

    Note over ISO: Uses last non-empty agent text<br/>Skips delivery if response is HEARTBEAT_OK
```

**Source:** `src/cron/isolated-agent.ts`

### Tests

| Test File | What It Tests |
|-----------|---------------|
| `src/cron/service.ts` (12+ test files) | Comprehensive service tests |
| `src/cron/service.delivery-plan.test.ts` | Delivery routing |
| `src/cron/service.every-jobs-fire.test.ts` | Recurring job execution |
| `src/cron/service.prevents-duplicate-timers.test.ts` | Timer deduplication |
| `src/cron/service.restart-catchup.test.ts` | Catchup after restart |
| `src/cron/service.runs-one-shot-main-job-disables-it.test.ts` | One-shot auto-disable |
| `src/cron/schedule.test.ts` | Schedule parsing |
| `src/cron/normalize.test.ts` | Job normalization |
| `src/cron/delivery.test.ts` | Delivery logic |
| `src/agents/tools/cron-tool.e2e.test.ts` | Cron tool integration |

---

## Hooks System

### Architecture

```mermaid
flowchart TD
    subgraph HookSources["Hook Sources"]
        INT[Internal Hooks<br/>agent:bootstrap, commands]
        PLG[Plugin Hooks<br/>lifecycle events]
        GM[Gmail Hooks<br/>Google Pub/Sub watch]
    end

    subgraph Events["Lifecycle Events"]
        AB[agent:bootstrap]
        AS[before_agent_start]
        AE[agent_end]
        BC[before_compaction]
        AC[after_compaction]
        BT[before_tool_call]
        AT[after_tool_call]
        MR[message_received]
        MS[message_sending]
        MSE[message_sent]
        SS[session_start]
        SE[session_end]
        GS[gateway_start]
        GST[gateway_stop]
    end

    HookSources --> Events
    Events --> Handler[Hook Handler<br/>user-defined logic]
```

### Internal Hooks (Gateway Hooks)

Event-driven scripts for commands and lifecycle events:

| Hook | Trigger | Use Case |
|------|---------|----------|
| `agent:bootstrap` | Building bootstrap files before system prompt | Add/remove bootstrap context files |
| Command hooks | `/new`, `/reset`, `/stop` | Custom command behavior |

**Source:** `src/hooks/internal-hooks.ts`

### Plugin Hooks

Extension points inside the agent/tool lifecycle:

| Hook | Phase | Use Case |
|------|-------|----------|
| `before_agent_start` | Before run starts | Inject context or override system prompt |
| `agent_end` | After completion | Inspect final message list and metadata |
| `before_compaction` | Before compaction | Observe or annotate |
| `after_compaction` | After compaction | Post-compaction actions |
| `before_tool_call` | Before tool execution | Intercept tool params |
| `after_tool_call` | After tool execution | Intercept tool results |
| `tool_result_persist` | Before transcript write | Transform tool results |
| `message_received` | Inbound message | Pre-processing |
| `message_sending` | Before delivery | Modify outbound message |
| `message_sent` | After delivery | Post-delivery actions |
| `session_start` | Session opens | Session initialization |
| `session_end` | Session closes | Cleanup |
| `gateway_start` | Gateway boots | Startup tasks |
| `gateway_stop` | Gateway shuts down | Shutdown tasks |

**Source:** `src/hooks/hooks.ts`, `src/hooks/loader.ts`

### Gmail Hooks

Watch for incoming emails via Google Pub/Sub:

```mermaid
flowchart LR
    GM[Gmail<br/>Google Pub/Sub] --> WATCH[Gmail Watcher]
    WATCH --> HOOK[Gmail Hook Handler]
    HOOK --> AGENT[Trigger Agent Action]
```

**Source:** `src/hooks/gmail.ts`, `src/hooks/gmail-watcher.ts`, `src/hooks/gmail-ops.ts`

### Tests

| Test File | What It Tests |
|-----------|---------------|
| `src/hooks/internal-hooks.test.ts` | Internal hook execution |
| `src/hooks/loader.test.ts` | Hook loading |
| `src/hooks/workspace.test.ts` | Workspace hook resolution |
| `src/hooks/install.test.ts` | Hook installation |
| `src/hooks/frontmatter.test.ts` | Hook frontmatter parsing |
| `src/hooks/gmail.test.ts` | Gmail integration |
| `src/hooks/gmail-setup-utils.test.ts` | Gmail setup utilities |
| `src/hooks/gmail-watcher.test.ts` | Gmail watcher |
| `src/hooks/hooks-install.e2e.test.ts` | Hook install E2E |

---

## Heartbeat System

### How It Works

```mermaid
sequenceDiagram
    participant HB as Heartbeat Timer
    participant AGENT as Agent
    participant GW as Gateway

    loop Periodic Interval
        HB->>AGENT: Send heartbeat prompt<br/>(configurable text)
        alt Nothing to report
            AGENT-->>HB: HEARTBEAT_OK
            HB->>HB: Discard silently
        else Something needs attention
            AGENT-->>GW: Alert text<br/>(delivered to user)
        end
    end
```

### System Prompt Integration

```
## Heartbeats
Heartbeat prompt: (configured)
If you receive a heartbeat poll matching the heartbeat prompt above,
and there is nothing that needs attention, reply exactly:
HEARTBEAT_OK
OpenClaw treats a leading/trailing "HEARTBEAT_OK" as a heartbeat ack
(and may discard it).
If something needs attention, do NOT include "HEARTBEAT_OK";
reply with the alert text instead.
```

**Source:** `src/auto-reply/heartbeat.ts`, `src/agents/system-prompt.ts`

### Tests

| Test File | What It Tests |
|-----------|---------------|
| `src/auto-reply/heartbeat.test.ts` | Heartbeat prompt detection and ack |
