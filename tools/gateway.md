# gateway

**Source:** `src/agents/tools/gateway-tool.ts`
**Group:** `group:automation`
**Mutation:** Depends on action

## Description

Restart, apply config, or run updates on the running OpenClaw process.

## Actions

| Action | Mutating | Description |
|--------|----------|-------------|
| `config.get` | No | Get current configuration |
| `config.schema` | No | Get configuration schema |
| `config.apply` | Yes | Validate + write full config, then restart |
| `update.run` | Yes | Update deps or git, then restart |
| `restart` | Yes | Restart the gateway daemon |
| `status` | No | Get gateway status |

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | Action to perform |
| `config` | object | No | Configuration object for `config.apply` |

## Notes

- Self-update is ONLY allowed when the user explicitly asks
- After restart, OpenClaw pings the last active session automatically
