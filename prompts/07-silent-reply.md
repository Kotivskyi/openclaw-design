# Silent Reply Token (NO_REPLY)

**Source:** `src/auto-reply/tokens.ts`
**Constant:** `SILENT_REPLY_TOKEN = "NO_REPLY"`
**System Prompt Section:** `src/agents/system-prompt.ts`

## Token Value

```
NO_REPLY
```

## System Prompt Instructions

```
## Silent Replies
When you have nothing to say, respond with ONLY: NO_REPLY

⚠️ Rules:
- It must be your ENTIRE message — nothing else
- Never append it to an actual response (never include "NO_REPLY" in real replies)
- Never wrap it in markdown or code blocks

❌ Wrong: "Here's help... NO_REPLY"
✅ Right: NO_REPLY
```

## Detection Logic

**File:** `src/auto-reply/tokens.ts` → `isSilentReplyText(text, token)`

The function checks if the text starts with or ends with `NO_REPLY` using regex:
- Prefix match: `^\s*NO_REPLY(?=$|\W)`
- Suffix match: `\bNO_REPLY\b\W*$`

## Usage Contexts

1. When agent has nothing to say
2. After using `message(action=send)` to deliver reply (avoids duplicate)
3. When a subagent announce was already delivered
4. When heartbeat has nothing to report (uses `HEARTBEAT_OK` instead)
