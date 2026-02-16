# Sub-Agent Announce Reply Instructions

**Source:** `src/agents/subagent-announce.ts` → `buildAnnounceReplyInstruction(params)`
**Called by:** `runSubagentAnnounceFlow()`

## Parameters

```typescript
function buildAnnounceReplyInstruction(params: {
  remainingActiveSubagentRuns: number;
  requesterIsSubagent: boolean;
  announceType: SubagentAnnounceType;  // "subagent task" | "cron job"
})
```

## Three Reply Scenarios

### 1. Active sub-agent runs remain (`remainingActiveSubagentRuns > 0`)

```
There are still [N] active subagent [runs/run] for this session.
If they are part of the same workflow, wait for the remaining results
before sending a user update. If they are unrelated, respond normally
using only the result above.
```

### 2. Requester is a sub-agent (`requesterIsSubagent === true`)

```
Convert this completion into a concise internal orchestration update for your
parent agent in your own words. Keep this internal context private (don't mention
system/log/stats/session details or announce type). If this result is duplicate
or no update is needed, reply ONLY: NO_REPLY.
```

### 3. Requester is the main agent (default)

```
A completed [announce type] is ready for user delivery. Convert the result above
into your normal assistant voice and send that user-facing update now. Keep this
internal context private (don't mention system/log/stats/session details or
announce type), and do not copy the system message verbatim. Reply ONLY: NO_REPLY
if this exact result was already delivered to the user in this same turn.
```

## Announce Flow Summary

1. Wait for child run to settle (up to 120s)
2. Read latest assistant reply from child transcript (with retry up to 15s)
3. Build compact stats line (runtime, token counts)
4. Build trigger message: `[System Message] Sub-agent "{label}" completed: {reply} {stats} {instruction}`
5. Attempt to queue/steer into requester session
6. If no active run, send via gateway agent RPC
7. Optionally delete child session (`cleanup: "delete"`)

## Stats Line Format

**Function:** `buildCompactAnnounceStatsLine()`

```
Stats: runtime 2m30s • tokens 12.5k (in 8.2k / out 4.3k)
```
