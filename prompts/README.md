# OpenClaw Prompts & Templates Reference

This folder contains all prompt templates, system prompts, and prompt-building functions found in the OpenClaw codebase.

## Index

| File | Prompt | Source |
|------|--------|--------|
| [01-main-system-prompt.md](01-main-system-prompt.md) | Main agent system prompt | `src/agents/system-prompt.ts` |
| [02-safety-section.md](02-safety-section.md) | Safety guardrails | `src/agents/system-prompt.ts` |
| [03-tooling-section.md](03-tooling-section.md) | Tooling section + tool summaries | `src/agents/system-prompt.ts` |
| [04-skills-section.md](04-skills-section.md) | Skills discovery instructions | `src/agents/system-prompt.ts` |
| [05-memory-section.md](05-memory-section.md) | Memory recall instructions | `src/agents/system-prompt.ts` |
| [06-messaging-section.md](06-messaging-section.md) | Messaging guidelines | `src/agents/system-prompt.ts` |
| [07-silent-reply.md](07-silent-reply.md) | NO_REPLY token rules | `src/auto-reply/tokens.ts` |
| [08-heartbeat-prompt.md](08-heartbeat-prompt.md) | Heartbeat polling prompt | `src/auto-reply/heartbeat.ts` |
| [09-session-reset-prompt.md](09-session-reset-prompt.md) | Session reset / /new greeting | `src/auto-reply/reply/session-reset-prompt.ts` |
| [10-subagent-system-prompt.md](10-subagent-system-prompt.md) | Sub-agent context prompt | `src/agents/subagent-announce.ts` |
| [11-subagent-announce.md](11-subagent-announce.md) | Announce reply instructions | `src/agents/subagent-announce.ts` |
| [12-sandbox-section.md](12-sandbox-section.md) | Sandbox runtime info | `src/agents/system-prompt.ts` |
| [13-reasoning-format.md](13-reasoning-format.md) | Think/final tag format | `src/agents/system-prompt.ts` |
| [14-reply-tags-section.md](14-reply-tags-section.md) | Reply tag instructions | `src/agents/system-prompt.ts` |
| [15-cli-reference.md](15-cli-reference.md) | OpenClaw CLI quick reference | `src/agents/system-prompt.ts` |
| [16-docs-section.md](16-docs-section.md) | Documentation links | `src/agents/system-prompt.ts` |
| [17-compaction-prompt.md](17-compaction-prompt.md) | Compaction merge instruction | `src/agents/compaction.ts` |
| [18-self-update-section.md](18-self-update-section.md) | Self-update guardrails | `src/agents/system-prompt.ts` |
| [19-runtime-line.md](19-runtime-line.md) | Runtime metadata format | `src/agents/system-prompt-params.ts` |
| [20-a2ui-eval-prompts.md](20-a2ui-eval-prompts.md) | A2UI eval test prompts | `vendor/a2ui/specification/0.9/eval/src/prompts.ts` |
