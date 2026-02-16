# Agent Loop Lifecycle

## Overview

The agent loop is the core execution path: intake, context assembly, model inference, tool execution, streaming replies, and persistence. Each loop is a single serialized run per session that emits lifecycle and stream events.

**Source:** `src/agents/pi-embedded-runner/run.ts`, `docs/concepts/agent-loop.md`

## Entry Points

| Entry | Description |
|-------|-------------|
| **Gateway RPC** | `agent` method via WebSocket from any channel, CLI, or automation |
| **CLI** | `openclaw agent --message` command |

## High-Level Flow

```mermaid
flowchart TD
    A[Message Arrives] --> B[Gateway agent RPC]
    B --> C{Validate params<br/>Resolve session}
    C --> D[Return runId immediately<br/>async fire-and-forget]
    D --> E[agentCommand]

    E --> F[Resolve model + thinking defaults]
    F --> G[Load skills snapshot]
    G --> H[runEmbeddedPiAgent]

    H --> I[Serialize via session lane]
    I --> J[Resolve model + auth profile]
    J --> K[Build pi session]
    K --> L[subscribeEmbeddedPiSession]

    L --> M{LLM Inference Loop}
    M -->|Text| N[Stream assistant deltas]
    M -->|Tool Call| O[Execute tool]
    O --> P[Return result to model]
    P --> M
    M -->|Context Overflow| Q[Auto-compaction]
    Q --> M
    M -->|Complete| R[Assemble final payloads]

    R --> S[Reply Dispatcher]
    S --> T[Deliver to channel]
```

## Detailed Step-by-Step

```mermaid
sequenceDiagram
    participant CH as Channel
    participant GW as Gateway
    participant SL as Session Lane
    participant RT as Agent Runtime
    participant LLM as LLM Provider
    participant TL as Tool System

    CH->>GW: Inbound message
    GW->>GW: Validate, resolve session key
    GW-->>CH: ack {runId, acceptedAt}

    GW->>SL: Enqueue run (per-session serialization)
    SL->>RT: Acquire session lock

    RT->>RT: Resolve model from config cascade
    RT->>RT: Resolve auth profile (API key)
    RT->>RT: Load skills snapshot
    RT->>RT: Build system prompt (see 03-system-prompt.md)
    RT->>RT: Inject bootstrap context files
    RT->>RT: Open pi-agent-core session

    loop LLM Inference Loop
        RT->>LLM: Send messages + tools
        LLM-->>RT: Stream assistant deltas
        RT-->>GW: event:assistant (stream)

        opt Tool Call
            LLM-->>RT: tool_use block
            RT-->>GW: event:tool (start)
            RT->>TL: Policy check + execute
            TL-->>RT: Tool result
            RT-->>GW: event:tool (end)
            RT->>LLM: tool_result
        end

        opt Context Overflow
            RT->>RT: Auto-compaction (summarize + retry)
            RT-->>GW: event:compaction
        end

        opt Auth Failure
            RT->>RT: Failover to next auth profile
        end
    end

    RT->>RT: Assemble final payloads
    RT-->>GW: event:lifecycle (end)
    GW->>CH: Deliver reply
```

## Run Lifecycle Events

| Phase | Stream | Description |
|-------|--------|-------------|
| Accepted | lifecycle | Run ID assigned, queued for execution |
| Start | lifecycle | Session lock acquired, model resolved, system prompt built |
| Inference | assistant | LLM generates text/reasoning tokens streamed as deltas |
| Tool Call | tool | Model requests a tool; tool executes and returns result |
| Compaction | compaction | Context overflow triggers summarize-and-retry |
| End | lifecycle | Run completes; payloads assembled |
| Error | lifecycle | Run errors; error text assembled |

## Queueing and Concurrency

```mermaid
graph LR
    subgraph Lanes["Lane System"]
        SL[Session Lane<br/>1 run per session key]
        GL[Global Lane<br/>Optional cross-session throttle]
        SAL[Subagent Lane<br/>Isolated from main queue]
    end

    M1[Message 1] --> SL
    M2[Message 2] --> SL
    M3[Sub-agent task] --> SAL
    SL --> GL
    SAL --> GL
    GL --> RT[Agent Runtime]
```

- **Session Lane:** One run at a time per session key. Prevents tool/session transcript races.
- **Global Lane:** Optional cross-session throttle for resource management.
- **Subagent Lane:** Sub-agents use `CommandLane.Subagent`, isolated from main session queue.

**Source:** `src/agents/lanes.ts`, `src/process/lanes.ts`, `src/process/command-queue.ts`

## Reply Shaping and Suppression

Final payloads are assembled from:
- Assistant text (and optional reasoning)
- Inline tool summaries (when verbose + allowed)
- Assistant error text when the model errors

Special handling:
- `NO_REPLY` token is filtered from outgoing payloads (silent token)
- Messaging tool duplicates are removed from the final payload list
- If no renderable payloads remain and a tool errored, a fallback error reply is emitted

**Source:** `src/agents/pi-embedded-subscribe.handlers.messages.ts`

## Timeouts

| Timeout | Default | Description |
|---------|---------|-------------|
| Agent run | 600s | `agents.defaults.timeoutSeconds`; enforced via abort timer |
| `agent.wait` | 30s | Wait-only; does not stop the agent run |
| Sub-agent | 0 (none) | Sub-agents are long-lived by default |

**Source:** `src/agents/timeout.ts`, `src/agents/pi-embedded-runner/run.ts`
