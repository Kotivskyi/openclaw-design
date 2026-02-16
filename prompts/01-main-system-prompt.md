# Main Agent System Prompt

**Source:** `src/agents/system-prompt.ts`
**Function:** `buildAgentSystemPrompt(params)`
**Entry:** `src/agents/pi-embedded-runner/system-prompt.ts` → `buildEmbeddedSystemPrompt()`

## Prompt Modes

| Mode | Description | Sections Included |
|------|-------------|-------------------|
| `"full"` | Default for main agent | All sections |
| `"minimal"` | Used for sub-agents | Tooling, Workspace, Runtime only |
| `"none"` | Bare identity only | Single line: identity |

## Opening Line

```
You are a personal assistant running inside OpenClaw.
```

When `promptMode === "none"`, only this line is returned.

## Full Mode Sections (in order)

1. **Identity** — "You are a personal assistant running inside OpenClaw."
2. **Tooling** — Available tools list with descriptions (see [03-tooling-section.md](03-tooling-section.md))
3. **Tool Call Style** — Narration rules
4. **Safety** — Guardrails (see [02-safety-section.md](02-safety-section.md))
5. **CLI Quick Reference** — OpenClaw subcommands (see [15-cli-reference.md](15-cli-reference.md))
6. **Skills (mandatory)** — Skill discovery (see [04-skills-section.md](04-skills-section.md))
7. **Memory Recall** — Memory search instructions (see [05-memory-section.md](05-memory-section.md))
8. **User Identity** — Owner numbers (optional)
9. **Current Date & Time** — Timezone info (optional)
10. **OpenClaw Self-Update** — Update guardrails (if gateway tool available)
11. **Model Aliases** — Model override aliases (optional)
12. **Workspace** — Working directory guidance
13. **Documentation** — Docs links (see [16-docs-section.md](16-docs-section.md))
14. **Sandbox** — Sandbox info (if enabled, see [12-sandbox-section.md](12-sandbox-section.md))
15. **Context Files** — Injected workspace files (AGENTS.md, TOOLS.md, SOUL.md, etc.)
16. **Reply Tags** — Native reply instructions (see [14-reply-tags-section.md](14-reply-tags-section.md))
17. **Messaging** — Channel routing (see [06-messaging-section.md](06-messaging-section.md))
18. **Voice (TTS)** — TTS hints (optional)
19. **Group Chat Context** — Extra system prompts (optional)
20. **Reactions** — Minimal/extensive reaction mode (optional)
21. **Reasoning Format** — Think/final tags (see [13-reasoning-format.md](13-reasoning-format.md))
22. **Silent Replies** — NO_REPLY rules (see [07-silent-reply.md](07-silent-reply.md))
23. **Heartbeats** — Heartbeat handling (see [08-heartbeat-prompt.md](08-heartbeat-prompt.md))
24. **Runtime** — Runtime metadata line (see [19-runtime-line.md](19-runtime-line.md))

## Minimal Mode Sections

Only these sections are included for sub-agents:

1. Identity
2. Tooling
3. Tool Call Style
4. Safety
5. Workspace
6. Runtime

## Parameters

```typescript
buildAgentSystemPrompt(params: {
  workspaceDir: string;
  defaultThinkLevel?: ThinkLevel;
  reasoningLevel?: ReasoningLevel;
  extraSystemPrompt?: string;
  ownerNumbers?: string[];
  reasoningTagHint?: boolean;
  toolNames?: string[];
  toolSummaries?: Record<string, string>;
  modelAliasLines?: string[];
  userTimezone?: string;
  userTime?: string;
  userTimeFormat?: ResolvedTimeFormat;
  contextFiles?: EmbeddedContextFile[];
  skillsPrompt?: string;
  heartbeatPrompt?: string;
  docsPath?: string;
  workspaceNotes?: string[];
  ttsHint?: string;
  promptMode?: PromptMode;
  runtimeInfo?: { ... };
  messageToolHints?: string[];
  sandboxInfo?: { ... };
  reactionGuidance?: { level: "minimal" | "extensive"; channel: string };
  memoryCitationsMode?: MemoryCitationsMode;
})
```
