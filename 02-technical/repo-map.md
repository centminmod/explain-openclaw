# Repo map (where to look in code)

## Table of contents (Explain OpenClaw)

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
- Deployment
  - [Standalone Mac mini](../03-deploy/standalone-mac-mini.md)
  - [Isolated VPS](../03-deploy/isolated-vps.md)
  - [Cloudflare Moltworker](../03-deploy/cloudflare-moltworker.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

This is a “practical navigation guide” for new contributors/readers.

> Tip: don’t start by reading everything. Start from the Gateway and follow the path of one message.

---

## Top-level directories

- `src/` — TypeScript source
- `docs/` — canonical documentation (the source of truth)
- `extensions/` — plugins (workspace packages)
- `apps/` — macOS/iOS/Android apps
- `dist/` — built output

---

## Key entrypoints

### CLI
- `src/entry.ts` — CLI entry (spawns/sets env, then loads `src/cli/run-main.js`)
- `src/cli/` — CLI command definitions

### Gateway
- `src/gateway/server.impl.ts` — Gateway startup, config validation/migration, runtime wiring
- `src/gateway/server-ws-runtime.ts` (and siblings) — WS server + RPC handler plumbing
- `docs/gateway/index.md` — runbook that matches how it operates in prod

### Channels
- `src/channels/` — shared channel logic (identities, allowlists, gating, registry)
- per-channel folders: `src/telegram`, `src/discord`, `src/slack`, `src/signal`, `src/imessage`, `src/web`, etc.
- `docs/channels/` — channel documentation

### Agent turns
- `src/auto-reply/` — reply pipeline
- `src/auto-reply/reply/agent-runner.ts` — core agent-turn orchestrator

### Security
- `src/security/` — security-related logic (audits, policy)
- `docs/gateway/security.md` — operator-facing threat model + checklist

---

## Useful ripgrep searches

From repo root:

- Find where the Gateway is started:
  - search: `startGatewayServer`
  - file: `src/gateway/server.impl.ts`

- Find security audit logic:
  - search: `security audit`
  - docs: `docs/gateway/security.md`, CLI docs in `docs/cli/security.md`

- Find pairing:
  - search: `pairing` in `src/pairing/` and `docs/start/pairing.md`

- Find a specific channel’s allowlist behavior:
  - search `allowFrom` or `dmPolicy` in the channel config types (`src/config/types.*.ts`) and channel runtime.

---

## How to “trace one message”

1) Pick a channel (e.g., Telegram).
2) Find its monitor/adapter in `src/telegram/`.
3) Follow where it emits a normalized event into shared channel/routing logic.
4) Track how the Gateway selects a session key.
5) Jump into `src/auto-reply/reply/agent-runner.ts` to see how the agent turn is run.
6) Track how the response is chunked/sent back.

---

## Docs you should keep open while reading code

- https://docs.openclaw.ai/start/getting-started
- https://docs.openclaw.ai/gateway
- https://docs.openclaw.ai/gateway/security
- https://docs.openclaw.ai/gateway/configuration
- https://docs.openclaw.ai/help/faq
