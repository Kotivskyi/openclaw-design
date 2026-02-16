# sessions_spawn

**Source:** `src/agents/tools/sessions-spawn-tool.ts`
**Group:** `group:sessions`
**Mutation:** Always mutating

## Description

Spawn a sub-agent session.

## Schema (`SessionsSpawnToolSchema`)

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `task` | string | Yes | Natural-language task description |
| `label` | string | No | Human-readable name for identification |
| `agentId` | string | No | Target different agent (subject to allowlist) |
| `model` | string | No | Override model |
| `thinking` | string | No | Override thinking/reasoning level |
| `runTimeoutSeconds` | number | No | Per-run timeout (default: 0 = no timeout) |
| `cleanup` | `"delete"` \| `"keep"` | No | Auto-cleanup session after completion |

## Spawn Flow

1. Validate spawn depth (`callerDepth < maxSpawnDepth`)
2. Validate active children count (`< maxChildrenPerAgent`)
3. Validate target agentId (allowAgents check)
4. Generate child session key: `agent:<targetAgentId>:subagent:<uuid>`
5. Patch session store with depth, model, thinking overrides
6. Build subagent system prompt (see prompts/10-subagent-system-prompt.md)
7. Dispatch via gateway agent RPC on Subagent lane
8. Register in subagent registry

## Returns

```json
{ "status": "accepted", "childSessionKey": "...", "runId": "..." }
```

## Limits

| Limit | Config Path | Default |
|-------|-------------|---------|
| Max spawn depth | `agents.defaults.subagents.maxSpawnDepth` | 1 |
| Max children per agent | `agents.defaults.subagents.maxChildrenPerAgent` | 5 |
| Allowed cross-agent IDs | `agents.list[].subagents.allowAgents` | [] |
