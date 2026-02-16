# OpenClaw Tools Reference

This folder documents all tools available in the OpenClaw agent runtime.

## Index by Group

### File System (`group:fs`)
| Tool | File |
|------|------|
| [read](read.md) | pi-agent-core built-in |
| [write](write.md) | pi-agent-core built-in |
| [edit](edit.md) | pi-agent-core built-in |
| [apply_patch](apply_patch.md) | `src/agents/apply-patch.ts` |
| [grep](grep.md) | pi-agent-core built-in |
| [find](find.md) | pi-agent-core built-in |
| [ls](ls.md) | pi-agent-core built-in |

### Runtime (`group:runtime`)
| Tool | File |
|------|------|
| [exec](exec.md) | `src/agents/bash-tools.exec.ts` |
| [process](process.md) | `src/agents/bash-tools.process.ts` |

### Web (`group:web`)
| Tool | File |
|------|------|
| [web_search](web_search.md) | `src/agents/tools/web-search.ts` |
| [web_fetch](web_fetch.md) | `src/agents/tools/web-fetch.ts` |

### UI (`group:ui`)
| Tool | File |
|------|------|
| [browser](browser.md) | `src/agents/tools/browser-tool.ts` |
| [canvas](canvas.md) | `src/agents/tools/canvas-tool.ts` |

### Sessions (`group:sessions`)
| Tool | File |
|------|------|
| [agents_list](agents_list.md) | `src/agents/tools/agents-list-tool.ts` |
| [sessions_list](sessions_list.md) | `src/agents/tools/sessions-list-tool.ts` |
| [sessions_history](sessions_history.md) | `src/agents/tools/sessions-history-tool.ts` |
| [sessions_send](sessions_send.md) | `src/agents/tools/sessions-send-tool.ts` |
| [sessions_spawn](sessions_spawn.md) | `src/agents/tools/sessions-spawn-tool.ts` |
| [subagents](subagents.md) | `src/agents/tools/subagents-tool.ts` |
| [session_status](session_status.md) | `src/agents/tools/session-status-tool.ts` |

### Automation (`group:automation`)
| Tool | File |
|------|------|
| [cron](cron.md) | `src/agents/tools/cron-tool.ts` |
| [gateway](gateway.md) | `src/agents/tools/gateway-tool.ts` |

### Messaging (`group:messaging`)
| Tool | File |
|------|------|
| [message](message.md) | `src/agents/tools/message-tool.ts` |

### Other
| Tool | File |
|------|------|
| [nodes](nodes.md) | `src/agents/tools/nodes-tool.ts` |
| [image](image.md) | `src/agents/tools/image-tool.ts` |
| [tts](tts.md) | `src/agents/tools/tts-tool.ts` |
| [memory_search](memory_search.md) | `src/agents/tools/memory-tool.ts` |
| [memory_get](memory_get.md) | `src/agents/tools/memory-tool.ts` |

## Tool Policy

See [_policy.md](_policy.md) for profiles, groups, aliases, and owner-only restrictions.

## Tool Mutation

See [_mutation.md](_mutation.md) for mutation safety categories.
