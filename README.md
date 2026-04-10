# Telegram + Whisper Voice Transcription

Telegram channel plugin for Claude Code with **local speech-to-text** via [whisper.cpp](https://github.com/ggerganov/whisper.cpp).

Based on the [official Telegram plugin](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/telegram), extended with automatic voice message transcription using the Whisper medium model. Voice messages are transcribed locally — no external API calls, no data leaves the machine.

## Installation

Install the plugin in Claude Code:

```
/plugin install telegram-voice@<marketplace>
/reload-plugins
```

## Setup

### 1. Create a Telegram bot

Open [@BotFather](https://t.me/BotFather), send `/newbot`, and copy the token (`123456789:AAH...`).

### 2. Configure the bot token

```
/telegram-voice:configure 123456789:AAHfiqksKZ8...
```

### 3. Launch with the channel

```sh
claude --channels plugin:telegram-voice@<marketplace>
```

### 4. Pair your Telegram account

DM your bot — it replies with a pairing code. In Claude Code:

```
/telegram-voice:access pair <code>
/telegram-voice:access policy allowlist
```

Done. Send a voice message to test transcription.

## Voice Transcription

When a voice message arrives, the plugin:

1. Downloads the audio from Telegram
2. Converts OGA to WAV via ffmpeg
3. Runs whisper-cli with the medium model (auto-detects language)
4. Sends the transcribed text to Claude as `[voice transcription] ...`

### Auto-install

On first voice message, the plugin automatically installs missing dependencies via the detected package manager:

| Platform | Package manager | What gets installed |
|----------|----------------|---------------------|
| macOS | brew | `whisper-cpp`, `ffmpeg` |
| Linux (Debian/Ubuntu) | apt-get | `whisper-cpp`, `ffmpeg` |
| Linux (Fedora) | dnf | `whisper-cpp`, `ffmpeg` |
| Linux (Arch) | pacman | `whisper-cpp`, `ffmpeg` |
| Windows | winget / choco / scoop | `whisper-cpp`, `ffmpeg` |

The Whisper medium model (`ggml-medium.bin`, ~1.5 GB) is downloaded from HuggingFace automatically.

If auto-install fails, install manually:

```bash
# macOS
brew install whisper-cpp ffmpeg

# Ubuntu/Debian
sudo apt-get install whisper-cpp ffmpeg

# Windows
winget install ggerganov.whisper-cpp Gyan.FFmpeg
```

### Graceful Degradation

If whisper-cli, ffmpeg, or the model are unavailable, the plugin falls back to the existing `(voice message)` behavior. Zero breakage for users who don't need voice transcription.

### Configuration

Override paths via environment variables in `~/.claude/channels/telegram/.env`:

| Variable | Default |
|----------|---------|
| `WHISPER_CLI_PATH` | auto-detected |
| `FFMPEG_PATH` | auto-detected |
| `WHISPER_MODEL_PATH` | `~/.local/share/whisper-cpp/models/ggml-medium.bin` |
| `WHISPER_MODEL_NAME` | `ggml-medium.bin` |
| `WHISPER_MODEL_URL` | HuggingFace CDN |

## Prerequisites

- [Bun](https://bun.sh) — `curl -fsSL https://bun.sh/install | bash`

## Access Control

See **[ACCESS.md](./ACCESS.md)** for DM policies, groups, mention detection, and the `access.json` schema.

## Tools

| Tool | Purpose |
|------|---------|
| `reply` | Send to a chat (text, files, threading) |
| `react` | Add emoji reaction |
| `edit_message` | Edit a previously sent message |
| `download_attachment` | Download file attachments |

## License

Apache-2.0
