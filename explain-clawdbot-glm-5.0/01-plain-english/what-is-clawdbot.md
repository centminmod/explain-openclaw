# What is OpenClaw? (plain English, beginner-first)

## Table of contents (Explain OpenClaw - GLM-5.0 Edition)

- [Home (README)](../README.md)
- Plain English
  - [What is OpenClaw?](./what-is-clawdbot.md)
  - [Glossary](./glossary.md)
- Technical
  - [Architecture](../02-technical/architecture.md)
  - [Repo map](../02-technical/repo-map.md)
- Privacy + safety
  - [Threat model](../04-privacy-safety/threat-model.md)
  - [Hardening checklist](../04-privacy-safety/hardening-checklist.md)
  - [High privacy config example](../04-privacy-safety/high-privacy-config.example.json5.md)
- Installation
  - [Install and onboarding](../03-install/install-and-onboard.md)
  - [Build from source](../03-install/from-source.md)
- Deployment
  - [Standalone Mac mini](../05-use-cases/mac-mini-standalone.md)
  - [Isolated VPS](../05-use-cases/vps-isolated.md)

---

OpenClaw is a framework for running a **personal AI assistant** on hardware you control.

If you've only used ChatGPT/Claude/Gemini in a browser:
- those products are *apps* hosted by someone else
- OpenClaw is the *platform* you run yourself, and you plug your chosen model/provider into it

OpenClaw's superpower is not "a smarter model." It's that it can:
- live **where you already talk** (WhatsApp, Telegram, Slack, Discord, iMessage, and more)
- stay **always-on** (Gateway service)
- keep **state** (sessions, memory, policies)
- optionally **take actions** (tools, device nodes)

Official overview (good high-level read): https://docs.openclaw.ai

---

## The one-sentence mental model

> **OpenClaw is a self-hosted Gateway that connects chat apps to an AI agent that can reason and (optionally) act.**

When you use GLM-5.0 as your model, you get Zhipu AI's powerful reasoning and code generation capabilities combined with OpenClaw's flexible deployment options.

---

## The cast of characters (with analogies)

### 1) The Gateway = "your assistant's home base"

The **Gateway** is a long-running process that you run on a machine you control.

It:
- receives inbound messages from chat platforms (channels)
- decides what should happen (routing + policy)
- runs the agent turn (calls your GLM-5.0 model via Zhipu AI)
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
- Slack
- and many more via plugins

Channels normalize "incoming message events" into a common internal shape.

Docs: https://docs.openclaw.ai/channels

### 3) Sessions = "conversation memory (by default) on disk"

A **session** is a conversation thread with state:
- chat history (transcripts)
- metadata (who/where it came from)
- optional memory indexes

Sessions live on disk under your state directory (usually `~/.openclaw/`).

Docs: https://docs.openclaw.ai/concepts/session

### 4) The agent = "the brain (GLM-5.0 model + rules + tool policy)"

The agent is where your AI model (GLM-5.0 via Zhipu AI) is actually called.

OpenClaw supplies:
- system prompt templates
- history/context
- safety wrappers
- tool availability rules

Docs: https://docs.openclaw.ai/concepts/agent

### 5) Tools = "hands" (powerful; risky)

Tools let the model do more than output text.

Depending on what you enable, tools can include:
- web fetch/search
- browser automation
- cron/automation
- exec or node/device invocations

Tools are where most real-world risk comes from.

Docs: https://docs.openclaw.ai/tools

### 6) Nodes/devices = "peripherals"

Nodes are devices (macOS/iOS/Android) that can connect to the Gateway and offer device-local capabilities.

Examples:
- camera
- audio input
- canvas/webviews
- (on macOS) remote execution with approvals

Docs: https://docs.openclaw.ai/nodes

---

## What OpenClaw is great for

### Personal / small-team assistant in chat

With GLM-5.0's improved reasoning, these tasks work especially well:
- "Summarize the last 200 messages in this group."
- "Draft a reply in my tone."
- "Keep track of ongoing tasks."
- "Generate and debug code for this project."
- "Analyze this document and extract key points."

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

OpenClaw keeps state locally, but your chosen model provider (Zhipu AI) still receives the prompts you send for inference unless you run a local model.

---

## Why GLM-5.0?

GLM-5.0 is Zhipu AI's latest model series. It brings significant improvements over GLM-4.7:

1. **Larger context window** - ~200K tokens vs ~128K in GLM-4.7
2. **Better reasoning** - Multi-step chain-of-thought for complex problems
3. **Improved code generation** - Fewer syntax errors, better structure
4. **Native function calling** - Better tool use integration with OpenClaw
5. **Stronger multilingual** - Better performance for Chinese and mixed-language content

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

Next: [Glossary](./glossary.md)
