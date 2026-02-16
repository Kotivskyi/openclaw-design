# session_status

**Source:** `src/agents/tools/session-status-tool.ts`
**Group:** `group:sessions`
**Mutation:** Read-only

## Description

Show a /status-equivalent status card (usage + time + Reasoning/Verbose/Elevated); use for model-use questions; optional per-session model override.

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `model` | string | No | Optional per-session model override |

## System Prompt Hint

```
session_status: Show a /status-equivalent status card
(usage + time + Reasoning/Verbose/Elevated);
use for model-use questions (📊 session_status);
optional per-session model override
```
