# Repo map (where to look in code)

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

This is a "practical navigation guide" for new contributors/readers.

> Tip: don't start by reading everything. Start from Gateway and follow the path of one message.

---

## Top-level directories

| Directory | Purpose |
|-----------|---------|
| `src/` | TypeScript source (~49 subdirectories) |
| `docs/` | Canonical documentation (source of truth) |
| `extensions/` | Plugins (workspace packages) |
| `apps/` | macOS/iOS/Android apps |
| `packages/` | Shared workspace packages |
| `ui/` | Web UI / dashboard |
| `skills/` | Skill definitions |
| `scripts/` | Build, release, and utility scripts |
| `test/` | Integration / E2E tests |
| `Swabble/` | Swabble integration |
| `exports/` | Public API exports |
| `vendor/` | Vendored third-party code |
| `patches/` | Patch files for dependencies |
| `git-hooks/` | Git hook scripts |
| `assets/` | Static assets (images, icons) |

---

## Key entrypoints

### CLI

- `src/entry.ts` — CLI entry (spawns/sets env, then loads `src/cli/run-main.js`)
- `src/cli/` — CLI command definitions
- `src/commands/` — command implementations (186 files)
- Gateway CLI docs: `docs/cli/gateway.md`.

### Gateway

- `src/gateway/server.impl.ts` — Gateway startup, config validation/migration, runtime wiring
- `src/gateway/server-ws-runtime.ts` (and siblings) — WS server + RPC handler plumbing
- `docs/gateway/index.md` — runbook that matches how it operates in prod

### Channels

- `src/channels/` — shared channel logic (identities, allowlists, gating, registry)
- Per-channel folders with full adapters: `src/telegram/`, `src/discord/`, `src/slack/`, `src/signal/`, `src/imessage/`, `src/web/`, `src/line/`, `src/whatsapp/`
- Config-only channels (no `src/` dir): `googlechat`, `msteams`, `feishu`
- `docs/channels/` — channel documentation (28 files including pairing, routing, groups)

### Agent turns

- `src/auto-reply/` — reply pipeline
- `src/auto-reply/reply/agent-runner.ts` — core agent-turn orchestrator
- `src/agents/` — agent framework (316+ files): tools, sandbox, auth profiles, multi-agent

### Routing

- `src/routing/` — dedicated routing module: `session-key.ts`, `resolve-route.ts`, `bindings.ts`

### Security

- `src/security/` — security-related logic (audits, policy, external content wrapping)
- `docs/gateway/security/index.md` — operator-facing threat model + checklist

---

## Major `src/` subdirectories

The `src/` directory contains ~49 subdirectories. Key ones beyond entrypoints above:

| Directory | Files (approx.) | Purpose |
|-----------|-----------------|---------|
| `src/agents/` | ~466 | Agent framework, tools, sandbox, auth profiles, GLM integration |
| `src/browser/` | ~82 | Browser automation (CDP/Puppeteer) |
| `src/commands/` | ~234 | CLI command implementations |
| `src/config/` | ~139 | Configuration schema, types, validation, migrations |
| `src/cron/` | — | Cron job scheduling |
| `src/daemon/` | — | Background daemon process |
| `src/hooks/` | — | Lifecycle hooks system |
| `src/infra/` | ~195 | Infrastructure: networking, SSRF guards, exec safety, archiving |
| `src/media/` | — | Media handling (upload, download, conversion) |
| `src/media-understanding/` | — | Image/audio/video understanding via AI |
| `src/memory/` | — | Memory/context management, QMD |
| `src/plugins/` | — | Plugin runtime and loading |
| `src/plugin-sdk/` | — | Plugin SDK for extension authors |
| `src/providers/` | — | LLM provider integrations (Anthropic, OpenAI, Z.AI, etc.) |
| `src/routing/` | — | Message routing, session key derivation |
| `src/sessions/` | — | Session management and compaction |
| `src/tts/` | — | Text-to-speech |
| `src/tui/` | — | Terminal UI |
| `src/wizard/` | — | Setup wizard |
| `src/logging/` | — | Logging, redaction |
| `src/link-understanding/` | — | URL preview / link understanding |
| `src/node-host/` | — | Node.js host for sandboxed execution |
| `src/pairing/` | — | Device pairing flow |
| `src/process/` | — | Process management |
| `src/terminal/` | — | Terminal integration |
| `src/types/` | — | Shared TypeScript types |
| `src/shared/` | — | Shared cross-module code |
| `src/acp/` | — | ACP (Agent Communication Protocol) |
| `src/canvas-host/` | — | Canvas/drawing host |
| `src/compat/` | — | Compatibility layers |
| `src/docs/` | — | In-app documentation helpers |
| `src/macos/` | — | macOS-specific native integrations |
| `src/markdown/` | — | Markdown processing |
| `src/scripts/` | — | Source-level scripts |
| `src/test-helpers/` | — | Test helper utilities |
| `src/test-utils/` | — | Test utility functions |

---

## GLM / Z.AI specific files

### Provider implementation

| File | Purpose |
|------|---------|
| `src/providers/together-models.ts` | GLM model definitions and aliases |
| `src/agents/zai.live.test.ts` | Z.AI integration tests |
| `src/infra/provider-usage.fetch.zai.ts` | Z.AI usage tracking |
| `src/infra/provider-usage.shared.ts` | Shared provider usage utilities |
| `src/commands/auth-choice.apply.api-providers.ts` | Provider selection CLI |
| `src/commands/auth-choice-options.ts` | Provider configuration options |

### Auth profiles

GLM API keys are stored in auth profiles:
- `src/agents/auth-profiles.ts` — Core auth profile management
- `src/agents/auth-profiles.resolve-auth-profile-order.*.test.ts` — Tests for provider ordering including GLM

---

## Useful ripgrep searches

From repo root:

- **Find where Gateway is started:**
  - search: `startGatewayServer`
  - file: `src/gateway/server.impl.ts`

- **Find security audit logic:**
  - search: `security audit`
  - docs: `docs/gateway/security/index.md`, CLI docs in `docs/cli/security.md`

- **Find GLM/Z.AI provider code:**
  - search: `zai` in `src/providers/`
  - search: `glm-5` or `glm-5.0` for model definitions

- **Find pairing:**
  - search: `pairing` in `src/pairing/` and `docs/channels/pairing.md`

- **Find a specific channel's allowlist behavior:**
  - search `allowFrom` or `dmPolicy` in channel config types (`src/config/types.*.ts` — 29 files) and channel runtime

- **Find ChatType / DM policy enum:**
  - search: `ChatType` — used across channel adapters for group vs DM routing

- **Find text sanitization:**
  - search: `sanitizeUserFacingText` — used in reply normalization and agent tools

- **Find memory search logic:**
  - search: `memorySearch` — in `src/config/` and `src/memory/`

---

## How to "trace one message"

1. Pick a channel (e.g., Telegram).
2. Find its monitor/adapter in `src/telegram/`.
3. Follow where it emits a normalized event into shared channel/routing logic.
4. The `src/routing/` module resolves the route and derives a session key (`session-key.ts`, `resolve-route.ts`).
5. `src/hooks/` may fire at pipeline stages (pre-reply, post-reply, etc.).
6. Track how the Gateway selects a session key.
7. Jump into `src/auto-reply/reply/agent-runner.ts` to see how the agent turn is run with GLM-5.0.
8. For function calling, track how tools are invoked and results returned.
9. Track how the response is chunked/sent back.

---

## Docs you should keep open while reading code

- https://docs.openclaw.ai/start/getting-started
- https://docs.openclaw.ai/gateway
- https://docs.openclaw.ai/gateway/security
- https://docs.openclaw.ai/gateway/configuration
- https://docs.openclaw.ai/help/faq
- https://docs.openclaw.ai/providers/zai (Z.AI provider documentation)
