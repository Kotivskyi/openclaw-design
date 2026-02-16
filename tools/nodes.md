# nodes

**Source:** `src/agents/tools/nodes-tool.ts`
**Group:** `group:nodes`
**Mutation:** Depends on action

## Description

List/describe/notify/camera/screen on paired nodes.

## Actions

| Action | Mutating | Description |
|--------|----------|-------------|
| `list` | No | List connected nodes |
| `describe` | No | Get node capabilities |
| `notify` | Yes | Send notification to a node |
| `camera` | Yes | Take a photo via node camera |
| `screen` | Yes | Record screen on a node |

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | Action to perform |
| `nodeId` | string | No | Target node ID |
| `text` | string | No | Notification text |

## Notes

Nodes are macOS/iOS/Android/headless devices that connect with `role: node`:
- Provide device identity in `connect`
- Expose commands: `canvas.*`, `camera.*`, `screen.record`, `location.get`
