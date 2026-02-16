# tts

**Source:** `src/agents/tools/tts-tool.ts`
**Group:** None (standalone)
**Mutation:** Read-only (generates audio output)

## Description

Text-to-speech synthesis.

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `text` | string | Yes | Text to synthesize |
| `voice` | string | No | Voice name/ID |

## Notes

- Configured via `agents.defaults.tts` in config
- TTS hints are included in the system prompt Voice section when configured
