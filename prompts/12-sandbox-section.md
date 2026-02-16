# Sandbox Section

**Source:** `src/agents/system-prompt.ts` (lines 476–518)
**Included in:** Full mode only (when `sandboxInfo.enabled`)

## Prompt Text

```
## Sandbox
You are running in a sandboxed runtime (tools execute in Docker).
Some tools may be unavailable due to sandbox policy.
Sub-agents stay sandboxed (no elevated/host access).
Need outside-sandbox read/write? Don't spawn; ask first.
Sandbox container workdir: /workspace
Sandbox host workspace: ~/.openclaw/workspace
Agent workspace access: rw (mounted at /workspace)
Sandbox browser: enabled.
Elevated exec is available for this session.
User can toggle with /elevated on|off|ask|full.
You may also send /elevated on|off|ask|full when needed.
Current elevated level: ask (ask runs exec on host with approvals; full auto-approves).
```

## Conditional Lines

| Condition | Line |
|-----------|------|
| `containerWorkspaceDir` set | `Sandbox container workdir: {path}` |
| `workspaceDir` set | `Sandbox host workspace: {path}` |
| `workspaceAccess` set | `Agent workspace access: {ro/rw/none}` |
| `browserBridgeUrl` set | `Sandbox browser: enabled.` |
| `browserNoVncUrl` set | `Sandbox browser observer (noVNC): {url}` |
| `hostBrowserAllowed === true` | `Host browser control: allowed.` |
| `hostBrowserAllowed === false` | `Host browser control: blocked.` |
| `elevated.allowed` | Elevated exec lines with current level |
