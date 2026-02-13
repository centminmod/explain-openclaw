# Architecture (technical, tied to this repo)

## Table of contents (Explain OpenClaw - GLM-5.0 Edition)

- [Home (README)](../README.md)
- Plain English
  - [What is OpenClaw?](../01-plain-english/what-is-clawdbot.md)
  - [Glossary](../01-plain-english/glossary.md)
- Technical
  - [Architecture](./architecture.md)
  - [Repo map](./repo-map.md)
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

This page explains how the OpenClaw codebase is organized and how a message becomes a response when using GLM-5.0 as your model provider.

Sources verified against:
- `../docs/index.md` (high-level)
- `../docs/gateway/index.md` (Gateway runbook)
- `../docs/gateway/security.md`
- `../src/gateway/server.impl.ts` (Gateway server startup + config validation)
- `../src/providers/` (Provider integrations including GLM/Z.AI)
- `../src/agents/pi-embedded-runner.ts` (Agent turn execution)

---

## High-level data flow (with GLM-5.0)

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
Z.AI API (GLM-5.0 model)
  - Function calling support
  - Streaming responses
  - ~200K token context
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
- Core channel implementations are in per-channel folders (e.g. `src/telegram/`, `src/discord/`, `src/imessage/`, `src/web/`, etc.).
- Shared channel logic + routing helpers live in `src/channels/`.

### Auto-reply / agent turns
- The "reply pipeline" is under `src/auto-reply/`.
- `src/auto-reply/reply/agent-runner.ts` is a central orchestrator for running a turn, managing session updates, typing signals, tool output emission rules, follow-ups, etc.

### Config schema
- Types are exported from `src/config/types.ts` (split into many `types.*.ts` files).
- The docs describe strict schema validation and how the UI uses it.

### Providers

GLM/Z.AI provider integration is in `src/providers/`:

| File | Purpose |
|------|---------|
| `src/providers/together-models.ts` | Model definitions and aliases |
| `src/agents/zai.live.test.ts` | Z.AI integration tests |
| `src/infra/provider-usage.*.ts` | Provider usage tracking and metrics |
| `src/commands/auth-choice.*.ts` | Provider selection CLI commands |

---

## Message lifecycle (detailed)

Below is a conceptual pipeline for GLM-5.0. Exact details vary by channel.

1) **Ingress**
- A channel adapter receives an event (DM/group message, attachment, mention).

2) **Identity + authorization**
- Resolve sender identity.
- Apply DM policy (pairing/allowlist/open/disabled).
- Apply group policy (mention gating, allowlists, command gating).

3) **Routing decision**
- Determine the target agent/session key.
- In many configs, DMs collapse into a "main" session; groups are isolated.

4) **Context assembly**
- Load transcript/session store entries from disk.
- Build the agent prompt context (templates + system prompt + history + attachments).

5) **Model call (agent turn with GLM-5.0)**
- Call the Z.AI provider with the GLM-5.0 model.
- GLM-5.0 supports function calling - the model can request structured tool invocations.
- Stream output events.
- If the model requests tools, invoke them via the Gateway's tool execution policy.

6) **Response delivery**
- Convert agent output into channel-specific messages.
- Apply reply threading rules, chunking, etc.
- Send back to the channel.

7) **Persistence**
- Append transcript and metadata updates to disk.

---

## GLM-5.0 specific architecture

### Function calling integration

GLM-5.0 uses structured function calling rather than unstructured tool patterns:

```typescript
// Tool definition format for GLM-5.0
interface GLMTool {
  name: string;
  description: string;
  parameters?: Record<string, any>;
}

// GLM-5.0 requests function calls like:
{
  "name": "web_search",
  "arguments": {
    "query": "latest OpenClaw documentation"
  }
}
```

This is handled in:
- `src/agents/pi-embedded-runner.ts` - Main agent runner
- `src/agents/pi-tools.ts` - Tool creation and invocation
- `src/providers/together-models.ts` - Model-specific formatting

### Provider configuration

GLM models are configured through the standard provider system:

```json
{
  "agents": {
    "defaults": {
      "provider": "zai",
      "model": "glm-5.0"
    }
  }
}
```

Source: `src/config/types.agent-defaults.ts` for schema

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
- model auth profiles: `~/.openclaw/openclaw/agents/<agentId>/agent/auth-profiles.json`
- transcripts: `~/.openclaw/agents/<agentId>/sessions/*.jsonl`

Docs: https://docs.openclaw.ai/gateway/security

---

## Why "security" is mostly configuration

Most security failures are not exotic. They are:
- open DMs/groups
- enabled high-power tools
- exposed Gateway surface without auth
- untrusted plugin installation

That's why `openclaw security audit` exists and why config validation is strict.

Docs: https://docs.openclaw.ai/gateway/security
