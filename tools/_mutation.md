# Tool Mutation Safety Categories

**Source:** `src/agents/tool-mutation.ts`

## Mutating Tools (`MUTATING_TOOL_NAMES`)

Tools that modify state when called:

| Tool | Mutating Actions |
|------|-----------------|
| `write` | Always mutating |
| `edit` | Always mutating |
| `apply_patch` | Always mutating |
| `exec` | Always mutating |
| `message` | `send`, `edit`, `unsend`, `react` |
| `sessions_send` | Always mutating |
| `sessions_spawn` | Always mutating |
| `cron` | `add`, `remove`, `enable`, `disable` |
| `gateway` | `config.apply`, `update.run`, `restart` |
| `canvas` | `set`, `eval`, `reset` |
| `subagents` | `kill`, `steer` |
| `browser` | Navigation, clicks, input |
| `nodes` | `notify`, `camera`, `screen` |

## Read-Only Actions (`READ_ONLY_ACTIONS`)

Actions that never mutate state:

| Action | Tool |
|--------|------|
| `list` | subagents, cron, sessions_list |
| `get` | cron, memory_get |
| `search` | memory_search |
| `status` | session_status, gateway |
| `snapshot` | canvas |
| `history` | sessions_history |
| `describe` | nodes |
| `config.get` | gateway |
| `config.schema` | gateway |
| `poll` | process |
| `info` | process |
| `screenshot` | browser |

## Key Functions

| Function | Purpose |
|----------|---------|
| `isMutatingToolCall(name, input)` | Check if a specific tool call is mutating |
| `buildToolActionFingerprint(name, input)` | Create a fingerprint for deduplication |
| `buildToolMutationState()` | Track mutation state across a run |
