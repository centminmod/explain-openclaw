# What is OpenClaw? (plain English, beginner-first)

## Table of contents (Explain OpenClaw)

- [Home (README)](../README.md)
- Plain English
  - [What is OpenClaw?](./what-is-clawdbot.md)
  - [Glossary](./glossary.md)
  - [CLI commands](./cli-commands.md)
- Technical
  - [Architecture](../02-technical/architecture.md)
  - [Repo map](../02-technical/repo-map.md)
- Privacy + safety
  - [Threat model](../04-privacy-safety/threat-model.md)
  - [Hardening checklist](../04-privacy-safety/hardening-checklist.md)
- Deployment
  - [Standalone Mac mini](../03-deploy/standalone-mac-mini.md)
  - [Isolated VPS](../03-deploy/isolated-vps.md)
  - [Cloudflare Moltworker](../03-deploy/cloudflare-moltworker.md)
- Optimizations
  - [Cost + token optimization](../06-optimizations/cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

OpenClaw is a framework for running a **personal AI assistant** on hardware you control.

If you've only used ChatGPT/Claude/Gemini in a browser:
- those products are *apps* hosted by someone else
- OpenClaw is the *platform* you run yourself, and you plug your chosen model/provider into it

OpenClaw's superpower is not "a smarter model." It's that it can:
- live **where you already talk** (WhatsApp, Telegram, Discord, iMessage, …)
- stay **always-on** (Gateway service — can run as a system daemon via `openclaw daemon install` on systemd/launchd/schtasks for automatic start on boot)
- keep **state** (sessions, memory, policies)
- optionally **take actions** (tools, device nodes)

Official overview (good high-level read): https://docs.openclaw.ai

---

## The one-sentence mental model

> **OpenClaw is a self-hosted Gateway that connects chat apps to an agent that can reason and (optionally) act.**

---

## The cast of characters (with analogies)

### 1) The Gateway = “your assistant’s home base”
The **Gateway** is a long-running process that you run on a machine you control.

It:
- receives inbound messages from chat platforms (channels)
- decides what should happen (routing + policy)
- runs the agent turn (calls your model provider)
- sends a response back to the same chat platform
- stores local state (config, credentials, session logs)

Think of it as:
- a router + scheduler + policy engine for your assistant

Docs: https://docs.openclaw.ai/gateway

### 2) Channels = "phone lines" into the assistant
A **channel** is how OpenClaw connects to a messaging platform.

Examples:
- WhatsApp (via WhatsApp Web / Baileys)
- Telegram (Bot API)
- Discord
- iMessage (macOS integration)
- plus plugins for many more

Channels normalize “incoming message events” into a common internal shape.

Docs: https://docs.openclaw.ai/channels

### 3) Sessions = “conversation memory (by default) on disk”
A **session** is a conversation thread with state:
- chat history (transcripts)
- metadata (who/where it came from)
- optional **semantic memory** (`MEMORY.md` + `memory/*.md` files indexed via embeddings for hybrid keyword + vector search — the agent can recall past context across sessions)

Sessions live on disk under your state directory (usually `~/.openclaw/`).

Docs: https://docs.openclaw.ai/concepts/session

### 4) The agent = "the brain (model + rules + tool policy)"
The agent is where your AI model (Claude/GPT/etc.) is actually called.

OpenClaw supplies:
- system prompt templates
- history/context
- safety wrappers
- tool availability rules

Docs: https://docs.openclaw.ai/concepts/agent

### 5) Tools = “hands” (powerful; risky)
Tools let the model do more than output text.

Depending on what you enable, tools can include:
- web fetch/search
- browser automation
- cron/automation
- exec or node/device invocations
- memory search (semantic search across indexed notes and past conversations)
- canvas/drawing (interactive visual outputs)
- image processing (resize, optimize, convert)
- text-to-speech (ElevenLabs, OpenAI, Edge TTS)
- session management (list, send, history across sessions)
- gateway control (config, status, restart from within agent)

Tools are where most real-world risk comes from.

Docs: https://docs.openclaw.ai/tools

### 6) Nodes/devices = “peripherals”
Nodes are devices (macOS/iOS/Android) that can connect to the Gateway and offer device-local capabilities.

Examples:
- camera (snap, clip, list cameras)
- screen recording
- location services
- notifications
- canvas/webviews
- (on macOS) remote execution with approvals

Docs: https://docs.openclaw.ai/nodes

---

## What OpenClaw is great for

### Personal / small-team assistant in chat
- “Summarize the last 200 messages in this group.”
- “Draft a reply in my tone.”
- “Keep track of ongoing tasks.”

### A private assistant with your own policies
Because you control the Gateway, you can enforce:
- who can talk to it
- what it can do
- where it can reach

### An always-on agent host
A VPS or home server can keep the Gateway running even if your laptop sleeps.

---

## What OpenClaw is *not*

### Not automatically safe for public bots
If you open inbound DMs/groups to the public *and* enable tools, you've basically built a remote-controlled automation engine.

OpenClaw ships safety features (pairing, allowlists, audits), but you must use them.

Docs: https://docs.openclaw.ai/gateway/security

### Not "privacy magic"
OpenClaw keeps state locally, but your chosen model provider still receives the prompts you send for inference unless you run a local model.

---

## Beginner-safe starting posture (recommended)

If you want high privacy and low risk:

1) Keep the Gateway **loopback-only** (localhost only)
2) Use **pairing** and/or **allowlists** so only you can trigger it
3) Avoid enabling powerful tools until you are comfortable
4) Run `openclaw security audit --deep` and fix findings

See:
- https://docs.openclaw.ai/gateway/security
- https://docs.openclaw.ai/start/pairing

Next: [Glossary](./glossary.md) | [CLI commands](./cli-commands.md)
