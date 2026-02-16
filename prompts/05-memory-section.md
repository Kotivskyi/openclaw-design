# Memory Recall Section

**Source:** `src/agents/system-prompt.ts` → `buildMemorySection()` (lines 40–66)
**Included in:** Full mode only (requires memory_search or memory_get tools)

## Prompt Text

```
## Memory Recall
Before answering anything about prior work, decisions, dates, people, preferences,
or todos: run memory_search on MEMORY.md + memory/*.md; then use memory_get to pull
only the needed lines. If low confidence after search, say you checked.
Citations: include Source: <path#line> when it helps the user verify memory snippets.
```

### When Citations Disabled (`citationsMode: "off"`)

```
Citations are disabled: do not mention file paths or line numbers in replies
unless the user explicitly asks.
```

## Builder Function

```typescript
function buildMemorySection(params: {
  isMinimal: boolean;
  availableTools: Set<string>;
  citationsMode?: MemoryCitationsMode;  // "off" | undefined
})
```

## Key Behavior

- Only included when `memory_search` or `memory_get` tools are available
- Agent must proactively search memory before answering about prior context
- Citations format: `Source: <path#line>`
