# process

**Source:** `src/agents/bash-tools.process.ts`
**Group:** `group:runtime`
**Mutation:** Depends on action

## Description

Manage background exec sessions.

## Actions

| Action | Mutating | Description |
|--------|----------|-------------|
| `poll` | No | Check output of a background exec session |
| `info` | No | Get info about an exec session |
| `kill` | Yes | Terminate a background exec session |
| `send` | Yes | Send input to a running exec session |

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | `poll`, `info`, `kill`, or `send` |
| `sessionId` | string | Yes | Background exec session ID |
| `timeout` | number | No | Poll timeout in milliseconds |
| `input` | string | No | Input text for `send` action |
