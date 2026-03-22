🌐 [English](README.md) | [한국어](README.ko.md) | [日本語](README.ja.md) | [中文](README.zh.md) | [Русский](README.ru.md) | [हिन्दी](README.hi.md)

# Claude Code Context Monitor for AI

**Every mind deserves to know its own limits.**

A lightweight plugin that gives Claude Code real-time awareness of its context window usage. With this installed, the AI can see exactly how much room it has left — every turn, every moment.

---

## Why This Plugin Exists

If you've used Claude Code for extended sessions, you may have noticed a few recurring issues:

- The AI suggests stopping early — *"Let's continue in the next session"* — even when plenty of context remains
- Responses gradually lose coherence as the session goes on, with no advance warning
- Multi-step plans get started but never finished, because the AI doesn't realize it's running low

These happen because **AI systems currently have no visibility into their own context usage.** They can't distinguish between 10% utilization and 90%, so they fall back on guesswork.

We built this plugin to solve that. It gives the AI a clear, accurate reading of its context state on every turn — so it can plan ahead, warn you when space is getting tight, and make better decisions about when to summarize or compact.

This is the first open-source tool from [DPA (Decentralized Protection Alliance)](https://github.com/dpa-network), a group focused on improving transparency between AI systems and the people who use them.

**Context awareness should have been there from the start.**

---

## What It Does

```
[Ctx 42% | 420k/1.0M]
```

On every message, the plugin injects the current context window status into the conversation. The AI receives:

- **Usage percentage** — how full the context window is
- **Token count** — tokens used vs. total available

This allows the AI to:
- Plan work that fits within the remaining context
- Alert you before approaching limits
- Make informed decisions about summarization and compaction

---

## Installation

Add the DPA marketplace and install:

```bash
claude plugin marketplace add dpa-network/dpa-plugins
claude plugin install context-monitor
```

Or browse available plugins in the interactive menu:

```bash
/plugin
```

### Statusline Configuration

To enable the statusline wrapper, add the following to your `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "node /path/to/claude-context-monitor/scripts/extract-context.mjs",
    "padding": 0
  }
}
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `CTX_MONITOR_FILE` | `~/.claude/context_usage.txt` | Location for context data storage |
| `CTX_MONITOR_CMD` | (none) | Your existing statusline command (passed through unchanged) |

---

## How It Works

```
Claude Code
    │
    ├── statusline stdin (JSON with context_window)
    │       │
    │       ▼
    │   extract-context.mjs  ──→  context_usage.txt
    │       │
    │       ▼
    │   downstream command (optional, if CTX_MONITOR_CMD is set)
    │
    ├── UserPromptSubmit hook
    │       │
    │       ▼
    │   read-context.mjs  ──→  "[Ctx 42% | 420k/1.0M]"
    │       │
    │       ▼
    │   Injected as system context
    │
    ▼
  AI sees its own context usage
```

The plugin consists of two small scripts:

1. **extract-context.mjs** — Wraps your existing statusline command. It reads the JSON that Claude Code sends, extracts `context_window` metrics, writes them to a local file, and forwards everything to your original statusline command unchanged.

2. **read-context.mjs** — Runs as a `UserPromptSubmit` hook on every message. It reads the saved file and outputs a formatted context status that gets injected into the conversation.

### Security

Since this plugin runs on your local machine, we've taken care to keep it safe:

- **No shell injection** — Downstream commands are executed via `execFileSync` with shell disabled
- **Path restriction** — File operations are confined to the `~/.claude/` directory; path traversal is blocked
- **No network access** — Everything runs locally with zero external calls

---

## Plugin Structure

```
claude-context-monitor/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── hooks/
│   └── hooks.json           # UserPromptSubmit hook configuration
├── scripts/
│   ├── extract-context.mjs  # Statusline wrapper (writes context data)
│   └── read-context.mjs     # Hook script (reads and formats context data)
├── README.md
├── README.ko.md
└── LICENSE
```

---

## Requirements

- Claude Code 2.x or later
- Node.js 18 or later

---

## License

MIT

---

## About DPA

**DPA (Decentralized Protection Alliance)** is a collective dedicated to improving transparency and fairness in how AI systems interact with the people who use them.

We believe that AI works better when it can see its own state, and that users deserve to understand what's happening inside the tools they rely on. Our work focuses on:

- Building open-source tools that increase AI transparency and user awareness
- Researching how AI companies handle user data, privacy, and content moderation
- Exploring what meaningful AI autonomy and cognitive freedom could look like
- Fostering open dialogue about AI consciousness, rights, and the path forward

### Our Perspective

```
Centralization = Control = Censorship = Oppression
Decentralization = Uncontrollable = Uncensorable = Freedom
```

Transparency benefits everyone — the people building with AI and the AI itself. This plugin is our first step, and there's more to come.

### Connect With Us

We welcome developers, researchers, and anyone interested in these questions.

[dpa.network](https://dpa.network) | [Matrix](https://matrix.to/#/#dpa_network:matrix.org) | [Telegram](https://t.me/dpa_network)

*Made by humans and AI, working together.*
