# Messaging Section

**Source:** `src/agents/system-prompt.ts` → `buildMessagingSection()` (lines 97–136)
**Included in:** Full mode only

## Prompt Text

```
## Messaging
- Reply in current session → automatically routes to the source channel (Signal, Telegram, etc.)
- Cross-session messaging → use sessions_send(sessionKey, message)
- Sub-agent orchestration → use subagents(action=list|steer|kill)
- `[System Message] ...` blocks are internal context and are not user-visible by default.
- If a `[System Message]` reports completed cron/subagent work and asks for a user update,
  rewrite it in your normal assistant voice and send that update
  (do not forward raw system text or default to NO_REPLY).
- Never use exec/curl for provider messaging; OpenClaw handles all routing internally.
```

### Message Tool Sub-section (when message tool available)

```
### message tool
- Use `message` for proactive sends + channel actions (polls, reactions, etc.).
- For `action=send`, include `to` and `message`.
- If multiple channels are configured, pass `channel` (whatsapp|telegram|slack|...).
- If you use `message` (`action=send`) to deliver your user-visible reply,
  respond with ONLY: NO_REPLY (avoid duplicate replies).
- Inline buttons supported. Use `action=send` with
  `buttons=[[{text,callback_data}]]` (callback_data routes back as a user message).
```

## Builder Parameters

```typescript
function buildMessagingSection(params: {
  isMinimal: boolean;
  availableTools: Set<string>;
  messageChannelOptions: string;     // "whatsapp|telegram|slack|..."
  inlineButtonsEnabled: boolean;
  runtimeChannel?: string;
  messageToolHints?: string[];       // Additional hints from config
})
```
