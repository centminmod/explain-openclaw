# Architecture Overview

This document explains how Clawdbot works under the hood, from receiving a message to delivering an AI response.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         YOUR DEVICE                             │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    CLAWDBOT GATEWAY                       │  │
│  │                     (Port 18789)                          │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │  │
│  │  │  Channel    │  │  Message    │  │    AI       │       │  │
│  │  │  Plugins    │  │  Router     │  │   Agent     │       │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘       │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
         │                                          │
         ▼                                          ▼
┌─────────────────────┐                 ┌─────────────────────┐
│  MESSAGING APPS     │                 │   AI PROVIDERS      │
│  - WhatsApp         │                 │   - Anthropic       │
│  - Telegram         │                 │   - OpenAI          │
│  - Discord          │                 │   - Google          │
│  - Signal           │                 │   - Ollama (local)  │
│  - Slack            │                 │                     │
└─────────────────────┘                 └─────────────────────┘
```

---

## Core Components

### 1. CLI (Command Line Interface)

**Location:** `src/cli/`

The CLI is how you interact with Clawdbot from the terminal:

```bash
clawdbot <command> [options]
```

**Key Responsibilities:**
- Setup and configuration (`clawdbot onboard`, `clawdbot config`)
- Starting/stopping the gateway (`clawdbot gateway run`)
- Health checks (`clawdbot doctor`, `clawdbot channels status`)
- Direct messaging (`clawdbot message send`)

---

### 2. Gateway Server

**Location:** `src/gateway/server.impl.ts`

The Gateway is the heart of Clawdbot - a server that:
- Listens on port **18789** (by default)
- Accepts WebSocket and HTTP connections
- Coordinates all channel plugins and AI interactions

```
┌─────────────────────────────────────────┐
│            GATEWAY SERVER               │
│                                         │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  │
│  │  HTTP   │  │WebSocket│  │  Auth   │  │
│  │ Routes  │  │ Handler │  │ Manager │  │
│  └─────────┘  └─────────┘  └─────────┘  │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │        Plugin Services          │    │
│  │   (Channels, AI, Tools, etc.)   │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

**Key Features:**
- Multi-tenant (supports multiple users/channels)
- Hot reload (config changes without restart)
- Health monitoring and diagnostics

---

### 3. Channel Plugins

**Location:** `src/channels/`, `extensions/`

Channel plugins connect Clawdbot to messaging platforms. Each plugin:
- Monitors for new messages (polling or webhooks)
- Parses incoming messages into a common format
- Delivers AI responses back to the platform

**Built-in Channels (in core):**

| Channel | Implementation |
|---------|----------------|
| Telegram | grammY Bot API |
| WhatsApp | Baileys (Web protocol) |
| Discord | discord.js |
| Slack | Bolt SDK |
| Signal | signal-cli REST API |
| iMessage | macOS native (AppleScript) |
| Google Chat | Google API |

**Extension Channels (28 plugins):**

| Plugin | Description |
|--------|-------------|
| `bluebubbles` | Alternative iMessage bridge |
| `matrix` | Matrix/Element protocol |
| `msteams` | Microsoft Teams |
| `mattermost` | Mattermost chat |
| `line` | LINE messenger |
| `nextcloud-talk` | Nextcloud Talk |
| `nostr` | Nostr protocol |
| `twitch` | Twitch chat |
| `voice-call` | Voice call handling |
| `zalo` / `zalouser` | Zalo messenger |
| `tlon` | Tlon/Urbit |

---

### 4. AI Agent

**Location:** `src/agents/`, `src/auto-reply/`

The Agent is the "brain" that processes messages and generates responses:

```
┌────────────────────────────────────────────────────────────┐
│                       AI AGENT                             │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
��  │   Context    │  │    Model     │  │    Tools     │     │
│  │   Builder    │  │   Provider   │  │   Executor   │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │               System Prompt + Memory                 │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

**Key Files:**
- `src/auto-reply/reply/agent-runner.ts` - Main agent execution loop
- `src/agents/system-prompt.ts` - System prompt generation
- `src/agents/auth-profiles/store.ts` - Credential management

---

### 5. Configuration System

**Location:** `src/config/`

Configuration is stored in JSON5 format at `~/.clawdbot/clawdbot.json`:

```
~/.clawdbot/
├── clawdbot.json          # Main configuration
├── credentials/           # AI provider credentials
├── agents/                # Per-agent data
│   └── <agentId>/
│       └── sessions/      # Conversation logs (*.jsonl)
└── sessions/              # Platform session data
```

**Key Files:**
- `src/config/io.ts` - Config file I/O
- `src/config/types.ts` - TypeScript type definitions
- `src/config/validation.ts` - Schema validation

---

## Message Flow

Here's what happens when you send a message:

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. USER SENDS MESSAGE                                           │
│    "What's the weather today?" via Telegram                     │
└────────────────────────────────────┬────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. CHANNEL MONITOR RECEIVES                                     │
│    Telegram plugin polls/receives via webhook                   │
│    Parses message: sender, text, attachments                    │
└────────────────────────────────────┬────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. ACCESS CHECK                                                 │
│    Is sender in allowlist?                                      │
│    Is this chat allowed?                                        │
└────────────────────────────────────┬────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. ROUTE RESOLUTION                                             │
│    Which agent should handle this?                              │
│    Apply bindings/routing rules                                 │
└────────────────────────────────────┬────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. CONTEXT BUILDING                                             │
│    Load conversation history                                    │
│    Build system prompt                                          │
│    Attach media if present                                      │
└────────────────────────────────────┬────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. AI MODEL CALL                                                │
│    Send to provider (Anthropic, OpenAI, etc.)                   │
│    Stream response chunks                                       │
│    Execute tools if needed (web search, code, etc.)             │
└────────────────────────────────────┬────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ 7. RESPONSE DELIVERY                                            │
│    Format for platform (split long messages, etc.)              │
│    Send back via Telegram                                       │
│    Log to session history                                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Directory Structure

```
clawdbot/
├── src/
│   ├── cli/              # Command-line interface
│   ├── gateway/          # Gateway server implementation
│   ├── channels/         # Built-in channel plugins
│   ├── agents/           # AI agent logic
│   ├── auto-reply/       # Message processing pipeline
│   ├── config/           # Configuration system
│   ├── security/         # Security features
│   ├── media/            # Media processing (audio/video/image)
│   └── infra/            # Infrastructure utilities
├── extensions/           # Extension plugins (28+)
├── docs/                 # Documentation
├── dist/                 # Built output
└── apps/                 # Native apps (iOS, Android, macOS)
```

---

## Key Concepts

### Bindings

Bindings route messages to specific agents:

```json5
{
  "bindings": [
    {
      "match": { "channel": "telegram", "chat": "@work_group" },
      "agent": "work-assistant"
    },
    {
      "match": { "channel": "whatsapp" },
      "agent": "personal-assistant"
    }
  ]
}
```

### Sessions

Each conversation has a session with:
- Message history
- Context/memory
- User preferences

Sessions are stored as JSONL files in `~/.clawdbot/agents/<agentId>/sessions/`

### Tools

The AI can use "tools" to take actions:
- **Web Fetch** - Get content from URLs
- **Code Execution** - Run sandboxed code
- **File Operations** - Read/write files
- **Browser** - Automated web browsing

---

## Runtime Requirements

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| Node.js | 22+ | 22 LTS |
| RAM | 512MB | 2GB+ |
| Storage | 100MB | 1GB+ (for sessions) |
| Network | Outbound HTTPS | Stable connection |

---

*Continue to [03 - Messaging Channels](./03-messaging-channels.md) →*
