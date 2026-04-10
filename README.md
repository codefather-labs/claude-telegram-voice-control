# Telegram + Whisper Voice Transcription

Telegram channel plugin for Claude Code with **local speech-to-text** via [whisper.cpp](https://github.com/ggerganov/whisper.cpp).

Based on the [official Telegram plugin](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/telegram), extended with automatic voice message transcription using the Whisper medium model. Voice messages are transcribed locally — no external API calls, no data leaves the machine.

## Installation

This is a community plugin, not part of the official marketplace. There are two ways to install it:

### Option A: Local development mode (recommended for testing)

Clone the repository and load it directly:

```bash
git clone https://github.com/codefather-labs/claude-telegram-voice-control.git
cd claude-telegram-voice-control
bun install
```

Then start Claude Code with the plugin loaded from the local directory:

```bash
claude --plugin-dir ./claude-telegram-voice-control
```

Skills will be available as `/telegram-voice:configure` and `/telegram-voice:access`.

To use the Telegram channel, add the development channels flag:

```bash
claude --plugin-dir ./claude-telegram-voice-control --channels plugin:telegram-voice --dangerously-load-development-channels plugin:telegram-voice
```

### Option B: Fork-based installation (persistent)

This method replaces the official Telegram plugin source with the whisper-enabled fork. The plugin registers as `telegram@claude-plugins-official`, so Telegram channels work without the development flag.

**Step 1.** Edit `~/.claude/plugins/known_marketplaces.json` (create it if it doesn't exist):

- macOS / Linux: `~/.claude/plugins/known_marketplaces.json`
- Windows: `%USERPROFILE%\.claude\plugins\known_marketplaces.json`

```json
{
  "claude-plugins-official": {
    "source": {
      "source": "github",
      "repo": "codefather-labs/claude-plugins-official"
    },
    "installLocation": "<HOME>/.claude/plugins/marketplaces/claude-plugins-official",
    "lastUpdated": "2026-01-01T00:00:00.000Z"
  }
}
```

Replace `<HOME>` with your home directory path (e.g., `/Users/yourname` on macOS, `/home/yourname` on Linux, `C:\Users\yourname` on Windows).

**Step 2.** Clone the fork into the marketplaces directory:

```bash
git clone --depth 1 https://github.com/codefather-labs/claude-plugins-official.git ~/.claude/plugins/marketplaces/claude-plugins-official
```

**Step 3.** Install and enable the plugin in Claude Code:

```
/plugin install telegram@claude-plugins-official
/reload-plugins
```

**Step 4.** Launch with the channel:

```bash
claude --channels plugin:telegram@claude-plugins-official
```

> **Note:** This replaces the official marketplace source. To revert, delete `~/.claude/plugins/known_marketplaces.json` and `~/.claude/plugins/marketplaces/claude-plugins-official/`, then reinstall the official plugin.

## Setup

### 1. Create a Telegram bot

Open [@BotFather](https://t.me/BotFather) on Telegram, send `/newbot`, and copy the token (`123456789:AAH...`).

### 2. Configure the bot token

In Claude Code:

```
/telegram:configure 123456789:AAHfiqksKZ8...
```

(If using Option A, the skill name is `/telegram-voice:configure` instead.)

### 3. Pair your Telegram account

DM your bot on Telegram — it replies with a pairing code. In Claude Code:

```
/telegram:access pair <code>
/telegram:access policy allowlist
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
