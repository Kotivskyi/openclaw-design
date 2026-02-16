# Tooling Section

**Source:** `src/agents/system-prompt.ts` (lines 223–431)
**Included in:** Full mode, Minimal mode

## Prompt Structure

```
## Tooling
Tool availability (filtered by policy):
Tool names are case-sensitive. Call tools exactly as listed.
- read: Read file contents
- write: Create or overwrite files
- edit: Make precise edits to files
...
TOOLS.md does not control tool availability; it is user guidance for how to use external tools.
If a task is more complex or takes longer, spawn a sub-agent. Completion is push-based.
Do not poll `subagents list` / `sessions_list` in a loop.
```

## Core Tool Summaries (`coreToolSummaries`)

These are the hardcoded tool descriptions included in the system prompt:

| Tool | Summary |
|------|---------|
| `read` | Read file contents |
| `write` | Create or overwrite files |
| `edit` | Make precise edits to files |
| `apply_patch` | Apply multi-file patches |
| `grep` | Search file contents for patterns |
| `find` | Find files by glob pattern |
| `ls` | List directory contents |
| `exec` | Run shell commands (pty available for TTY-required CLIs) |
| `process` | Manage background exec sessions |
| `web_search` | Search the web (Brave API) |
| `web_fetch` | Fetch and extract readable content from a URL |
| `browser` | Control web browser |
| `canvas` | Present/eval/snapshot the Canvas |
| `nodes` | List/describe/notify/camera/screen on paired nodes |
| `cron` | Manage cron jobs and wake events (use for reminders) |
| `message` | Send messages and channel actions |
| `gateway` | Restart, apply config, or run updates on the running OpenClaw process |
| `agents_list` | List agent ids allowed for sessions_spawn |
| `sessions_list` | List other sessions (incl. sub-agents) with filters/last |
| `sessions_history` | Fetch history for another session/sub-agent |
| `sessions_send` | Send a message to another session/sub-agent |
| `sessions_spawn` | Spawn a sub-agent session |
| `subagents` | List, steer, or kill sub-agent runs for this requester session |
| `session_status` | Show usage/time/model state |
| `image` | Analyze an image with the configured image model |

## Tool Call Style Section

```
## Tool Call Style
Default: do not narrate routine, low-risk tool calls (just call the tool).
Narrate only when it helps: multi-step work, complex/challenging problems,
sensitive actions (e.g., deletions), or when the user explicitly asks.
Keep narration brief and value-dense; avoid repeating obvious steps.
Use plain human language for narration unless in a technical context.
```
