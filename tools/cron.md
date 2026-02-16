# cron

**Source:** `src/agents/tools/cron-tool.ts`
**Group:** `group:automation`
**Mutation:** Depends on action

## Description

Manage cron jobs and wake events (use for reminders).

## Actions

| Action | Mutating | Description |
|--------|----------|-------------|
| `list` | No | List all cron jobs |
| `get` | No | Get details of a specific cron job |
| `add` | Yes | Create a new cron job |
| `remove` | Yes | Delete a cron job |
| `enable` | Yes | Enable a disabled cron job |
| `disable` | Yes | Disable a cron job |

## Parameters (add)

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | `add` |
| `schedule` | string | Yes | Cron expression or relative time |
| `systemEvent` | string | Yes | Event text (reads like a reminder when fired) |
| `label` | string | No | Human-readable label |

## System Prompt Hint

```
cron: Manage cron jobs and wake events (use for reminders; when scheduling a reminder,
write the systemEvent text as something that will read like a reminder when it fires,
and mention that it is a reminder depending on the time gap between setting and firing;
include recent context in reminder text if appropriate)
```
