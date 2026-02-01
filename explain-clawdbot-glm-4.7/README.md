# Beginner's Guide to Clawdbot

Welcome! This documentation explains Clawdbot in plain English for anyone who wants to understand what it is and how to use it.

## What is this documentation?

This is a beginner-friendly introduction to **Clawdbot** — a personal AI assistant that you run on your own devices. If you've used ChatGPT or similar AI tools, Clawdbot is like having your own private version that:

- Lives on your computer (not someone else's server)
- Talks to you on apps you already use (WhatsApp, Telegram, etc.)
- Keeps your conversations private
- Works the way you want it to

## Quick overview

**Clawdbot** is a personal AI assistant framework that:

- Runs locally on your own hardware (Mac, Linux, VPS)
- Connects to your favorite messaging apps
- Uses powerful AI models (Claude, GPT-4, and others)
- Stores everything on your device
- Has no telemetry or tracking

### Prerequisites

Before you start, you need:

- **Node.js 22 or higher** — download from [nodejs.org](https://nodejs.org)
- A macOS or Linux computer (Windows works via WSL2)
- An account with an AI provider (Anthropic for Claude, or OpenAI for GPT-4)
- Basic comfort with terminal commands

## Two main ways to use Clawdbot

### 1. On a Mac Mini (recommended for home use)

A Mac Mini sitting in your home is ideal because:
- It's always on and available
- Low power consumption
- Can run the macOS app with a nice menu bar interface
- Great for privacy (data stays in your home)

**Start here:** [usage-mac-mini.md](./usage-mac-mini.md)

### 2. On a VPS (recommended for remote access)

A Virtual Private Server in the cloud is great because:
- Accessible from anywhere
- No hardware to maintain
- Can be isolated for maximum security
- Lower cost than buying dedicated hardware

**Start here:** [usage-vps.md](./usage-vps.md)

## Documentation contents

| File | What it covers |
|------|----------------|
| [what-is-clawdbot.md](./what-is-clawdbot.md) | Plain English explanation of what Clawdbot is and why it exists |
| [how-it-works.md](./how-it-works.md) | Technical architecture, simplified for beginners |
| [installation.md](./installation.md) | Step-by-step installation guide |
| [configuration.md](./configuration.md) | How to configure Clawdbot to your needs |
| [privacy-security.md](./privacy-security.md) | Privacy and security features explained |
| [usage-mac-mini.md](./usage-mac-mini.md) | Guide for setting up on a Mac Mini |
| [usage-vps.md](./usage-vps.md) | Guide for setting up on a VPS |
| [reference.md](./reference.md) | Quick reference for commands and troubleshooting |

## Quick start

If you want to jump right in, here's the fastest way to get started:

```bash
# Install Clawdbot
npm install -g clawdbot@latest

# Run the onboarding wizard
clawdbot onboard --install-daemon

# Start the gateway
clawdbot gateway --port 18789

# Send your first message
clawdbot agent --message "Hello, Clawdbot!"
```

## What makes Clawdbot different?

| Commercial AI (ChatGPT, Claude, etc.) | Clawdbot |
|----------------------------------------|----------|
| Runs on company servers | Runs on your hardware |
| Your data processed remotely | Your data stays local |
| One-size-fits-all interface | Works in your existing apps |
| Subscription required | Use your own API keys |
| Limited control | Fully customizable |

## Security Audit

Two security audits have been published about Clawdbot. Here's an honest assessment of both.

### Audit #1: GitHub Issue #1796 (January 2026)

A comprehensive audit was reported in [GitHub Issue #1796](https://github.com/clawdbot/clawdbot/issues/1796) with 512 findings across multiple categories.

| Category | Audit Finding | Reality |
|----------|---------------|----------|
| OAuth CSRF | Critical vulnerability in state validation | **False positive** — Device OAuth flows use PKCE, not state parameters |
| Credential storage | Tokens stored in plaintext without encryption | **Accurate, but by design** — Uses `0o600` file permissions; relies on OS-level security (encrypted disk) |
| Webhook bypass | Signature verification can be disabled | **Accurate** — Dev-only flag exists; verify production configs |

### Audit #2: Medium Article "Why Clawdbot is a Bad Idea" (Saad Khalid)

A Medium article titled ["Why Clawdbot is a Bad Idea: Critical Zero-Days Found in My Audit"](https://saadkhalidhere.medium.com/why-clawdbot-is-a-bad-idea-critical-zero-days-found-in-my-audit-full-report-634602cb053f) claimed multiple critical vulnerabilities.

| Claim | Reality |
|-------|----------|
| Directory Traversal (CVE-2024-44946) | **False positive** — Media handling has proper path validation and filename sanitization |
| OS Command Injection via Filename | **False positive** — No shell execution with user filenames; uses safe Node.js APIs |
| Hardcoded Credentials | **Exaggerated** — Only test fixtures with mock keys; no production secrets in source |
| Insecure Dependencies | **Partially accurate** — Express 5.x pre-release is a concern; missing `npm audit` in CI |
| Lack of Input Validation | **Misleading** — Web APIs have comprehensive JSON Schema validation; CLI has gaps |

### What this means for you

Clawdbot's security model is **trust-based**: it assumes you control your machine and your operating system provides basic security (file permissions, disk encryption).

- **If you run on a personal Mac Mini with FileVault**: Current security is appropriate
- **If you run on a shared server**: Review your threat model and consider additional isolation
- **If you use the voice-call plugin**: Verify signature verification is enabled in production

### Trust but verify

Both audit reports contain accurate findings alongside false positives or exaggerated claims. The project maintainers have reviewed and responded to these reports. For complete security guidance, see [Privacy and Security](./privacy-security.md).

## Need help?

- **Official docs:** https://docs.clawd.bot
- **GitHub:** https://github.com/clawdbot/clawdbot
- **Discord:** https://discord.gg/clawd
- **Run `clawdbot doctor`** to diagnose common issues

## Next steps

1. Read [what-is-clawdbot.md](./what-is-clawdbot.md) to understand what Clawdbot does
2. Check [how-it-works.md](./how-it-works.md) for the architecture overview
3. Follow [installation.md](./installation.md) to install it
4. Choose your setup path: [Mac Mini](./usage-mac-mini.md) or [VPS](./usage-vps.md)

---

**Remember:** Clawdbot is about privacy and control. Your AI, your data, your rules. 🦞
