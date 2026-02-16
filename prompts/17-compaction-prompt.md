# Compaction Merge Instruction

**Source:** `src/agents/compaction.ts`

## Merge Instruction

Used when merging partial summaries during multi-stage compaction:

```
Merge these partial summaries into a single cohesive summary.
Preserve decisions, TODOs, open questions, and any constraints.
```

## Compaction Flow

1. `estimateMessagesTokens()` — count tokens in session
2. `splitMessagesByTokenShare()` — divide into chunks (BASE_CHUNK_RATIO=0.4)
3. `generateSummary()` — LLM summarizes each chunk
4. Merge summaries using the instruction above
5. `repairToolUseResultPairing()` — fix orphaned pairs
6. Replace session transcript

## Functions

| Function | Purpose |
|----------|---------|
| `summarizeChunks()` | Summarizes message chunks |
| `summarizeWithFallback()` | Progressive fallback for oversized messages |
| `summarizeInStages()` | Multi-stage summarization with merging |
