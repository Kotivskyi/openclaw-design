# exec

**Source:** `src/agents/bash-tools.exec.ts`
**Group:** `group:runtime`
**Mutation:** Always mutating
**Alias:** `bash`

## Description

Run shell commands (PTY available for TTY-required CLIs).

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `command` | string | Yes | Shell command to execute |
| `yieldMs` | number | No | Yield control after N milliseconds |
| `background` | boolean | No | Run in background (returns session ID) |
| `cwd` | string | No | Working directory override |
| `env` | object | No | Environment variable overrides |

## Notes

- Supports PTY mode for interactive CLIs
- Background mode returns an exec session ID for use with `process` tool
- For long waits, use `yieldMs` or `process(action=poll)`
