# canvas

**Source:** `src/agents/tools/canvas-tool.ts`
**Group:** `group:ui`
**Mutation:** Depends on action

## Description

Present/evaluate/snapshot the Canvas (agent-editable HTML/CSS/JS surface).

## Actions

| Action | Mutating | Description |
|--------|----------|-------------|
| `set` | Yes | Set canvas HTML/CSS/JS content |
| `eval` | Yes | Evaluate JavaScript on the canvas |
| `snapshot` | No | Take a screenshot of the canvas |
| `reset` | Yes | Clear the canvas |

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | `set`, `eval`, `snapshot`, or `reset` |
| `html` | string | No | HTML content for `set` |
| `css` | string | No | CSS content for `set` |
| `js` | string | No | JavaScript for `set` or `eval` |

## Notes

- Canvas is served at `/__openclaw__/canvas/` by the gateway HTTP server
- A2UI structured UI also available at `/__openclaw__/a2ui/`
