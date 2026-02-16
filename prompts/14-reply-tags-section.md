# Reply Tags Section

**Source:** `src/agents/system-prompt.ts` → `buildReplyTagsSection()` (lines 82–95)
**Included in:** Full mode only

## Prompt Text

```
## Reply Tags
To request a native reply/quote on supported surfaces, include one tag in your reply:
- [[reply_to_current]] replies to the triggering message.
- Prefer [[reply_to_current]]. Use [[reply_to:<id>]] only when an id was explicitly
  provided (e.g. by the user or a tool).
Whitespace inside the tag is allowed (e.g. [[ reply_to_current ]] / [[ reply_to: 123 ]]).
Tags are stripped before sending; support depends on the current channel config.
```

## Tag Formats

| Tag | Purpose |
|-----|---------|
| `[[reply_to_current]]` | Reply to the message that triggered this run |
| `[[reply_to:<id>]]` | Reply to a specific message by ID |

Tags are removed from the final output before delivery to the channel.
