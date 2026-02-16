# web_fetch

**Source:** `src/agents/tools/web-fetch.ts`
**Group:** `group:web`
**Mutation:** Read-only

## Description

Fetch and extract readable content from a URL.

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | Yes | URL to fetch |
| `selector` | string | No | CSS selector to extract specific content |

## Notes

- Extracts readable text content from web pages
- Handles HTML, JSON, and plain text responses
- Applies content extraction and cleaning
