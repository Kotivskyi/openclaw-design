# Heartbeat Prompt

**Source:** `src/auto-reply/heartbeat.ts`
**Constant:** `HEARTBEAT_PROMPT`

## Prompt Text

```
Read HEARTBEAT.md if it exists (workspace context). Follow it strictly.
Do not infer or repeat old tasks from prior chats.
If nothing needs attention, reply HEARTBEAT_OK.
```

## Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `HEARTBEAT_PROMPT` | (above) | Default prompt when config unset |
| `DEFAULT_HEARTBEAT_EVERY` | `"30m"` | Default frequency |
| `DEFAULT_HEARTBEAT_ACK_MAX_CHARS` | `300` | Max chars for ack reply |
| `HEARTBEAT_TOKEN` | `"HEARTBEAT_OK"` | Ack token (from `tokens.ts`) |

## System Prompt Integration

```
## Heartbeats
[heartbeat prompt text]
If you receive a heartbeat poll (a user message matching the heartbeat prompt above),
and there is nothing that needs attention, reply exactly:
HEARTBEAT_OK
OpenClaw treats a leading/trailing "HEARTBEAT_OK" as a heartbeat ack
(and may discard it).
If something needs attention, do NOT include "HEARTBEAT_OK";
reply with the alert text instead.
```

## "Effectively Empty" Check

**Function:** `isHeartbeatContentEffectivelyEmpty(content)`

HEARTBEAT.md is considered empty if it only contains:
- Whitespace
- Markdown header lines (`# ...`)
- Empty list items (`- [ ]`)

When empty, heartbeat API calls are skipped entirely.

## Custom Prompt

Configurable via `agents.defaults.heartbeat.prompt` in config. Falls back to `HEARTBEAT_PROMPT`.
