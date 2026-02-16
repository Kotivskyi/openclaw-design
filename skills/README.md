# OpenClaw Skills Reference

This folder documents all 60 skills available in the OpenClaw agent runtime.

## Bundled Skills (51)

| Skill | Emoji | Description | OS | Requires |
|-------|-------|-------------|-----|----------|
| [1password](1password.md) | 🔐 | 1Password CLI (op) | All | `op` |
| [apple-notes](apple-notes.md) | 📝 | Apple Notes via `memo` CLI | macOS | `memo` |
| [apple-reminders](apple-reminders.md) | ⏰ | Apple Reminders via `remindctl` | macOS | `remindctl` |
| [bear-notes](bear-notes.md) | 🐻 | Bear notes via grizzly CLI | macOS | `grizzly` |
| [blogwatcher](blogwatcher.md) | 📰 | Monitor blogs/RSS feeds | All | `blogwatcher` |
| [blucli](blucli.md) | 🫐 | BluOS speaker control | All | `blu` |
| [bluebubbles](bluebubbles.md) | 🫧 | iMessage via BlueBubbles | All | — |
| [camsnap](camsnap.md) | 📸 | RTSP/ONVIF camera capture | All | `camsnap` |
| [canvas](canvas.md) | — | Display HTML on OpenClaw nodes | All | — |
| [clawhub](clawhub.md) | — | ClawHub skill marketplace CLI | All | `clawhub` |
| [coding-agent](coding-agent.md) | 🧩 | Run coding agents (Codex, Claude Code) | All | — |
| [discord](discord.md) | 🎮 | Discord operations | All | — |
| [eightctl](eightctl.md) | 🎛️ | Eight Sleep pod control | All | `eightctl` |
| [food-order](food-order.md) | 🥡 | Reorder food via ordercli | All | `ordercli` |
| [gemini](gemini.md) | ♊️ | Gemini CLI for Q&A | All | `gemini` |
| [gifgrep](gifgrep.md) | 🧲 | Search GIF providers | All | `gifgrep` |
| [github](github.md) | 🐙 | GitHub via `gh` CLI | All | `gh` |
| [gog](gog.md) | 🎮 | Google Workspace CLI | All | `gog` |
| [goplaces](goplaces.md) | 📍 | Google Places API | All | `goplaces` |
| [healthcheck](healthcheck.md) | — | Host security hardening | All | — |
| [himalaya](himalaya.md) | 📧 | Email via IMAP/SMTP | All | `himalaya` |
| [imsg](imsg.md) | 📨 | iMessage/SMS CLI | macOS | `imsg` |
| [mcporter](mcporter.md) | 📦 | MCP server/tool CLI | All | `mcporter` |
| [model-usage](model-usage.md) | 📊 | Per-model usage/cost data | macOS | `codexbar` |
| [nano-banana-pro](nano-banana-pro.md) | 🍌 | Gemini 3 image generation | All | `uv` |
| [nano-pdf](nano-pdf.md) | 📄 | PDF editing via natural language | All | `nano-pdf` |
| [notion](notion.md) | 📝 | Notion API integration | All | `NOTION_API_KEY` |
| [obsidian](obsidian.md) | 💎 | Obsidian vault management | All | `obsidian-cli` |
| [openai-image-gen](openai-image-gen.md) | 🖼️ | OpenAI image generation | All | `python3` |
| [openai-whisper](openai-whisper.md) | 🎙️ | Local speech-to-text (Whisper) | All | `whisper` |
| [openai-whisper-api](openai-whisper-api.md) | ☁️ | OpenAI Whisper API | All | `curl` |
| [openhue](openhue.md) | 💡 | Philips Hue light control | All | `openhue` |
| [oracle](oracle.md) | 🧿 | Oracle CLI for prompting | All | `oracle` |
| [ordercli](ordercli.md) | 🛵 | Foodora order status CLI | All | `ordercli` |
| [peekaboo](peekaboo.md) | 👀 | macOS UI automation | macOS | `peekaboo` |
| [sag](sag.md) | 🗣️ | ElevenLabs TTS | All | `sag` |
| [session-logs](session-logs.md) | 📜 | Search session logs | All | `jq`, `rg` |
| [sherpa-onnx-tts](sherpa-onnx-tts.md) | 🗣️ | Local TTS (sherpa-onnx) | macOS/Linux/Win | env vars |
| [skill-creator](skill-creator.md) | — | Create/update agent skills | All | — |
| [slack](slack.md) | 💬 | Slack operations | All | — |
| [songsee](songsee.md) | 🌊 | Audio spectrogram visualizer | All | `songsee` |
| [sonoscli](sonoscli.md) | 🔊 | Sonos speaker control | All | `sonos` |
| [spotify-player](spotify-player.md) | 🎵 | Spotify playback/search | All | `spogo` |
| [summarize](summarize.md) | 🧾 | Summarize URLs/podcasts/files | All | `summarize` |
| [things-mac](things-mac.md) | ✅ | Things 3 task manager | macOS | `things` |
| [tmux](tmux.md) | 🧵 | Tmux session control | macOS/Linux | `tmux` |
| [trello](trello.md) | 📋 | Trello board management | All | `jq` |
| [video-frames](video-frames.md) | 🎞️ | Extract video frames | All | `ffmpeg` |
| [voice-call](voice-call.md) | 📞 | Voice calls | All | — |
| [wacli](wacli.md) | 📱 | WhatsApp CLI | All | `wacli` |
| [weather](weather.md) | 🌤️ | Weather forecasts | All | `curl` |

## Extension Skills (5)

| Skill | Extension | Description |
|-------|-----------|-------------|
| [feishu-doc](feishu-doc.md) | feishu | Feishu document read/write |
| [feishu-drive](feishu-drive.md) | feishu | Feishu cloud storage |
| [feishu-perm](feishu-perm.md) | feishu | Feishu permission management |
| [feishu-wiki](feishu-wiki.md) | feishu | Feishu knowledge base |
| [prose](prose.md) | open-prose | Multi-agent workflows |

## Agent Skills (4)

| Skill | Description |
|-------|-------------|
| [review-pr](review-pr.md) | PR review workflow |
| [prepare-pr](prepare-pr.md) | PR preparation workflow |
| [merge-pr](merge-pr.md) | PR merge workflow |
| [mintlify](mintlify.md) | Documentation site management |
