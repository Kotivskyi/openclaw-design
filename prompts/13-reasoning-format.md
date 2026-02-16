# Reasoning Format Section

**Source:** `src/agents/system-prompt.ts` (lines 328–339)
**Included in:** Full mode (when `reasoningTagHint` enabled)

## Prompt Text

```
ALL internal reasoning MUST be inside <think>...</think>.
Do not output any analysis outside <think>.
Format every reply as <think>...</think> then <final>...</final>, with no other text.
Only the final user-visible reply may appear inside <final>.
Only text inside <final> is shown to the user; everything else is discarded and never seen by the user.
Example:
<think>Short internal reasoning.</think>
<final>Hey there! What would you like to do next?</final>
```

## When Active

This section is injected when `params.reasoningTagHint === true`, typically when:
- The model supports extended thinking
- Reasoning level is set to something other than `"off"`
- The agent needs structured think/final output separation
