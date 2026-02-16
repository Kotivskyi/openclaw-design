# message

**Source:** `src/agents/tools/message-tool.ts`
**Group:** `group:messaging`
**Mutation:** Depends on action

## Description

Send messages and channel actions (polls, reactions, etc.).

## Actions

| Action | Mutating | Description |
|--------|----------|-------------|
| `send` | Yes | Send a message to a channel |
| `react` | Yes | React to a message with an emoji |
| `edit` | Yes | Edit a previously sent message |
| `unsend` | Yes | Delete a previously sent message |

## Parameters (send)

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | `send` |
| `to` | string | Yes | Recipient identifier |
| `message` | string | Yes | Message text |
| `channel` | string | No | Channel (whatsapp, telegram, slack, etc.) |
| `buttons` | array | No | Inline buttons `[{text, callback_data}]` |
| `media` | object | No | Media attachment |

## Notes

- When using `message(action=send)` to deliver the user-visible reply, respond with `NO_REPLY` to avoid duplicate delivery
- Channel-specific actions may vary (see per-channel action files)
- Channel-specific action files: `whatsapp-actions.ts`, `telegram-actions.ts`, `slack-actions.ts`, `discord-actions.ts`
