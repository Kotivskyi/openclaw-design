# Runtime Metadata Line

**Source:** `src/agents/system-prompt-params.ts` → `buildRuntimeLine()`

## Format

```
Runtime: agent=[id] host=[name] repo=[root] os=[name] (arch) node=[version]
model=[current] default_model=[fallback] shell=[type] channel=[name]
capabilities=[list] thinking=[level]
```

## Parameters

```typescript
buildRuntimeLine(
  runtimeInfo: {
    agentId?: string;
    host?: string;
    os?: string;
    arch?: string;
    node?: string;
    model?: string;
    defaultModel?: string;
    shell?: string;
    channel?: string;
    capabilities?: string[];
    repoRoot?: string;
  },
  runtimeChannel?: string,
  runtimeCapabilities?: string[],
  defaultThinkLevel?: ThinkLevel,
)
```

## Example Output

```
Runtime: agent=main host=macbook repo=/Users/alice/.openclaw/workspace
os=darwin (arm64) node=v20.11.1 model=claude-sonnet-4-5-20250929
default_model=claude-sonnet-4-5-20250929 shell=zsh channel=whatsapp
capabilities=inlineButtons thinking=off
```

## Builder

**Function:** `buildSystemPromptParams()` in `src/agents/system-prompt-params.ts` resolves all runtime parameters including timezone, repo root, agent ID, etc.
