# web_search

**Source:** `src/agents/tools/web-search.ts`
**Group:** `group:web`
**Mutation:** Read-only

## Description

Search the web (Brave API).

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `query` | string | Yes | Search query |
| `count` | number | No | Number of results (default: 5) |

## Notes

- Uses Brave Search API
- Requires `BRAVE_API_KEY` configuration
- Returns search results with titles, URLs, and snippets
