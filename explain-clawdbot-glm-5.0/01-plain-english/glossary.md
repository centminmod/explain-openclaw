# Glossary (plain English)

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
- Installation
  - [Install and onboarding](../03-install/install-and-onboard.md)
  - [Build from source](../03-install/from-source.md)
- Deployment
  - [Standalone Mac mini](../05-use-cases/mac-mini-standalone.md)
  - [Isolated VPS](../05-use-cases/vps-isolated.md)

---

This glossary is intentionally biased toward words you'll see in CLI output and docs.

## Agent

The "brain" that turns an incoming message + context into a response. In practice it's:
- your chosen model (GLM-5.0 via Z.AI provider)
- a system prompt + templates
- tool availability rules
- session/memory behavior

Docs: https://docs.openclaw.ai/concepts/agent

## Allowlist

A list of identities allowed to trigger bot (DMs) or allowed to interact in groups.

Security note: allowlists are often *real* security boundary in messaging bots.

Docs: https://docs.openclaw.ai/gateway/security

## Bind (gateway.bind)

Where Gateway listens:
- `loopback` = localhost only (safest default)
- `lan` = your LAN interfaces (requires auth)
- `tailnet` = bind only to Tailscale IP (requires Tailscale)

Docs: https://docs.openclaw.ai/gateway/remote and https://docs.openclaw.ai/gateway/tailscale

## Channel

A connector for a messaging surface: WhatsApp/Telegram/Discord/iMessage/etc.

Docs: https://docs.openclaw.ai/channels

## Control UI / Dashboard

The web interface served by Gateway (on the same port as the WebSocket).

Docs: https://docs.openclaw.ai/web/dashboard

## Gateway

The long-running process that owns:
- channel connections
- routing
- sessions
- plugins
- tool execution policy
- node/device pairing

Docs: https://docs.openclaw.ai/gateway

## GLM-5.0

General Language Model 5.0 - Z.AI's flagship model series with improved reasoning, larger context window (~200K tokens), and better code generation capabilities compared to GLM-4.7.

**Provider Documentation:** https://docs.openclaw.ai/providers/zai

## Pairing

Explicit owner approval.

Used for:
- DM pairing (who may DM bot)
- device pairing (which nodes may connect)

Docs: https://docs.openclaw.ai/start/pairing

## Plugin / extension

Extra code that runs **in-process** with Gateway to add channels/tools/features.

Treat it like installing code into your assistant.

Docs: https://docs.openclaw.ai/plugin and https://docs.openclaw.ai/gateway/security

## Session

A stored conversation thread (history + metadata). By default sessions are stored on disk as JSONL.

Docs: https://docs.openclaw.ai/concepts/session

## Tool

A capability the GLM-5.0 model can invoke (web fetch/search, browser automation, cron, exec, node calls).

Docs: https://docs.openclaw.ai/tools

## Node / device

A companion device connected to Gateway (iOS/Android/macOS/headless nodes).

Docs: https://docs.openclaw.ai/nodes

## Provider

The external AI service you connect to for model inference. For GLM-5.0, this is **Z.AI** (`api.z.ai`).

OpenClaw supports multiple providers - you choose during onboarding or via configuration.

## Function Calling (GLM-specific)

GLM-5.0 has native support for function calling - the ability to structuredly request tool invocations from the model. This is different from unstructured tool use patterns in some older models.

When GLM-5.0 uses function calling:
1. Model analyzes the request and available tools
2. Model outputs a structured function call request
3. Gateway executes the tool and returns results
4. Results are fed back to the model for final response

## Z.AI

The API platform for GLM models. Z.AI provides REST APIs for GLM and uses API keys for authentication. OpenClaw uses the `zai` provider with a `ZAI_API_KEY`.

> **Opus 4.6 correction:** Originally listed as "Zhipu AI" with `open.bigmodel.cn` URLs. Zhipu AI (智谱AI) is the parent research company that developed the GLM model family; Z.AI (`api.z.ai`) is the international API platform used by OpenClaw, with infrastructure in Singapore.

**Provider Docs:** https://docs.openclaw.ai/providers/zai
