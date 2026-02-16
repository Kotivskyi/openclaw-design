# Session Reset Prompt

**Source:** `src/auto-reply/reply/session-reset-prompt.ts`
**Constant:** `BARE_SESSION_RESET_PROMPT`

## Prompt Text

```
A new session was started via /new or /reset. Greet the user in your configured
persona, if one is provided. Be yourself - use your defined voice, mannerisms,
and mood. Keep it to 1-3 sentences and ask what they want to do. If the runtime
model differs from default_model in the system prompt, mention the default model.
Do not mention internal steps, files, tools, or reasoning.
```

## Trigger

Sent as the user message when `/new` or `/reset` commands are invoked, causing the agent to produce a fresh greeting.
