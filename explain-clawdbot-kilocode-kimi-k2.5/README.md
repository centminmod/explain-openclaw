# Understanding Moltbot (formerly Clawdbot): A Beginner's Guide

> **Analysis Date:** January 2026  
> **Analyzed by:** Kimi K2.5 via Kilo Code  
> **Repository:** https://github.com/moltbot/moltbot  
> **Version Analyzed:** 2026.1.27-beta.1

---

## Table of Contents

1. [What is Moltbot? (Plain English)](#what-is-moltbot-plain-english)
2. [What is Moltbot? (Technical Overview)](#what-is-moltbot-technical-overview)
3. [How It Works](#how-it-works)
4. [Key Features](#key-features)
5. [Quick Start Guide](#quick-start-guide)
6. [Deployment Scenarios](#deployment-scenarios)
7. [Security Considerations](#security-considerations)
8. [Related Documentation](#related-documentation)

---

## What is Moltbot? (Plain English)

**Moltbot** is like having a personal AI assistant that lives on your own computer (or server) and can talk to you through the messaging apps you already use—WhatsApp, Telegram, Slack, Discord, and more.

Imagine you could:
- **Text your AI assistant** on WhatsApp and ask it to check your calendar, write code, or search the web
- **Have it monitor your systems** and alert you when something goes wrong
- **Control it with voice commands** on your Mac, iPhone, or Android device
- **Keep all your data private** because it runs on your own hardware, not in some company's cloud

That's Moltbot. It's an open-source project that bridges the gap between powerful AI models (like Claude, GPT-4, etc.) and your everyday communication tools.

### Why the name "Clawdbot" / "Moltbot"?

The project was originally called **Clawdbot** (with a 🦞 lobster mascot named "Molty"). It was rebranded to **Moltbot** in early 2026, though the `clawdbot` command remains available for backward compatibility.

---

## What is Moltbot? (Technical Overview)

Moltbot is a **self-hosted AI agent gateway** written in TypeScript/Node.js. It provides:

### Core Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        MOLTBOT SYSTEM                           │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │   Channels   │    │   Gateway    │    │   AI Agent   │      │
│  │              │◄──►│  (Control    │◄──►│   (Pi RPC)   │      │
│  │ - WhatsApp   │    │   Plane)     │    │              │      │
│  │ - Telegram   │    │              │    │ Tools:       │      │
│  │ - Slack      │    │ WebSocket    │    │ - bash/exec  │      │
│  │ - Discord    │    │ Server       │    │ - browser    │      │
│  │ - Signal     │    │ Port 18789   │    │ - read/write │      │
│  │ - iMessage   │    │              │    │ - canvas     │      │
│  │ - WebChat    │    │ Auth,        │    │ - cron       │      │
│  └──────────────┘    │ Sessions,    │    └──────────────┘      │
│                      │ Routing      │                          │
│                      └──────────────┘                          │
│                              ▲                                  │
│                              │                                  │
│         ┌────────────────────┼────────────────────┐            │
│         ▼                    ▼                    ▼            │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │   CLI Tool   │    │  macOS App   │    │  iOS/Android │      │
│  │  (moltbot)   │    │  (Menu Bar)  │    │    Nodes     │      │
│  └──────────────┘    └──────────────┘    └──────────────┘      │
└─────────────────────────────────────────────────────────────────┘
```

### Technology Stack

| Component | Technology |
|-----------|------------|
| **Runtime** | Node.js 22+ (LTS) |
| **Language** | TypeScript (ESM) |
| **Package Manager** | pnpm (preferred), npm, or Bun |
| **WebSocket Server** | Custom WS implementation |
| **AI Runtime** | Pi RPC Agent (via `@mariozechner/pi-agent-core`) |
| **WhatsApp** | Baileys library |
| **Telegram** | grammY framework |
| **Slack** | Bolt SDK |
| **Discord** | discord.js |
| **Mobile Apps** | Swift (iOS), Kotlin (Android) |
| **Testing** | Vitest with V8 coverage |

### Key Dependencies

- **[@mariozechner/pi-agent-core](https://github.com/badlogic/pi-mono)** - AI agent runtime
- **[Baileys](https://github.com/WhiskeySockets/Baileys)** - WhatsApp Web API
- **[grammY](https://grammy.dev)** - Telegram Bot framework
- **[discord.js](https://discord.js.org)** - Discord API library
- **[@slack/bolt](https://slack.dev/bolt-js)** - Slack app framework

---

## How It Works

### 1. The Gateway (Central Hub)

The **Gateway** is the heart of Moltbot. It's a WebSocket server that:
- Manages connections to all messaging channels
- Handles authentication and sessions
- Routes messages between users and AI agents
- Provides a web-based Control UI

Default configuration:
- **Port:** 18789
- **Bind:** 127.0.0.1 (loopback only, for security)
- **Protocol:** WebSocket with JSON message framing

### 2. Message Flow

```
1. User sends message on WhatsApp
         │
         ▼
2. Baileys (WhatsApp library) receives it
         │
         ▼
3. Gateway validates sender (allowlist/pairing)
         │
         ▼
4. Message routed to AI agent via Pi RPC
         │
         ▼
5. AI generates response (using configured model)
         │
         ▼
6. Response sent back through Gateway
         │
         ▼
7. User receives reply on WhatsApp
```

### 3. Security Model

Moltbot uses a **"trust but verify"** approach:

| Layer | Mechanism |
|-------|-----------|
| **Network** | Loopback-only by default; optional Tailscale/VPN |
| **Authentication** | Token or password-based Gateway auth |
| **DM Access** | Pairing system (unknown senders get a code) |
| **Group Access** | Mention gating + allowlists |
| **Tool Access** | Per-agent sandbox configuration |
| **File System** | Permission checks (600/700) |

---

## Key Features

### Multi-Channel Support

Connect with your AI through:
- 📱 WhatsApp (via QR code pairing)
- ✈️ Telegram (bot token)
- 💬 Slack (workspace app)
- 🎮 Discord (bot integration)
- 💌 iMessage (macOS only)
- 🔵 Signal (via signal-cli)
- 🌐 WebChat (built-in web UI)

### AI Model Support

Works with multiple AI providers:
- **Anthropic** (Claude 3.5 Sonnet, Opus 4.5)
- **OpenAI** (GPT-4, GPT-5.2, Codex)
- **Local Models** (Ollama, llama.cpp)
- **Other Providers** (OpenRouter, Venice, etc.)

### Tools & Capabilities

The AI can use various tools:
- **bash/exec** - Execute shell commands
- **browser** - Control a real browser (Chrome/Chromium)
- **read/write/edit** - File system operations
- **canvas** - Create visual content via A2UI
- **cron** - Schedule recurring tasks
- **web_search/web_fetch** - Internet research
- **nodes** - Control connected devices (camera, screen, location)

### Voice & Mobile

- **Voice Wake** - Always-listening voice trigger (macOS/iOS/Android)
- **Talk Mode** - Continuous voice conversation
- **Canvas** - Visual workspace for agent-driven content

---

## Quick Start Guide

### Prerequisites

- **Node.js 22+** (check with `node --version`)
- **pnpm** recommended: `npm install -g pnpm`
- For macOS app: Xcode Command Line Tools
- For mobile apps: Xcode (iOS) or Android Studio (Android)

### Installation

```bash
# Option 1: Automated installer (recommended)
curl -fsSL https://molt.bot/install.sh | bash

# Option 2: npm global install
npm install -g moltbot@latest

# Option 3: From source
git clone https://github.com/moltbot/moltbot.git
cd moltbot
pnpm install
pnpm build
```

### Initial Setup

```bash
# Run the onboarding wizard
moltbot onboard --install-daemon

# This will guide you through:
# - Gateway configuration
# - AI model authentication (OAuth/API keys)
# - Channel setup (WhatsApp, Telegram, etc.)
# - Pairing and security settings
```

### Verify Installation

```bash
# Check Gateway status
moltbot gateway status

# Run security audit
moltbot security audit --deep

# Send a test message
moltbot message send --to +1234567890 --message "Hello from Moltbot!"
```

---

## Deployment Scenarios

### 1. Standalone Mac Mini

Perfect for home/personal use with maximum privacy.

**Pros:**
- Complete data control
- Local network only
- Easy macOS app integration

**Cons:**
- Requires always-on Mac
- Limited to local network (without VPN)

See: [Deployment: Mac Mini](./deployment-scenarios.md#scenario-1-standalone-mac-mini)

### 2. Isolated VPS (Linux)

Deploy on a cloud server for 24/7 availability.

**Pros:**
- Always online
- Accessible from anywhere
- Can be isolated from personal devices

**Cons:**
- Requires server management
- Potential cloud provider trust issues

See: [Deployment: VPS](./deployment-scenarios.md#scenario-2-isolated-vps)

### 3. Cloudflare Moltworker

Serverless deployment using Cloudflare's edge network.

**Pros:**
- No server management
- Global edge deployment
- Built-in DDoS protection

**Cons:**
- Limited execution time
- Different architecture

See: [Deployment: Cloudflare Moltworker](./cloudflare-moltworker.md)

---

## Security Considerations

**⚠️ IMPORTANT:** Moltbot is a powerful tool that can execute shell commands, access files, and send messages. Running an AI with these capabilities requires careful security consideration.

### Critical Security Issues (as of January 2026)

Two major security audits have identified significant concerns:

1. **[GitHub Issue #1796](https://github.com/clawdbot/clawdbot/issues/1796)** - Argus Security audit found 512 findings including:
   - Plaintext OAuth token storage
   - Missing CSRF protection
   - Path traversal vulnerabilities
   - Hardcoded secrets

2. **[Saad Khalid's Security Audit](https://saadkhalidhere.medium.com/why-clawdbot-is-a-bad-idea-critical-zero-days-found-in-my-audit-full-report-634602cb053f)** - Identified:
   - SSRF (Server-Side Request Forgery) vulnerabilities
   - RCE (Remote Code Execution) via logic bombs
   - Environment variable injection attacks
   - Shell argument injection

### Security Best Practices

1. **Never expose Gateway to public internet** without authentication
2. **Use pairing mode** for all DM-capable channels
3. **Enable sandboxing** for non-trusted agents
4. **Run `moltbot security audit --deep`** regularly
5. **Keep file permissions strict** (600/700)
6. **Use Tailscale or VPN** for remote access, not port forwarding

See detailed analysis: [Security Analysis](./security-analysis.md)

---

## Related Documentation

| Document | Description |
|----------|-------------|
| [Installation & Setup](./installation-and-setup.md) | Detailed installation instructions |
| [Deployment Scenarios](./deployment-scenarios.md) | Mac Mini, VPS, and cloud deployments |
| [Cloudflare Moltworker](./cloudflare-moltworker.md) | Serverless deployment guide |
| [Security Analysis](./security-analysis.md) | Security issues and hardening |
| [Usage Examples](./usage-examples.md) | Common use cases and commands |
| [Configuration Reference](./configuration-reference.md) | Complete config options |

---

## Official Resources

- **Website:** https://molt.bot
- **Documentation:** https://docs.molt.bot
- **GitHub:** https://github.com/moltbot/moltbot
- **Discord Community:** https://discord.gg/clawd
- **Blog:** https://blog.molt.bot

---

## License

Moltbot is released under the MIT License. See [LICENSE](https://github.com/moltbot/moltbot/blob/main/LICENSE) for details.

---

*This documentation was generated by Kilo Code using the Kimi K2.5 model for comprehensive analysis and explanation.*