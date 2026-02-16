# read

**Source:** pi-agent-core built-in (`src/agents/pi-tools.read.ts`)
**Group:** `group:fs`
**Mutation:** Read-only

## Description

Read file contents.

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `file_path` | string | Yes | Absolute or workspace-relative path |
| `offset` | number | No | Line offset to start reading |
| `limit` | number | No | Number of lines to read |

## Alias

None (but the tool name is resolved by `resolveToolName("read")`)
