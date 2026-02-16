# Tool Policy System

**Source:** `src/agents/tool-policy.ts`, `src/agents/tool-policy-pipeline.ts`

## Tool Profiles (`TOOL_PROFILES`)

| Profile | Allowed Tools |
|---------|--------------|
| `minimal` | `session_status` only |
| `coding` | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image` |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send` |
| `full` | Everything |

## Tool Groups (`TOOL_GROUPS`)

| Group | Members |
|-------|---------|
| `group:fs` | read, write, edit, apply_patch |
| `group:runtime` | exec, process |
| `group:web` | web_search, web_fetch |
| `group:sessions` | sessions_list, sessions_history, sessions_send, sessions_spawn, subagents, session_status |
| `group:ui` | browser, canvas |
| `group:automation` | cron, gateway |
| `group:messaging` | message |
| `group:nodes` | nodes |
| `group:memory` | memory_search, memory_get |
| `group:openclaw` | All OpenClaw native tools (excludes provider plugins) |

## Tool Name Aliases (`TOOL_NAME_ALIASES`)

| Alias | Canonical Name |
|-------|---------------|
| `bash` | `exec` |
| `apply-patch` | `apply_patch` |

## Owner-Only Tools (`OWNER_ONLY_TOOL_NAMES`)

```typescript
const OWNER_ONLY_TOOL_NAMES = new Set<string>(["whatsapp_login"]);
```

These tools are restricted to verified owner phone numbers only.

## Per-Agent Configuration

```json
{
  "agents": {
    "list": [{
      "id": "family",
      "tools": {
        "profile": "messaging",
        "allow": ["read", "sessions_list"],
        "deny": ["exec", "write", "browser"]
      }
    }]
  }
}
```

## Policy Pipeline

**Source:** `src/agents/tool-policy-pipeline.ts`

Pipeline steps evaluate in order:
1. Profile check (minimal/coding/messaging/full)
2. Allow list (explicit whitelist)
3. Deny list (explicit blacklist)
4. Owner-only check
5. Plugin allowlist

Each step can allow, deny, or pass through to the next step.
