# browser

**Source:** `src/agents/tools/browser-tool.ts`
**Group:** `group:ui`
**Mutation:** Depends on action

## Description

Control web browser (Playwright/CDP).

## Actions

| Action | Mutating | Description |
|--------|----------|-------------|
| `navigate` | Yes | Navigate to a URL |
| `click` | Yes | Click an element |
| `type` | Yes | Type text into an element |
| `screenshot` | No | Take a screenshot |
| `scroll` | Yes | Scroll the page |
| `select` | Yes | Select an option from a dropdown |
| `evaluate` | Yes | Execute JavaScript in the page |

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | Action to perform |
| `url` | string | No | URL for navigate action |
| `selector` | string | No | CSS selector for element actions |
| `text` | string | No | Text for type action |
| `script` | string | No | JavaScript for evaluate action |

## Notes

- Uses Playwright or CDP (Chrome DevTools Protocol)
- In sandbox mode, uses a sandboxed browser instance
- Bridge URL support for cross-container browser control
