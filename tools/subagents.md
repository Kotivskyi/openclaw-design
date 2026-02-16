# subagents

**Source:** `src/agents/tools/subagents-tool.ts`
**Group:** `group:sessions`
**Mutation:** Depends on action

## Description

List, steer, or kill sub-agent runs for this requester session.

## Actions (`SUBAGENT_ACTIONS`)

| Action | Mutating | Description |
|--------|----------|-------------|
| `list` | No | Show active and recently completed sub-agents |
| `kill` | Yes | Terminate a sub-agent (cascading to descendants) |
| `steer` | Yes | Redirect an active sub-agent mid-execution |

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | `list`, `kill`, or `steer` |
| `target` | string | No | Sub-agent index, label, or session key (for kill/steer) |
| `message` | string | No | New message for steer action |

## Kill Flow

1. Resolve target sub-agent
2. Cascade kill all descendants recursively (`cascadeKillChildren()`)
3. Abort embedded run
4. Clear session queues
5. Mark run terminated in registry
6. Update session store (`abortedLastRun=true`)

## Steer Flow

1. Rate limit check (2s between steers)
2. Suppress announce for interrupted run
3. Abort current run via AbortSignal
4. Clear pending queue items
5. Wait for settle (up to 5s)
6. Dispatch new run with steer message
7. Replace run record in registry

## Returns

- `list`: Formatted list of active/recent sub-agents with status, model, runtime, tokens
- `kill`: `{status: "killed"}`
- `steer`: `{status: "accepted", mode: "restart"}`
