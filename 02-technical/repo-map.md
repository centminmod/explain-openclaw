# Repo map (where to look in code)

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

This is a "practical navigation guide" for new contributors/readers.

> Tip: don't start by reading everything. Start from the Gateway and follow the path of one message.

---

## Top-level directories

| Directory | Purpose |
|-----------|---------|
| `src/` | TypeScript source (~50 subdirectories) |
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

- `src/entry.ts` — CLI entry (spawns/sets env, then loads `src/cli/run-main.ts`)
- `src/cli/` — CLI command definitions
- `src/commands/` — command implementations (280 files)

### Gateway

- `src/gateway/server.impl.ts` — Gateway startup, config validation/migration, runtime wiring
- `src/gateway/server-ws-runtime.ts` (and siblings) — WS server + RPC handler plumbing
- `docs/gateway/index.md` — runbook that matches how it operates in prod

### Channels

- `src/channels/` — shared channel logic (identities, allowlists, gating, registry)
- Per-channel folders with full adapters: `src/telegram/`, `src/discord/`, `src/slack/`, `src/signal/`, `src/imessage/`, `src/web/`, `src/line/`, `src/whatsapp/`
- Config-only channels (no `src/` dir): `googlechat`, `msteams`, `feishu`
- `docs/channels/` — channel documentation (29 files including pairing, routing, groups)

### Agent turns

- `src/auto-reply/` — reply pipeline
- `src/auto-reply/reply/agent-runner.ts` — core agent-turn orchestrator
- `src/agents/` — agent framework (602 files, 10 subdirectories): tools, sandbox, auth profiles, skills, multi-agent

### Routing

- `src/routing/` — dedicated routing module: `session-key.ts`, `resolve-route.ts`, `bindings.ts`

### Security

- `src/security/` — security-related logic (audits, policy, external content wrapping)
- `docs/gateway/security/index.md` — operator-facing threat model + checklist

---

## Major `src/` subdirectories

The `src/` directory contains ~50 subdirectories. Key ones beyond the entrypoints above:

| Directory | Files (approx.) | Purpose |
|-----------|-----------------|---------|
| `src/agents/` | ~602 | Agent framework, tools, sandbox, auth profiles, skills |
| `src/gateway/` | ~257 | Gateway server, WS runtime, RPC handlers, config validation |
| `src/auto-reply/` | ~228 | Reply pipeline, agent turn orchestration |
| `src/cli/` | ~220 | CLI command definitions and parsing |
| `src/commands/` | ~280 | CLI command implementations |
| `src/infra/` | ~265 | Infrastructure: networking, SSRF guards, exec safety, archiving |
| `src/config/` | ~159 | Configuration schema, types, validation, migrations |
| `src/browser/` | ~109 | Browser automation (CDP/Puppeteer) |
| `src/channels/` | ~107 | Shared channel logic, identities, allowlists, registry |
| `src/memory/` | ~81 | Memory/context management, QMD |
| `src/cron/` | ~65 | Cron job scheduling |
| `src/plugins/` | ~57 | Plugin runtime and loading |
| `src/media-understanding/` | ~43 | Image/audio/video understanding via AI |
| `src/tui/` | ~38 | Terminal UI |
| `src/daemon/` | — | Background daemon process |
| `src/hooks/` | — | Lifecycle hooks system |
| `src/media/` | — | Media handling (upload, download, conversion) |
| `src/plugin-sdk/` | — | Plugin SDK for extension authors |
| `src/providers/` | — | LLM provider integrations (Anthropic, OpenAI, Ollama, etc.) |
| `src/routing/` | — | Message routing, session key derivation |
| `src/sessions/` | — | Session management and compaction |
| `src/tts/` | — | Text-to-speech |
| `src/wizard/` | — | Setup wizard |
| `src/logging/` | — | Logging, redaction |
| `src/link-understanding/` | — | URL preview / link understanding |
| `src/node-host/` | — | Node.js host for sandboxed execution |
| `src/pairing/` | — | Device pairing flow |
| `src/process/` | — | Process management |
| `src/terminal/` | — | Terminal integration |
| `src/types/` | — | Shared TypeScript types |
| `src/utils/` | — | General utilities |
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

## Useful ripgrep searches

From repo root:

- **Find where the Gateway is started:**
  - search: `startGatewayServer`
  - file: `src/gateway/server.impl.ts`

- **Find security audit logic:**
  - search: `security audit`
  - docs: `docs/gateway/security/index.md`, CLI docs in `docs/cli/security.md`

- **Find pairing:**
  - search: `pairing` in `src/pairing/` and `docs/channels/pairing.md`

- **Find a specific channel's allowlist behavior:**
  - search `allowFrom` or `dmPolicy` in channel config types (`src/config/types.*.ts` — 30 files) and channel runtime

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
7. Jump into `src/auto-reply/reply/agent-runner.ts` to see how the agent turn is run.
8. For multi-agent flows, `src/agents/` orchestrates tool use, sub-agents, and sandbox interactions.
9. Track how the response is chunked/sent back.

---

## Docs you should keep open while reading code

- https://docs.openclaw.ai/start/getting-started
- https://docs.openclaw.ai/gateway
- https://docs.openclaw.ai/gateway/security
- https://docs.openclaw.ai/gateway/configuration
- https://docs.openclaw.ai/help/faq

---

## Documentation Coverage

This section tracks how well the `explain-clawdbot/` documentation covers the codebase. Snapshot verified on 2026-02-21.

### Coverage by Area

| Area | Files | Last Verified | Coverage | Notes |
|------|-------|---------------|----------|-------|
| CLI Commands | 41 top-level commands (40 non-legacy) | 2026-02-21 | ✅ Complete | `01-plain-english/cli-commands.md` |
| Gateway Config | 31 root + 14 gateway fields | 2026-02-21 | ✅ Complete | `99-reference/commands-and-troubleshooting.md` |
| Security | 15 docs + 105 hardening logs + 261 PR refs + 95 issue refs | 2026-02-21 | ✅ Complete | `08-security-analysis/` |
| Architecture | Core paths | 2026-02-21 | ✅ Complete | `02-technical/architecture.md` |
| Deployment | 4 runbooks | 2026-02-21 | ✅ Complete | `03-deploy/` |
| Privacy/Safety | 4 guides | 2026-02-21 | ✅ Complete | `04-privacy-safety/` |

### Known Gaps

| Gap | Impact | Priority |
|-----|--------|----------|
| Browser automation (CDP) | Medium | Low (security-irrelevant) |
| Plugin SDK | Low | Low |
| TUI internals | Low | Low |
| Provider integrations | Medium | Medium (may have security implications) |

### Verification Schedule

| Skill | Frequency | Scope |
|-------|-----------|-------|
| `doc-health-check` | Weekly | Single-file drift audit (report-only) |
| `align-docs-upstream` | As needed | Apply targeted doc fixes after drift is found |
