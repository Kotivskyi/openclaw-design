# memory_search

**Source:** `src/agents/tools/memory-tool.ts` (plugin-provided via `src/plugins/runtime/index.ts`)
**Group:** `group:memory`
**Mutation:** Read-only

## Description

Search memory via vector embeddings + BM25 hybrid search.

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `query` | string | Yes | Search query |
| `paths` | string[] | No | Restrict search to specific paths |

## Search Pipeline

1. Sync memory files if needed (chunk + embed new content)
2. Vector search (cosine similarity, weight 0.7)
3. Text search (BM25, weight 0.3)
4. Merge results with weighted combination
5. Rank by combined score
6. Filter by minScore (0.35)
7. Limit to maxResults (6)
8. Add citations: `Source: <path#line>`

## Memory Sources

- `MEMORY.md` (bootstrap, always loaded)
- `memory/*.md` (daily memory files)
- Session transcripts (when `experimental.sessionMemory` enabled)
