# memory_get

**Source:** `src/agents/tools/memory-tool.ts` (plugin-provided via `src/plugins/runtime/index.ts`)
**Group:** `group:memory`
**Mutation:** Read-only

## Description

Retrieve specific lines from memory files.

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | Yes | Memory file path |
| `lines` | string | No | Line range (e.g., "10-20") |

## Usage

Used after `memory_search` to pull specific lines referenced in search results. The workflow is:
1. `memory_search` finds relevant chunks with citations
2. `memory_get` retrieves the exact lines needed
