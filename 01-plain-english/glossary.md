# Glossary (plain English)

## Table of contents (Explain OpenClaw)

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
- Deployment
  - [Standalone Mac mini](../03-deploy/standalone-mac-mini.md)
  - [Isolated VPS](../03-deploy/isolated-vps.md)
  - [Cloudflare Moltworker](../03-deploy/cloudflare-moltworker.md)
- Optimizations
  - [Cost + token optimization](../06-optimizations/cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

This glossary is intentionally biased toward the words you’ll see in the CLI output and docs.

## Agent
The “brain” that turns an incoming message + context into a response. In practice it’s:
- your chosen model (provider + model name)
- a system prompt + templates
- tool availability rules
- session/memory behavior

Docs: https://docs.openclaw.ai/concepts/agent

## Allowlist
A list of identities allowed to trigger the bot (DMs) or allowed to interact in groups.

Security note: allowlists are often the *real* security boundary in messaging bots.

Docs: https://docs.openclaw.ai/gateway/security

## Bind (gateway.bind)
Where the Gateway listens:
- `loopback` = localhost only (safest default)
- `lan` = your LAN interfaces (requires auth)
- `tailnet` = bind only to Tailscale IP (requires Tailscale)

Docs: https://docs.openclaw.ai/gateway/remote and https://docs.openclaw.ai/gateway/tailscale

## Channel
A connector for a messaging surface: WhatsApp/Telegram/Discord/iMessage/etc.

Docs: https://docs.openclaw.ai/channels

## Control UI / Dashboard
The web interface served by the Gateway (on the same port as the WebSocket).

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

## Pairing
Explicit owner approval.

Used for:
- DM pairing (who may DM the bot)
- device pairing (which nodes may connect)

Docs: https://docs.openclaw.ai/start/pairing

## Plugin / extension
Extra code that runs **in-process** with the Gateway to add channels/tools/features.

Treat it like installing code into your assistant.

Docs: https://docs.openclaw.ai/plugin and https://docs.openclaw.ai/gateway/security

## Session
A stored conversation thread (history + metadata). By default sessions are stored on disk as JSONL.

Docs: https://docs.openclaw.ai/concepts/session

## Tool
A capability the model can invoke (web fetch/search, browser automation, cron, exec, node calls).

Docs: https://docs.openclaw.ai/tools

## Node / device
A companion device connected to the Gateway (iOS/Android/macOS/headless nodes).

Docs: https://docs.openclaw.ai/nodes
