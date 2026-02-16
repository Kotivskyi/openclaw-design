# Complete File Reference Index

## Agent Prompts

| File | Purpose |
|------|---------|
| `src/agents/system-prompt.ts` | Main system prompt builder (`buildAgentSystemPrompt`) |
| `src/agents/system-prompt-params.ts` | Runtime parameter resolution for system prompt |
| `src/agents/pi-embedded-runner/system-prompt.ts` | Embedded runner system prompt wrapper (`buildEmbeddedSystemPrompt`) |
| `src/agents/subagent-announce.ts` | Sub-agent system prompt builder (`buildSubagentSystemPrompt`) |
| `src/auto-reply/reply/session-reset-prompt.ts` | Session reset greeting prompt (`BARE_SESSION_RESET_PROMPT`) |
| `src/agents/system-prompt-report.ts` | System prompt report generation |
| `extensions/open-prose/skills/prose/guidance/system-prompt.md` | Prose writing system prompt |
| `vendor/a2ui/specification/0.9/eval/src/prompts.ts` | A2UI eval/generation prompts |
| `docs/concepts/system-prompt.md` | System prompt documentation |

## Agent Core (Runtime)

| File | Purpose |
|------|---------|
| `src/agents/pi-embedded-runner/run.ts` | Main agent run orchestration |
| `src/agents/pi-embedded-runner/runs.ts` | Run state management (abort, queue, wait) |
| `src/agents/pi-embedded-runner/model.ts` | Model resolution for embedded runs |
| `src/agents/pi-embedded-runner/history.ts` | Session history management |
| `src/agents/pi-embedded-runner/compact.ts` | Compaction execution |
| `src/agents/pi-embedded-runner/lanes.ts` | Lane resolution for embedded runs |
| `src/agents/pi-embedded-runner/abort.ts` | Run abort handling |
| `src/agents/pi-embedded-runner/extensions.ts` | Pi extension management |
| `src/agents/pi-embedded-runner/extra-params.ts` | Extra parameter resolution |
| `src/agents/pi-embedded-runner/tool-split.ts` | SDK vs OpenClaw tool splitting |
| `src/agents/pi-embedded-runner/tool-result-truncation.ts` | Tool result size limiting |
| `src/agents/pi-embedded-runner/types.ts` | Type definitions |
| `src/agents/pi-embedded-subscribe.ts` | Pi event subscription and bridging |
| `src/agents/pi-embedded-subscribe.handlers.ts` | Stream event handlers (main) |
| `src/agents/pi-embedded-subscribe.handlers.messages.ts` | Assistant message handlers |
| `src/agents/pi-embedded-subscribe.handlers.tools.ts` | Tool event handlers |
| `src/agents/pi-embedded-subscribe.handlers.lifecycle.ts` | Lifecycle event handlers |
| `src/agents/pi-embedded-subscribe.handlers.compaction.ts` | Compaction handlers |

## Sub-Agent System

| File | Purpose |
|------|---------|
| `src/agents/tools/sessions-spawn-tool.ts` | Sub-agent spawn tool |
| `src/agents/tools/subagents-tool.ts` | List/kill/steer sub-agent tool |
| `src/agents/subagent-registry.ts` | In-memory registry of active/recent runs |
| `src/agents/subagent-registry.store.ts` | Disk persistence for registry |
| `src/agents/subagent-announce.ts` | Announce flow (read result, notify parent) |
| `src/agents/subagent-announce-queue.ts` | Queue for serializing announce ops |
| `src/agents/subagent-depth.ts` | Spawn depth resolution |
| `src/agents/announce-idempotency.ts` | Announce deduplication |
| `src/agents/lanes.ts` | Lane constants (Nested, Subagent) |

## Session Tools

| File | Purpose |
|------|---------|
| `src/agents/tools/agents-list-tool.ts` | List available agent IDs |
| `src/agents/tools/sessions-list-tool.ts` | List sessions |
| `src/agents/tools/sessions-history-tool.ts` | Fetch session history |
| `src/agents/tools/sessions-send-tool.ts` | Send to another session |
| `src/agents/tools/sessions-send-tool.a2a.ts` | Agent-to-agent send |
| `src/agents/tools/sessions-send-helpers.ts` | Session send utilities |
| `src/agents/tools/sessions-helpers.ts` | Session key resolution |
| `src/agents/tools/sessions-announce-target.ts` | Announce target resolution |
| `src/agents/tools/session-status-tool.ts` | Session status display |
| `src/agents/tools/agent-step.ts` | Agent step reading |

## Tool Policy

| File | Purpose |
|------|---------|
| `src/agents/openclaw-tools.ts` | Tool factory (creates all tool instances) |
| `src/agents/tool-policy.ts` | Profiles, groups, allow/deny policy |
| `src/agents/tool-policy-pipeline.ts` | Policy evaluation pipeline |
| `src/agents/tool-policy.conformance.ts` | Policy conformance checks |
| `src/agents/tool-mutation.ts` | Tool mutation safety |
| `src/agents/tool-display.ts` | Tool display formatting |
| `src/agents/tool-images.ts` | Tool image handling |
| `src/agents/tool-call-id.ts` | Tool call ID generation |
| `src/agents/tool-summaries.ts` | Tool summary generation |
| `src/agents/pi-tools.ts` | Pi tool integration |
| `src/agents/pi-tools.policy.ts` | Pi tool policy |
| `src/agents/pi-tools.schema.ts` | Pi tool schema |
| `src/agents/pi-tools.read.ts` | Pi tool read operations |
| `src/agents/pi-tools.before-tool-call.ts` | Pre-tool-call hooks |
| `src/agents/pi-tools.abort.ts` | Tool abort handling |

## Other Tools

| File | Purpose |
|------|---------|
| `src/agents/tools/browser-tool.ts` | Browser automation (Playwright/CDP) |
| `src/agents/tools/canvas-tool.ts` | Canvas presentation |
| `src/agents/tools/cron-tool.ts` | Cron/reminder scheduling |
| `src/agents/tools/memory-tool.ts` | Memory search and retrieval |
| `src/agents/tools/message-tool.ts` | Cross-channel messaging |
| `src/agents/tools/web-tools.ts` | Web search and fetch (factory) |
| `src/agents/tools/web-search.ts` | Web search (Brave API) |
| `src/agents/tools/web-fetch.ts` | Web page fetch and extraction |
| `src/agents/tools/image-tool.ts` | Image analysis |
| `src/agents/tools/gateway-tool.ts` | Gateway management |
| `src/agents/tools/nodes-tool.ts` | Paired device control |
| `src/agents/tools/tts-tool.ts` | Text-to-speech |
| `src/agents/bash-tools.ts` | Bash/exec tool factory |
| `src/agents/bash-tools.exec.ts` | Exec implementation |
| `src/agents/bash-tools.process.ts` | Process management |

## Skills System

| File | Purpose |
|------|---------|
| `src/agents/skills.ts` | Skills barrel exports |
| `src/agents/skills/workspace.ts` | Workspace skill loading and prompt building |
| `src/agents/skills/bundled-dir.ts` | Bundled skills directory resolution |
| `src/agents/skills/plugin-skills.ts` | Plugin skill discovery |
| `src/agents/skills/config.ts` | Skill configuration and filtering |
| `src/agents/skills/frontmatter.ts` | SKILL.md frontmatter parsing |
| `src/agents/skills/types.ts` | Skill type definitions |
| `src/agents/skills/serialize.ts` | Skill serialization |
| `src/agents/skills/refresh.ts` | Skill refresh logic |
| `src/agents/skills/env-overrides.ts` | Skill environment overrides |
| `src/agents/skills/bundled-context.ts` | Bundled context loading |
| `src/agents/skills-install.ts` | Skill installation |
| `src/agents/skills-status.ts` | Skill status reporting |

## Memory System

| File | Purpose |
|------|---------|
| `src/memory/index.ts` | Memory barrel exports |
| `src/memory/manager.ts` | Memory manager (core operations) |
| `src/memory/embeddings.ts` | Embedding generation |
| `src/memory/embeddings-openai.ts` | OpenAI embeddings |
| `src/memory/embeddings-gemini.ts` | Gemini embeddings |
| `src/memory/embeddings-voyage.ts` | Voyage embeddings |
| `src/memory/hybrid.ts` | Hybrid search (vector + BM25) |
| `src/memory/search-manager.ts` | Search orchestration |
| `src/memory/sqlite.ts` | SQLite storage |
| `src/memory/sqlite-vec.ts` | SQLite vector extension |
| `src/memory/sync-memory-files.ts` | Memory file sync |
| `src/memory/sync-session-files.ts` | Session file sync |
| `src/agents/memory-search.ts` | Memory search config resolution |
| `src/agents/tools/memory-tool.ts` | Memory tools (search + get) |

## Context Management

| File | Purpose |
|------|---------|
| `src/agents/compaction.ts` | Compaction algorithm |
| `src/agents/context-window-guard.ts` | Context window enforcement |
| `src/agents/context.ts` | Model context window lookup |
| `src/agents/pi-extensions/context-pruning/` | Context pruning extension (5 files) |
| `src/agents/session-transcript-repair.ts` | Transcript repair utilities |

## Auth and Model

| File | Purpose |
|------|---------|
| `src/agents/auth-profiles.ts` | Auth profile management |
| `src/agents/auth-health.ts` | Auth health monitoring |
| `src/agents/model-auth.ts` | API key resolution |
| `src/agents/model-selection.ts` | Model selection logic |
| `src/agents/model-fallback.ts` | Model fallback logic |
| `src/agents/model-catalog.ts` | Model catalog |
| `src/agents/model-compat.ts` | Model compatibility |
| `src/agents/failover-error.ts` | Failover error handling |
| `src/agents/defaults.ts` | Default model/provider constants |

## Auto-Reply Pipeline

| File | Purpose |
|------|---------|
| `src/auto-reply/dispatch.ts` | Message dispatch |
| `src/auto-reply/command-detection.ts` | Command detection |
| `src/auto-reply/commands-registry.ts` | Command registry |
| `src/auto-reply/group-activation.ts` | Group chat activation |
| `src/auto-reply/inbound-debounce.ts` | Message debouncing |
| `src/auto-reply/envelope.ts` | Message envelope construction |
| `src/auto-reply/heartbeat.ts` | Heartbeat handling |
| `src/auto-reply/reply/` | Reply pipeline (dispatcher, queue, history) |

## Cron and Hooks

| File | Purpose |
|------|---------|
| `src/cron/service.ts` | Cron service (scheduler, timer, jobs) |
| `src/cron/isolated-agent.ts` | Isolated agent execution |
| `src/cron/delivery.ts` | Cron delivery routing |
| `src/cron/schedule.ts` | Schedule parsing |
| `src/cron/store.ts` | Persistent cron store |
| `src/hooks/hooks.ts` | Hook system |
| `src/hooks/internal-hooks.ts` | Internal (gateway) hooks |
| `src/hooks/gmail.ts` | Gmail integration |
| `src/hooks/gmail-watcher.ts` | Gmail Pub/Sub watcher |

## Gateway

| File | Purpose |
|------|---------|
| `src/gateway/agent-prompt.ts` | Agent prompt construction for gateway |
| `src/gateway/call.ts` | Gateway RPC call utility |
| `src/gateway/gateway-config-prompts.shared.ts` | Gateway config prompts |

## Configuration and Routing

| File | Purpose |
|------|---------|
| `src/config/config.ts` | Config loading and validation |
| `src/config/sessions.ts` | Session store management |
| `src/routing/session-key.ts` | Session key parsing and normalization |
| `src/agents/agent-scope.ts` | Agent scope and config resolution |
| `src/agents/agent-paths.ts` | Agent directory paths |
| `src/agents/workspace.ts` | Workspace management |
| `src/agents/workspace-dir.ts` | Workspace directory resolution |
| `src/agents/identity.ts` | Agent identity |

## Eval and Test Files

| Test File | What It Tests |
|-----------|---------------|
| `src/agents/system-prompt.e2e.test.ts` | System prompt assembly |
| `src/agents/system-prompt-params.e2e.test.ts` | Prompt param resolution |
| `src/agents/system-prompt-report.test.ts` | Prompt report generation |
| `src/agents/tool-policy.e2e.test.ts` | Tool policy enforcement |
| `src/agents/tool-policy.conformance.e2e.test.ts` | Tool policy conformance |
| `src/agents/tool-policy-pipeline.test.ts` | Policy pipeline logic |
| `src/agents/skills.e2e.test.ts` | Skills loading/filtering |
| `src/agents/skills.resolveskillspromptforrun.e2e.test.ts` | Skills prompt resolution |
| `src/agents/agent-scope.e2e.test.ts` | Agent scope resolution |
| `src/agents/context-window-guard.e2e.test.ts` | Context window guard |
| `src/agents/pi-embedded-runner/run.overflow-compaction.test.ts` | Overflow compaction |
| `src/agents/tools/memory-tool.citations.e2e.test.ts` | Memory citations |
| `src/agents/tools/web-search.e2e.test.ts` | Web search |
| `src/auto-reply/dispatch.test.ts` | Auto-reply dispatch |
| `src/cron/service.*.test.ts` | Cron service (12+ test files) |
| `vendor/a2ui/specification/0.9/eval/src/evaluation_flow.ts` | A2UI eval flow |
| `vendor/a2ui/specification/0.9/eval/src/evaluator.ts` | A2UI evaluator |
| `vendor/a2ui/specification/0.9/eval/src/prompts.ts` | A2UI eval prompts |

## Vitest Configurations

| Config | Purpose |
|--------|---------|
| `vitest.config.ts` | Default test config |
| `vitest.e2e.config.ts` | E2E tests |
| `vitest.unit.config.ts` | Unit tests |
| `vitest.gateway.config.ts` | Gateway tests |
| `vitest.extensions.config.ts` | Extension tests |
| `vitest.live.config.ts` | Live (real API key) tests |

## Documentation

| File | Purpose |
|------|---------|
| `docs/concepts/architecture.md` | Gateway architecture |
| `docs/concepts/agent-loop.md` | Agent loop lifecycle |
| `docs/concepts/agent.md` | Agent concept |
| `docs/concepts/multi-agent.md` | Multi-agent routing |
| `docs/concepts/system-prompt.md` | System prompt |
| `docs/concepts/compaction.md` | Compaction |
| `docs/concepts/memory.md` | Memory system |
| `docs/concepts/session.md` | Sessions |
| `docs/concepts/sessions.md` | Session management |
| `docs/concepts/queue.md` | Command queue |
| `docs/concepts/model-failover.md` | Model failover |
| `docs/automation/cron-jobs.md` | Cron jobs |
| `docs/automation/hooks.md` | Hooks |
| `docs/automation/webhook.md` | Webhooks |
