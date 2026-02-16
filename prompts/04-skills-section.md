# Skills Section

**Source:** `src/agents/system-prompt.ts` → `buildSkillsSection()` (lines 16–38)
**Included in:** Full mode only (skipped in minimal mode)

## Prompt Text

```
## Skills (mandatory)
Before replying: scan <available_skills> <description> entries.
- If exactly one skill clearly applies: read its SKILL.md at <location> with `read`, then follow it.
- If multiple could apply: choose the most specific one, then read/follow it.
- If none clearly apply: do not read any SKILL.md.
Constraints: never read more than one skill up front; only read after selecting.

<available_skills>
  <skill name="github" description="GitHub integration" location="skills/github/" />
  <skill name="coding-agent" description="Coding assistance" location="skills/coding-agent/" />
  ...
</available_skills>
```

## Builder Function

```typescript
function buildSkillsSection(params: {
  skillsPrompt?: string;     // Pre-built skills XML from workspace.ts
  isMinimal: boolean;         // Skip entirely if minimal
  readToolName: string;       // Resolved read tool name
})
```

## Skills Prompt Construction

The `<available_skills>` XML block is built by `buildWorkspaceSkillsPrompt()` in `src/agents/skills/workspace.ts`. It:

1. Collects skills from all sources (workspace, bundled, plugin, managed, agents)
2. Filters by config, OS, and binary requirements
3. Formats each skill as `<skill name="..." description="..." location="..." />`
4. Returns the complete XML block

## Key Behavior

- Agent scans skill descriptions **before** replying
- Reads **at most one** SKILL.md per turn
- Selects the most specific matching skill
- If none apply, proceeds without loading any skill
