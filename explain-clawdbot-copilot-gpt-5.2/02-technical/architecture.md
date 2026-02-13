# Architecture overview (technical)

This repo implements Clawdbot as a **Gateway-centric control plane**.

High-level flow (from `README.md`):

```
WhatsApp/Telegram/Slack/Discord/...  →  Gateway (ws://127.0.0.1:18789)  →  Agent (model + tools)  →  reply back
```

## Key runtime components

### 1) CLI binary (`clawdbot`)
- Defined in `package.json` as `bin.clawdbot = dist/entry.js`.
- In development, the repo runs TypeScript via `pnpm clawdbot ...` (see `README.md` “From source”).

### 2) Gateway
- Runs as a background service (launchd/systemd) or foreground process.
- Owns:
  - channel connections
  - session storage
  - auth profiles
  - routing
  - tool execution (and/or sandboxing)

Docs explain it as the control plane: `README.md` + `docs/gateway/*`.

### 3) Channels (adapters)
Each channel has a protocol adapter that converts “incoming message events” into a normalized internal format.
Documentation lives under `docs/channels/*`.

### 4) Agent runtime
The agent is invoked by the Gateway to produce:
- messages
- tool calls
- structured actions

Docs: https://docs.clawd.bot/concepts/agent and https://docs.clawd.bot/concepts/models

### 5) Web surfaces
The Gateway also serves a Control UI + WebChat.
This is useful for:
- first-run verification without connecting any chat app
- local admin actions (pairing approvals, settings)

Docs: https://docs.clawd.bot/web

## Where to look in code (entrypoints)

- CLI / command wiring: `src/cli/`
- Commands: `src/commands/`
- Gateway implementation: `src/gateway/`
- Channel implementations: `src/telegram`, `src/discord`, `src/slack`, `src/signal`, `src/imessage`, `src/web`, plus others
- Config schema + loading: `src/config/`

Next: [Repo map](./repo-map.md).
