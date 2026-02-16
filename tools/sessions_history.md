# sessions_history

**Source:** `src/agents/tools/sessions-history-tool.ts`
**Group:** `group:sessions`
**Mutation:** Read-only

## Description

Fetch history for another session/sub-agent.

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `sessionKey` | string | Yes | Target session key |
| `last` | number | No | Return only the N most recent messages |
