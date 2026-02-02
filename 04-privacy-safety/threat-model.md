# Threat model (beginner-friendly)

## Table of contents (Explain OpenClaw)

- [Home (README)](../README.md)
- Plain English
  - [What is OpenClaw?](../01-plain-english/what-is-clawdbot.md)
  - [Glossary](../01-plain-english/glossary.md)
- Technical
  - [Architecture](../02-technical/architecture.md)
  - [Repo map](../02-technical/repo-map.md)
- Privacy + safety
  - [Threat model](./threat-model.md)
  - [Hardening checklist](./hardening-checklist.md)
- Deployment
  - [Standalone Mac mini](../03-deploy/standalone-mac-mini.md)
  - [Isolated VPS](../03-deploy/isolated-vps.md)
  - [Cloudflare Moltworker](../03-deploy/cloudflare-moltworker.md)
- Optimizations
  - [Cost + token optimization](../06-optimizations/cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

This page explains *why* privacy and safety configuration matters for OpenClaw.

If you only take one idea from this:

> Treat the **Gateway host** (and anything it can reach) as the security boundary.

---

## Why OpenClaw is "different-risk" than a normal chat bot

OpenClaw can be configured to do much more than respond with text.

Depending on what you enable, it can:
- send outbound messages on real messaging accounts
- fetch or browse untrusted websites
- run scheduled jobs (automation)
- invoke tools that touch files/network
- invoke node/device actions (camera/audio/canvas)

That means an attacker doesn’t need to “hack” the codebase to cause harm.
Often they just need to:

1) get a message (or web page) into the agent’s context, and
2) convince the model to take an unsafe action

This is the practical form of **prompt injection**.

Official security doc (source of truth): https://docs.openclaw.ai/gateway/security

---

## Threat surfaces

### 1) Inbound messages (DMs + groups)
- strangers DMing your bot
- untrusted participants in a group channel
- “mention baiting” (tagging the bot with instructions)

Mitigations:
- DM pairing / allowlists
- group allowlists + require-mention defaults
- command gating/access groups

### 2) Untrusted content the bot reads
Even if only you can message the bot, the bot may read untrusted content:
- web pages (fetch/search/browser)
- attachments (PDFs, docs)
- pasted logs
- forwarded messages

Mitigations:
- treat untrusted content like you would treat an email attachment
- use a “reader” agent with tools disabled
- avoid combining “reads random web pages” with “can run exec”

### 3) Network exposure
If the Gateway is reachable from more networks than intended, the risk goes up fast.

Mitigations:
- keep `gateway.bind: "loopback"` where possible
- use SSH tunnel / Tailscale Serve
- require auth (token/password) for non-loopback binds
- be careful with reverse proxies; configure trusted proxies

### 4) Local disk + secrets
OpenClaw stores transcripts and credentials on disk under `~/.openclaw/`.
If another user/process on the host can read that directory, privacy is gone.

Mitigations:
- file permissions (audit fixes these)
- avoid syncing `~/.openclaw` to cloud drives
- OS-level hardening (separate user, disk encryption)

### 5) Supply chain / plugins
Plugins run in-process and plugin installation can execute code during install.

Mitigations:
- only install trusted plugins
- pin versions
- inspect code on disk
- keep an allowlist (if supported)

---

## A practical attacker model

Ask: “who could realistically cause trouble?”

- Random internet users (if you open DMs/groups)
- People in your group chats
- A compromised plugin/package
- A compromised model provider account / API key
- A compromised host (VPS intrusion, stolen laptop)

---

## Core principle: access control before intelligence

OpenClaw's docs emphasize an ordering that works in practice:

1) **Identity first** — who is allowed to trigger the bot (pairing/allowlists)
2) **Scope next** — what the bot is allowed to do (tool policy/sandboxing/nodes)
3) **Model last** — assume the model can be manipulated; design so manipulation has limited blast radius

---

## What "high privacy" means in OpenClaw terms

High privacy usually implies:
- Gateway host is locked down
- Gateway binds to localhost (or tailnet-only)
- the Control UI is not public
- only you (or a small allowlist) can trigger the bot
- tool surface area is minimal
- sensitive logs are redacted
- credentials are stored in env vars or encrypted stores where possible

Next: [Hardening checklist](./hardening-checklist.md)
