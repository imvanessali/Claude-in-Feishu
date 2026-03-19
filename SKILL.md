---
name: claude-in-feishu
description: Turn Claude Code into a mobile-controlled autopilot — operate your Mac, Feishu docs/calendar, and local dev environment from your phone. Includes session journaling with auto-summarization.
argument-hint: "setup | start | stop | status | logs | doctor | journal"
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - AskUserQuestion
---

# Claude Autopilot

**Control Claude Code from your phone.** Send tasks via Feishu/Discord/Telegram, operate local files, manage Feishu docs & calendar, and auto-journal every session.

## Architecture

```
Phone (Feishu/Discord) → Local daemon → Claude Code CLI → Your Mac + Feishu API
                                                         ↓
                                              Auto-journal on exit (Haiku)
```

## Capabilities

| Capability | How |
|---|---|
| **Mobile → Claude Code** | IM bot daemon bridges messages to local Claude Code sessions |
| **Feishu Docs** | Create, read, search, append, move documents |
| **Feishu Calendar** | Create events, list calendars |
| **Send files to phone** | Upload images/files directly to Feishu chat |
| **Screenshot review** | Take screenshots of local apps, send to Feishu for mobile product review |
| **Cross-session visibility** | All sessions run in tmux, can read each other |
| **Session journals** | Stop hook auto-summarizes every session via Haiku |

## Subcommands

### `setup`
Interactive setup wizard. Configures:
- IM platform (Feishu, Discord, Telegram)
- Bot tokens and credentials
- Feishu App ID/Secret for docs & calendar
- Default model and working directory

### `start`
Start the bridge daemon (auto-restarts via launchd).

### `stop`
Stop the bridge daemon.

### `status`
Show daemon status, active sessions, and channel health.

### `logs [N]`
Show last N lines of bridge logs (default 50).

### `doctor`
Run diagnostics: Node.js version, Claude CLI, SDK, token validity, config permissions.

### `journal`
Manually trigger session journal generation for the current session.

## Components

### 1. IM Bridge (`src/`)
Node.js daemon using `claude-to-im` + `@anthropic-ai/claude-agent-sdk`:
- **llm-provider.ts** — Spawns Claude Code via SDK, auto-retries on stale sessions
- **store.ts** — File-based persistence (sessions, bindings, messages, audit)
- **permission-gateway.ts** — Async tool permission via IM buttons (5-min timeout)
- **config.ts** — Reads `~/.claude-in-feishu/config.env`
- **logger.ts** — Secret-redacting log rotation

### 2. Feishu Toolkit (`scripts/feishu_docs.py`)
Python CLI for Feishu Open API:
- `read/create/append/search/move/list` — Document ops
- `cal_list/event_create/event_list` — Calendar ops
- `send_image/send_file` — Send files to Feishu chat
- `mkdir/folders/organize` — Drive management

### 3. Session Journals (`journals/`)
Auto-journaling on session exit:
- **journal.sh** — Stop hook entry, extracts JSONL transcript, spawns worker
- **journal-worker.sh** — Calls Haiku to generate structured summary
- **extract-transcript.py** — Converts Claude Code `.jsonl` to readable text
- **INDEX.md** — Cover index with tags for retrieval

### 4. Tmux Wrapper
Shell function that auto-wraps `claude` in tmux sessions for cross-session visibility.

## Prerequisites

- macOS with Claude Code CLI installed
- Node.js 20+
- Python 3
- tmux
- A Feishu custom app (for Feishu integration)
- Discord bot token (for Discord integration)
