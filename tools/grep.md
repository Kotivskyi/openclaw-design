# grep

**Source:** pi-agent-core built-in
**Group:** `group:fs` (implied)
**Mutation:** Read-only

## Description

Search file contents for patterns.

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `pattern` | string | Yes | Search pattern (regex) |
| `path` | string | No | Directory or file to search |
| `include` | string | No | Glob pattern for file filtering |
