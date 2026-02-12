# Architecture (technical, tied to this repo)

## Table of contents (Explain OpenClaw)

- [Home (README)](../README.md)
- Plain English
  - [What is OpenClaw?](../01-plain-english/what-is-clawdbot.md)
  - [Glossary](../01-plain-english/glossary.md)
  - [CLI commands](../01-plain-english/cli-commands.md)
- Technical
  - [Architecture](./architecture.md)
  - [Repo map](./repo-map.md)
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

This page explains how the OpenClaw codebase is organized and how a message becomes a response.

Sources verified against:
- `../docs/index.md` (high-level)
- `../docs/gateway/index.md` (Gateway runbook)
- `../docs/gateway/security.md`
- `../src/gateway/server.impl.ts` (Gateway server startup + config validation)
- `../src/auto-reply/reply/agent-runner.ts` (agent turn execution)

---

## High-level data flow

```
Messaging apps (WhatsApp/Telegram/Discord/iMessage/...)  
        │
        ▼
Gateway (single always-on process)
  - channel ingress/egress
  - routing + policy checks
  - sessions (disk)
  - agent turns
  - tool calls
        │
        ▼
AI provider(s) (Anthropic/OpenAI/etc.) OR local model endpoint
        │
        ▼
Back to the channel
```

OpenClaw is **Gateway-centric**. Most things you do (status, logs, pairing, sending, agent runs) talk to the Gateway via its WebSocket RPC.

---

## The Gateway is the control plane

The Gateway:
- binds a single port (default `18789`) and multiplexes:
  - WebSocket control plane
  - Control UI / dashboard
  - optional HTTP endpoints (`/v1/chat/completions`, `/v1/responses`, `/tools/invoke`)
- owns channel connections (WhatsApp Web session, Telegram bot long-poll, etc.)
- owns local state (config, credentials, transcripts)

### Config validation as a safety feature
In `src/gateway/server.impl.ts`, the Gateway:
- reads config snapshot
- migrates legacy config entries when possible
- refuses to start on invalid schema

This is intentional: unknown keys and malformed values are treated as unsafe.

Docs: https://docs.openclaw.ai/gateway/configuration

---

## Key modules in this repo (where to look)

### CLI
- Entry script: `src/entry.ts` → loads CLI main.
- Command wiring: `src/cli/`.
- Gateway CLI docs: `docs/cli/gateway.md`.

### Gateway
- Gateway start + core runtime: `src/gateway/` (entry in `src/gateway/server.impl.ts`).
- WebSocket runtime + methods live under `src/gateway/server-*` modules.

### Channels
- Core channel implementations are in per-channel folders (e.g. `src/telegram`, `src/discord`, `src/imessage`, `src/web`, etc.).
- Shared channel logic + routing helpers live in `src/channels/`.

### Auto-reply / agent turns
- The “reply pipeline” is under `src/auto-reply/`.
- `src/auto-reply/reply/agent-runner.ts` is a central orchestrator for running a turn, managing session updates, typing signals, tool output emission rules, follow-ups, etc.

### Config schema
- Types are exported from `src/config/types.ts` (split into many `types.*.ts` files).
- The docs describe strict schema validation and how the UI uses it.

---

## Message lifecycle (detailed)

Below is a conceptual pipeline. Exact details vary by channel.

1) **Ingress**
- A channel adapter receives an event (DM/group message, attachment, mention).

2) **Identity + authorization**
- Resolve sender identity.
- Apply DM policy (pairing/allowlist/open/disabled).
- Apply group policy (mention gating, allowlists, command gating).

3) **Routing decision**
- Determine the target agent/session key.
- In many configs, DMs collapse into a “main” session; groups are isolated.

4) **Context assembly**
- Load transcript/session store entries from disk.
- Build the agent prompt context (templates + system prompt + history + attachments).

5) **Model call (agent turn)**
- Call the selected provider/model (with fallback logic if configured).
- Stream output events.
- If the model requests tools, invoke them via the Gateway’s tool execution policy.

6) **Response delivery**
- Convert agent output into channel-specific messages.
- Apply reply threading rules, chunking, etc.
- Send back to the channel.

7) **Persistence**
- Append transcript and metadata updates to disk.

---

## Ports (what listens where)

Default base port: `18789`.

On that port:
- WebSocket control plane
- Dashboard (HTTP)

Related derived ports may exist (browser control, canvas host), depending on configuration.

Docs:
- https://docs.openclaw.ai/gateway (service runbook)
- https://docs.openclaw.ai/help/faq (port precedence)

---

## Where state lives (operator view)

The most important directory is your state dir:
- default: `~/.openclaw/`
- per-profile: `~/.openclaw-<profile>/`

Common paths (see official "credential storage map"):
- config: `~/.openclaw/openclaw.json`
- model auth profiles: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- transcripts: `~/.openclaw/agents/<agentId>/sessions/*.jsonl`

Docs: https://docs.openclaw.ai/gateway/security

---

## Why “security” is mostly configuration

Most security failures are not exotic. They are:
- open DMs/groups
- enabled high-power tools
- exposed Gateway surface without auth
- untrusted plugin installation

That's why `openclaw security audit` exists and why config validation is strict.

Docs: https://docs.openclaw.ai/gateway/security
