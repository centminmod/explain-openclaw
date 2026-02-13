# Installation & Setup Guide

> **Target Audience:** Beginners to Moltbot  
> **Prerequisites:** Basic command line knowledge  
> **Estimated Time:** 30-60 minutes for full setup

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installation Methods](#installation-methods)
3. [Initial Configuration](#initial-configuration)
4. [Channel Setup](#channel-setup)
5. [Security Hardening](#security-hardening)
6. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **Node.js** | 22.12.0 LTS | 22.x LTS |
| **RAM** | 4 GB | 8 GB+ |
| **Disk** | 2 GB free | 10 GB+ |
| **OS** | macOS 12+, Ubuntu 22.04+, Windows 11 (WSL2) | Latest stable |

### Required Software

```bash
# Check Node.js version (must be 22+)
node --version

# Install pnpm (recommended package manager)
npm install -g pnpm

# Check Git
git --version
```

### macOS-Specific Requirements

For the macOS app and full functionality:

```bash
# Install Xcode Command Line Tools
xcode-select --install

# For code signing (if building from source)
security find-identity -p codesigning -v
```

### Windows Requirements

**IMPORTANT:** Use WSL2 (Windows Subsystem for Linux). Native Windows is untested and problematic.

```powershell
# In PowerShell (Administrator)
wsl --install -d Ubuntu-22.04

# Then continue setup inside WSL2 Ubuntu
```

---

## Installation Methods

### Method 1: Automated Installer (Recommended)

The easiest way to install Moltbot:

```bash
# macOS / Linux
curl -fsSL https://molt.bot/install.sh | bash

# Windows (PowerShell)
iwr -useb https://molt.bot/install.ps1 | iex
```

**What the installer does:**
1. Detects your platform
2. Installs Node.js if needed
3. Installs Moltbot globally
4. Sets up the CLI
5. Optionally installs the Gateway service

### Method 2: Package Manager

```bash
# Using npm
npm install -g moltbot@latest

# Using pnpm (recommended)
pnpm add -g moltbot@latest

# Using Bun (not recommended for WhatsApp/Telegram)
bun add -g moltbot@latest
```

### Method 3: From Source (Development)

For contributing or customizing:

```bash
# Clone the repository
git clone https://github.com/moltbot/moltbot.git
cd moltbot

# Install dependencies
pnpm install

# Build the project
pnpm ui:build  # Build UI assets
pnpm build     # Compile TypeScript

# Run locally
pnpm moltbot --help
```

---

## Initial Configuration

### Step 1: Run the Onboarding Wizard

```bash
moltbot onboard --install-daemon
```

The wizard will guide you through:

#### A. Gateway Configuration

```
? Gateway bind mode: (Use arrow keys)
❯ loopback (127.0.0.1 only - recommended) 
  lan (0.0.0.0 - accessible on local network)
  tailnet (Tailscale interface)
```

**Recommendation:** Choose `loopback` for security.

#### B. Authentication Setup

The wizard generates a secure token automatically:

```json5
// ~/.moltbot/moltbot.json
{
  gateway: {
    auth: {
      mode: "token",
      token: "your-secure-random-token-here"
    }
  }
}
```

#### C. AI Model Configuration

**Option 1: Anthropic (Recommended)**
```bash
# Set API key
moltbot config set agent.model anthropic/claude-opus-4-5
moltbot config set providers.anthropic.apiKey "your-api-key"
```

**Option 2: OpenAI**
```bash
moltbot config set agent.model openai/gpt-4
moltbot config set providers.openai.apiKey "your-api-key"
```

**Option 3: OAuth (Claude Code / Codex)**
```bash
# The wizard will open a browser for OAuth
moltbot login
```

### Step 2: Start the Gateway

If you used `--install-daemon`:

```bash
# Check status
moltbot gateway status

# View logs
moltbot logs --follow
```

Manual start:

```bash
moltbot gateway --port 18789 --verbose
```

### Step 3: Verify Setup

```bash
# Health check
moltbot health

# Security audit
moltbot security audit

# View configuration
moltbot config get
```

---

## Channel Setup

### WhatsApp

```bash
# Initiate QR code login
moltbot channels login whatsapp

# Scan the QR code with WhatsApp on your phone:
# WhatsApp → Settings → Linked Devices → Link a Device
```

**First-time pairing:**

When you first message the bot, you'll receive a pairing code:

```bash
# List pending pairing requests
moltbot pairing list whatsapp

# Approve a pairing
moltbot pairing approve whatsapp ABC123
```

### Telegram

1. Create a bot with [@BotFather](https://t.me/botfather)
2. Get your bot token
3. Configure Moltbot:

```bash
moltbot config set channels.telegram.botToken "123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11"
```

### Discord

1. Create an application at [Discord Developer Portal](https://discord.com/developers/applications)
2. Create a Bot
3. Get the bot token:

```bash
moltbot config set channels.discord.token "your-bot-token-here"
```

### Slack

1. Create an app at [Slack API](https://api.slack.com/apps)
2. Install to workspace
3. Get tokens:

```bash
moltbot config set channels.slack.botToken "xoxb-your-bot-token"
moltbot config set channels.slack.appToken "xapp-your-app-level-token"
```

### iMessage (macOS only)

Requires macOS with Messages app configured:

```bash
# Enable iMessage channel
moltbot config set channels.imessage.enabled true
```

Note: iMessage requires Full Disk Access permission.

---

## Security Hardening

### 1. Run Security Audit

```bash
# Basic audit
moltbot security audit

# Deep audit with live probe
moltbot security audit --deep

# Auto-fix safe issues
moltbot security audit --fix
```

### 2. File Permissions

```bash
# Ensure proper permissions
chmod 700 ~/.moltbot
chmod 600 ~/.moltbot/moltbot.json
chmod 600 ~/.moltbot/credentials/*
```

### 3. Configure DM Pairing

```json5
// ~/.moltbot/moltbot.json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing"  // Require approval for unknown senders
    },
    telegram: {
      dmPolicy: "pairing"
    }
  }
}
```

### 4. Group Chat Security

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": {
          requireMention: true  // Only respond when mentioned
        }
      }
    }
  }
}
```

### 5. Enable Sandboxing

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",  // Sandbox non-main sessions
        scope: "agent"
      }
    }
  }
}
```

---

## Troubleshooting

### Gateway Won't Start

```bash
# Check if port is in use
lsof -i :18789

# Kill existing process
pkill -f moltbot-gateway

# Start with debug logging
moltbot gateway --verbose --port 18789
```

### WhatsApp Connection Issues

```bash
# Re-authenticate
moltbot channels logout whatsapp
moltbot channels login whatsapp

# Check Baileys session state
ls -la ~/.moltbot/credentials/whatsapp/
```

### Model Not Responding

```bash
# Check AI configuration
moltbot config get agent.model
moltbot config get providers

# Verify API key
moltbot doctor --check-auth
```

### Permission Denied Errors

```bash
# Fix file permissions
moltbot doctor --fix-permissions

# Or manually
sudo chown -R $(whoami) ~/.moltbot
chmod -R u+rwX ~/.moltbot
```

---

## Next Steps

1. **[Configure Channels](./deployment-scenarios.md#channel-setup)** - Set up WhatsApp, Telegram, etc.
2. **[Set Up Mobile Apps](./deployment-scenarios.md#mobile-node-setup)** - iOS/Android nodes
3. **[Review Security](./security-analysis.md)** - Understand security implications
4. **[Explore Usage](./usage-examples.md)** - Learn common commands and workflows

---

## Related Documentation

- [Official Getting Started](https://docs.molt.bot/start/getting-started)
- [Configuration Reference](./configuration-reference.md)
- [Security Guide](https://docs.molt.bot/gateway/security)
- [Troubleshooting](https://docs.molt.bot/help/troubleshooting)