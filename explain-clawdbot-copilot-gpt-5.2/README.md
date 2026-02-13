# Clawdbot (beginner guide, Copilot GPT‑5.2)

This folder is a **beginner-oriented explanation** of the Clawdbot codebase you cloned.
It is written for someone new to “agent frameworks” and focuses on:

- **What Clawdbot is** (plain English)
- **What it can do** (capabilities + boundaries)
- **How it works** (technical overview tied to this repo)
- **How to install + configure safely** (high privacy / least privilege)
- Two practical setups:
  - **Standalone Mac mini** (local-first, safest default)
  - **Isolated VPS** (remote gateway with tight access controls)

> Authoring note: this guide cites source/docs paths in this repository and points to the official docs site for deeper reading.

## Start here

1) **What is Clawdbot?** (plain English)
- [Plain-English overview](./01-plain-english/what-is-clawdbot.md)
- [Key concepts glossary](./01-plain-english/glossary.md)

2) **How it works** (technical)
- [Architecture overview](./02-technical/architecture.md)
- [Repo map (where to look in code)](./02-technical/repo-map.md)

3) **Install + first chat**
- [Install and onboarding (fast path)](./03-install/install-and-onboard.md)
- [Running from source (dev)](./03-install/from-source.md)

4) **Privacy + safety (high priority)**
- [Threat model for beginners](./04-privacy-safety/threat-model.md)
- [Hardening checklist (recommended defaults)](./04-privacy-safety/hardening-checklist.md)
- [Example “high privacy” config](./04-privacy-safety/high-privacy-config.example.json5.md)

5) **Usage cases**
- [Standalone Mac mini (local-first)](./05-use-cases/mac-mini-standalone.md)
- [Isolated VPS gateway (remote)](./05-use-cases/vps-isolated.md)

6) **Reference**
- [Useful commands (copy/paste)](./99-reference/commands.md)
- [Where Clawdbot stores state on disk](./99-reference/state-on-disk.md)

## Security note (Issue #1796)

The automated security report in **GitHub Issue #1796** is a mix of real risks, “true but by design” tradeoffs, and some overstated items.
This guide’s practical takeaway is: treat your **state dir** as sensitive, keep the gateway **least-privilege / loopback-first**, and use the built-in security tooling.

- Issue: https://github.com/clawdbot/clawdbot/issues/1796
- **Accurate (risk depends on threat model):** some credentials/tokens are stored on disk as JSON with restrictive permissions (e.g. `0o600`) but **without encryption at rest** (see e.g. `src/infra/device-auth-store.ts`, `src/agents/auth-profiles/*`). If your machine or backups are compromised, those tokens can be exfiltrated.
- **Mostly mitigated / overstated:** OAuth `state` handling is validated in the local callback path for Gemini CLI (state mismatch rejects the callback), and Qwen uses a device + PKCE flow rather than a browser redirect callback.
- **Config-footgun (but off by default):** the voice-call extension can skip webhook signature verification only when explicitly enabled in config (`skipSignatureVerification`, default `false`); docs also label this as dev-only.
- **“Hardcoded client secret” context:** `extensions/google-antigravity-auth` includes an OAuth client secret encoded in source; for public/native OAuth clients this is commonly treated as **non-secret** (still: rotate if you suspect it was meant to be private).

If you’re hardening a deployment, start with the official security docs and run `clawdbot security audit` / `clawdbot security fix`.

## Security note (Medium audit article, Jan 2026)

A third-party Medium post claims eight “critical zero-days”. I extracted the saved article and spot-checked each claim against the **current** codebase; the main pattern is that several items assume an attacker already has **operator/admin** gateway access (in which case they can already execute arbitrary tools), while one major SSRF claim appears **outdated** versus Clawdbot’s pinned-DNS dispatcher.

- Article: https://saadkhalidhere.medium.com/why-clawdbot-is-a-bad-idea-critical-zero-days-found-in-my-audit-full-report-634602cb053f
- **(1) “RCE via config.patch → sandbox.docker.setupCommand”**: `config.patch` is real (`src/gateway/server-methods/config.ts`) and `setupCommand` does execute as `sh -lc` inside the sandbox container (`src/agents/sandbox/docker.ts`). The missing nuance is that `config.*` methods are **admin-scoped** at the gateway layer (`src/gateway/server-methods.ts`), so this is primarily a *“if admin is compromised, attacker can persist config”* finding (still worth hardening by keeping admin tokens tight and sandbox scope minimal).
- **(2) “Arbitrary write via nodes:screen_record outPath”**: accurate — `nodes` accepts a user-provided `outPath` and writes decoded bytes there (`src/agents/tools/nodes-tool.ts` → `src/cli/nodes-screen.ts`). Default behavior writes to a temp path, but if you allow `outPath` from untrusted prompts/users, it becomes a write primitive.
- **(3) “Arbitrary read via logs.tail traversal”**: partially accurate, but threat model is narrower. `resolveLogFile()` does `stat(file)` and returns the provided path if it exists (`src/gateway/server-methods/logs.ts`), however `logs.tail` reads **the configured log file** (`getResolvedLoggerSettings().file`), not an arbitrary request-supplied path; to turn it into arbitrary read you’d need to first change `logging.file` in config (again, admin-scoped).
- **(4) “SSRF via DNS rebinding in web-fetch”**: not accurate for current code. `web-fetch` resolves and then uses an Undici dispatcher with a **pinned DNS lookup** (`src/infra/net/ssrf.ts`, used by `src/agents/tools/web-fetch.ts`), which is specifically intended to close the DNS-rebinding TOCTOU gap.
- **(5) “Approval bypass: exec.approval.resolve has no RBAC”**: not accurate as written; gateway methods are scope-checked, and `exec.approval.*` requires `operator.approvals` (or `operator.admin`) (`src/gateway/server-methods.ts`). If you give the agent an admin/operator token, it can approve its own actions — that’s a deployment decision, not a hidden bypass.
- **(6) “Field shifting attack on device auth token format”**: not supported by current implementation. The “pipe-delimited payload” (`src/gateway/device-auth.ts`) is not parsed by splitting; instead the server *rebuilds the payload* and verifies a device signature over it (`src/gateway/server/ws-connection/message-handler.ts`).
- **(7) “Regex blacklist → shell injection”**: the referenced blacklist exists (`src/infra/exec-safety.ts`) but it is used to validate *executable names/paths in config*, not to “sanitize arbitrary shell”. The actual `exec` tool uses a separate approvals/allowlist pipeline, and should be treated as dangerous-by-design unless locked down.
- **(8) “RCE via env var injection (LD_PRELOAD, etc.)”**: nuance required. The `exec` tool merges user-provided env into the process env (`src/agents/bash-tools.exec.ts`), which is powerful; however the node-host path explicitly blocks `LD_*`/`DYLD_*` and `NODE_OPTIONS` (`sanitizeEnv` in `src/node-host/runner.ts`). If you allow host exec for untrusted callers, you should assume env overrides are part of the attack surface.

## Official docs (recommended)

- Getting started: https://docs.clawd.bot/start/getting-started
- Wizard (onboarding): https://docs.clawd.bot/start/wizard
- Security: https://docs.clawd.bot/gateway/security
- Remote access: https://docs.clawd.bot/gateway/remote
- Tailscale: https://docs.clawd.bot/gateway/tailscale
