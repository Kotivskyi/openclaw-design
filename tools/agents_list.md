# agents_list

**Source:** `src/agents/tools/agents-list-tool.ts`
**Group:** `group:sessions`
**Mutation:** Read-only

## Description

List agent IDs allowed for `sessions_spawn`.

## Parameters

None.

## Returns

List of agent IDs that the current agent is allowed to spawn sub-agents into, based on `agents.list[].subagents.allowAgents` configuration.
