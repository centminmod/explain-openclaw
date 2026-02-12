<p align="center">
    <picture>
        <img src="logo3a.png" srcset="logo3a.png 1x, logo3.png 2x" alt="OpenClaw" width="440">
    </picture>
</p>

# Explain OpenClaw (formerly Moltbot/Clawdbot) - Integrated Beginner + Technical Guide


## Table of contents

- [What is OpenClaw? (plain English)](./01-plain-english/what-is-clawdbot.md)
- [Glossary](./01-plain-english/glossary.md)
- [CLI commands (plain English)](./01-plain-english/cli-commands.md)
- [What is Moltbook?](./07-moltbook/what-is-moltbook.md)
- [Threat model](./04-privacy-safety/threat-model.md)
- [Hardening checklist](./04-privacy-safety/hardening-checklist.md)
- [High privacy config example](./04-privacy-safety/high-privacy-config.example.json5.md)
- [Detecting OpenClaw requests (for hosting services)](./04-privacy-safety/detecting-openclaw-requests.md)
- [Architecture (technical)](./02-technical/architecture.md)
- [Repo map](./02-technical/repo-map.md)
- [Deployment: Standalone Mac mini](./03-deploy/standalone-mac-mini.md)
- [Deployment: Isolated VPS](./03-deploy/isolated-vps.md)
- [Deployment: Cloudflare Moltworker](./03-deploy/cloudflare-moltworker.md)
- [Deployment: Docker Model Runner](./03-deploy/docker-model-runner.md)
- [Commands + troubleshooting](./99-reference/commands-and-troubleshooting.md)
- **Optimizations:**
  - [Overview](./06-optimizations/README.md)
  - [Resource usage analysis (CPU, memory, disk)](./06-optimizations/resource-usage.md)
  - [Cost + token optimization](./06-optimizations/cost-token-optimization.md)
  - [Model recommendations by function](./06-optimizations/cost-token-optimization.md#model-recommendations-by-function)
- **Security documentation:**
  - [`openclaw security audit` command reference](./08-security-analysis/security-audit-command-reference.md)
  - [Official security advisories (CVEs/GHSAs)](./08-security-analysis/official-security-advisories.md)
  - [Security audit analysis (Issue #1796)](./08-security-analysis/issue-1796-argus-audit.md)
  - [Second security audit (Medium article)](./08-security-analysis/medium-article-audit.md)
  - [Third security audit (ZeroLeeks AI Red Team)](./08-security-analysis/zeroleeks-audit.md)
  - [Post-merge security hardening](./08-security-analysis/post-merge-hardening.md)
  - [Open upstream security issues](./08-security-analysis/open-upstream-issues.md)
  - [Open upstream security PRs](./08-security-analysis/open-upstream-prs.md)
  - [Ecosystem security threats](./08-security-analysis/ecosystem-security-threats.md)
  - [SecurityScorecard STRIKE report analysis](./08-security-analysis/securityscorecard-strike-report.md) *(Feb 2026, 28k+ exposed instances)*
  - [Model poisoning and sleeper agent backdoors](./08-security-analysis/model-poisoning-sleeper-agents.md) *(Feb 2026 Microsoft research)*
  - [Cisco AI Defense skill scanner analysis](./08-security-analysis/cisco-ai-defense-skill-scanner.md) *(Feb 2026, blog post + tool evaluation)*
- [AI model analysis comparison](./08-security-analysis/ai-model-analysis-comparison.md)
- **Worst-case security scenarios:**
  - [Overview](./05-worst-case-security/README.md)
  - [Mac Mini risks](./05-worst-case-security/mac-mini-risks.md)
  - [VPS risks](./05-worst-case-security/vps-risks.md)
  - [Moltworker risks](./05-worst-case-security/moltworker-risks.md)
  - [Cross-cutting vulnerabilities](./05-worst-case-security/cross-cutting.md)
  - [ClawHub marketplace risks](./05-worst-case-security/clawhub-marketplace-risks.md) *(Feb 2026 campaign)*
  - [Skills.sh risks](./05-worst-case-security/skills-sh-risks.md) *(supply chain)*
  - [Prompt injection attacks](./05-worst-case-security/prompt-injection-attacks.md) *(27 examples)*
  - [Misconfiguration examples](./05-worst-case-security/misconfiguration-examples.md)
  - [AI self-misconfiguration](./05-worst-case-security/ai-self-misconfiguration.md)

---

This folder is a **living knowledge base** for the OpenClaw framework — actively maintained documentation that has grown from an initial multi-model AI analysis into a broad reference covering security audits, deployment operations, threat intelligence, and beginner-friendly explanations.

**What you'll find here:**

| Section | What it covers |
|---------|---------------|
| **Plain English** | What OpenClaw is, glossary, "explain it like I'm new" |
| **Technical** | Architecture deep-dive, repo map for contributors |
| **Deployment** | **Mac mini** (local-first), **Isolated VPS** (remote + hardened), **Cloudflare Moltworker** (serverless), **Docker Model Runner** (local AI, zero API cost) |
| **Privacy & Safety** | Threat model, hardening checklist, request fingerprint detection |
| **Security Audits** | Independently verified audit analyses, CVE/GHSA tracking, upstream issue monitoring |
| **Worst-Case Scenarios** | Attack catalogs, prompt injection examples, supply chain threats, incident response |
| **Optimizations** | Resource usage analysis (CPU/memory/disk), cost/token reduction, model routing |
| **AI Model Comparison** | Accuracy benchmarks across five AI models' analyses |

**What started as** a synthesis of five AI models' analyses has expanded through continuous upstream tracking and independent code verification. It reconciles analyses from [Copilot GPT-5.2](./explain-clawdbot-copilot-gpt-5.2/), [Gemini 3.0 Pro](./explain-clawdbot-gemini-3.0-pro/), [GLM 4.7](./explain-clawdbot-glm-4.7/), [Opus 4.5](./explain-clawdbot-opus-4.5/), and [Kimi K2.5](./explain-clawdbot-kilocode-kimi-k2.5/) — with an [accuracy comparison](./08-security-analysis/ai-model-analysis-comparison.md) showing which models verified claims against source code and which accepted them at face value.

> **Repo docs + code win.** Model summaries are supporting material.

---

## What is OpenClaw? (30-second version)

OpenClaw is a **self-hosted AI assistant platform**. You run an always-on process called the **Gateway** on a machine you control (a Mac mini at home or an isolated VPS). The Gateway connects to messaging apps (WhatsApp/Telegram/Discord/iMessage/… via built-in channels + plugins), receives messages, runs an agent turn (the “brain”), optionally invokes tools/devices, and sends responses back.

**Key idea:** your **Gateway host** is the trust boundary. If it’s compromised (or configured too openly), your assistant can be turned into a data-exfil / automation engine.

Official docs starting point:
- https://docs.openclaw.ai/start/getting-started
- https://docs.openclaw.ai/gateway
- https://docs.openclaw.ai/gateway/security

---

## The four deployment scenarios this guide focuses on

1) **Standalone Mac mini (local-first, high privacy)**
- The Gateway runs on a Mac mini you own.
- Default best practice: keep it **loopback-only** (`gateway.bind: "loopback"`) and access it locally.
- Optional remote access should be via **SSH tunnels** or **Tailscale Serve**, not public ports.

2) **Isolated VPS server (remote, locked down)**
- The Gateway runs on a small Linux VPS.
- **Fastest path:** [DigitalOcean 1-Click Deploy](./03-deploy/isolated-vps.md#11-digitalocean-1-click-deploy) pre-configures security hardening automatically.
- Default best practice: keep it **loopback-only** and access it via **SSH tunnel** or **tailnet**.
- Harden the host like any admin system (dedicated user, firewall, patching, log hygiene).

3) **Cloudflare Moltworker (serverless, managed infrastructure)**
- The Gateway runs inside Cloudflare's Sandbox SDK container on their global edge network.
- No hardware to manage; automatic scaling and isolation.
- Uses R2 for persistence, AI Gateway for model routing, Browser Rendering for web automation.
- Proof-of-concept; requires Cloudflare Workers paid plan ($5/month minimum).

4) **Docker Model Runner (local AI, zero API cost)**
- Run LLMs locally via Docker Desktop's Model Runner.
- Zero API costs after initial model download.
- Complete privacy — no data leaves your machine.
- Requires Docker Desktop 4.40+ and compatible hardware (Apple Silicon, NVIDIA GPU, or AMD GPU).

---

## Start here (recommended reading order)

### 1) Plain English
- [What is OpenClaw?](./01-plain-english/what-is-clawdbot.md)
- [What is Moltbook?](./07-moltbook/what-is-moltbook.md)
- [Glossary](./01-plain-english/glossary.md)

### 2) Privacy + safety first (highly recommended)
- [Threat model (beginner-friendly)](./04-privacy-safety/threat-model.md)
- [Hardening checklist (high privacy)](./04-privacy-safety/hardening-checklist.md)
- [High privacy config example](./04-privacy-safety/high-privacy-config.example.json5.md)

### 3) Technical overview (how it works)
- [Architecture (Gateway → channels → agent → tools)](./02-technical/architecture.md)
- [Repo map (where to look in code)](./02-technical/repo-map.md)

### 4) Deployment runbooks
- [Standalone Mac mini (local-first)](./03-deploy/standalone-mac-mini.md)
- [Isolated VPS (remote + locked down)](./03-deploy/isolated-vps.md)
  - [DigitalOcean 1-Click Deploy](./03-deploy/isolated-vps.md#11-digitalocean-1-click-deploy) *(recommended)*
- [Cloudflare Moltworker (serverless)](./03-deploy/cloudflare-moltworker.md)
- [Docker Model Runner (local AI, zero cost)](./03-deploy/docker-model-runner.md)

### 5) Reference
- [Commands + troubleshooting quick reference](./99-reference/commands-and-troubleshooting.md)

---

## Quick start (safe-ish defaults)

The repo strongly recommends using the onboarding wizard; it sets up:
- a working Gateway service (launchd/systemd)
- auth/provider credentials
- safe access defaults (pairing, token)

### Install

Recommended installer:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Alternative:

```bash
npm install -g openclaw@latest
```

### Onboard + install background service

```bash
openclaw onboard --install-daemon
```

### Verify

```bash
openclaw gateway status
openclaw status
openclaw health
```

### Security audit

Three levels of security auditing:

```bash
# Read-only scan of config + filesystem permissions (no network calls)
openclaw security audit

# Everything above + live WebSocket probe of the running gateway
openclaw security audit --deep

# Apply safe auto-fixes first, then run full audit to show remaining issues
openclaw security audit --fix
```

| Flag | What it adds | Modifies system? |
| ------ | ------------- | ----------------- |
| *(none)* | Scans config, filesystem permissions, channel policies, model hygiene, plugin trust, attack surface summary (50+ check IDs across 12 categories) | No — read-only |
| `--deep` | All base checks + live WebSocket probe of running gateway (5 s timeout), verifies auth handshake | No — read-only probe |
| `--fix` | Applies safe fixes **before** running the full audit: `chmod 600/700` on state/config/credentials, flips `groupPolicy open→allowlist`, sets `logging.redactSensitive off→"tools"`. Report shows remaining issues post-fix | Yes — safe defaults only; no destructive changes |

> **Note:** `--fix` runs the fix pass **before** the audit (`src/cli/security-cli.ts:46`), so the report you see reflects the hardened state. Any findings that remain are issues `--fix` cannot auto-resolve.

If you only do one security thing, do this:

```bash
openclaw security audit --fix
```

See the [full command reference](./08-security-analysis/security-audit-command-reference.md) for what each check covers, what `--fix` changes, and which documented issues the audit can and cannot detect.

(Security audit docs: <https://docs.openclaw.ai/gateway/security>)

---

## How to think about OpenClaw (beginner mental model)

OpenClaw is easiest to understand as 6 layers:

1. **Gateway (control plane)** — one long-running process that owns:
   - message ingress/egress
   - sessions + transcripts
   - routing rules
   - plugin loading
   - tool execution policy + sandboxing
   - node/device pairing and invocations

2. **Channels** — adapters from Telegram/WhatsApp/etc. into a normalized message/event shape.

3. **Routing + sessions** — decides which “agent/session” handles which chat.

4. **Agent runtime** — takes context (system prompt + history + attachments), calls your chosen model provider, streams responses, and can request tools.

5. **Tools** — optional capabilities beyond text (web fetch/search, browser control, exec, cron, nodes/devices).

6. **Surfaces** — where you interact:
   - chat apps (WhatsApp/Telegram/…)
   - Control UI dashboard (web)
   - macOS menu bar app

This matters because your security choices mostly reduce to:
- **Who can trigger the agent?** (pairing + allowlists + group policies)
- **What can the agent do once triggered?** (tools/sandboxing/nodes)
- **What can the agent reach?** (network exposure, filesystem access, accounts)

---

## FAQ (Beginner → Intermediate → Advanced)

This FAQ is intentionally long and practical; it’s the “things you’ll actually Google at 2am.”

### Beginner FAQ

#### Q: What should I install this on: my laptop, a Mac mini, a VPS, or Cloudflare?
- **Mac mini (recommended for most privacy-first users):** always-on, easy local access, no cloud exposure by default.
- **VPS (recommended for always-on + remote access):** great uptime, but higher security responsibility. [DigitalOcean 1-Click](./03-deploy/isolated-vps.md#11-digitalocean-1-click-deploy) handles hardening automatically.
- **Cloudflare Moltworker (low-maintenance serverless):** no hardware to manage, pay-as-you-go, but proof-of-concept status.
- **Docker Model Runner (maximum privacy + zero cost):** run local LLMs via Docker Desktop for complete privacy and no API fees. Requires Apple Silicon, NVIDIA, or AMD GPU.
- **Laptop (okay for learning/dev):** simplest to start, but sleeps often and you may be tempted to expose it.

See runbooks:
- [Mac mini](./03-deploy/standalone-mac-mini.md)
- [VPS](./03-deploy/isolated-vps.md)
- [Cloudflare Moltworker](./03-deploy/cloudflare-moltworker.md)
- [Docker Model Runner](./03-deploy/docker-model-runner.md)

#### Q: Is OpenClaw "an AI model" like ChatGPT?
No. OpenClaw is a **self-hosted assistant platform** that *talks to* models (Anthropic/OpenAI/etc.) and *wraps them* with routing, sessions, tools, and chat integrations.

#### Q: What runs on my machine?
The main always-on process is the **Gateway** (default port **18789**) which multiplexes:
- a WebSocket control plane
- the dashboard/control UI (HTTP)
- optional HTTP endpoints (OpenAI-compatible APIs)

See: https://docs.openclaw.ai/gateway

#### Q: Where is my data stored?
By default, OpenClaw stores state under `~/.openclaw/` (or `~/.openclaw-<profile>/` for profiles). This includes config, credentials, and session transcripts.

See: https://docs.openclaw.ai/gateway/security ("Credential storage map")

#### Q: Does OpenClaw have telemetry?
This repo's positioning is local-first control. Still, your chosen **model provider** will receive whatever text/media is sent to it for inference, unless you run a local model.

#### Q: What’s the safest first setup?
- Run on a **single-user machine** you control (Mac mini).
- Keep the Gateway **loopback-only**.
- Use **pairing/allowlists** so only you can talk to it.
- Don’t enable powerful tools until you understand the blast radius.

Use the wizard:

```bash
openclaw onboard --install-daemon
```

#### Q: I opened the dashboard and it says “unauthorized” or keeps reconnecting.
The Gateway likely has auth enabled and the UI is missing the token/password.

Fast fixes:
- Run `openclaw dashboard` (it prints a tokenized URL).
- If remote: bring up an SSH tunnel first:
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
  ```
  then open `http://127.0.0.1:18789/?token=...`.

See: https://docs.openclaw.ai/help/faq (Control UI unauthorized)

#### Q: What does “pairing” mean?
Pairing is owner approval for:
- **DM pairing** (who can message the bot)
- **device/node pairing** (which devices can connect)

See: https://docs.openclaw.ai/start/pairing

---

### Intermediate FAQ

#### Q: What's the difference between `openclaw gateway` and `openclaw gateway restart`?
- `openclaw gateway` runs the Gateway in the **foreground** in your terminal.
- `openclaw gateway restart` restarts the **background service** (launchd/systemd).

See: https://docs.openclaw.ai/help/faq

#### Q: What port does OpenClaw use?
`gateway.port` controls the single multiplexed port for WebSocket + HTTP. Precedence is:

```text
--port > OPENCLAW_GATEWAY_PORT > gateway.port > default 18789
```

See: https://docs.openclaw.ai/help/faq

#### Q: I want remote access. Should I set `gateway.bind: "lan"`?
Usually no.

Preferred patterns:
- **Loopback + SSH tunnel** (universal)
- **Loopback + Tailscale Serve** (best UX)

Only bind to LAN/tailnet when you understand the auth requirements.

See: https://docs.openclaw.ai/gateway/remote and https://docs.openclaw.ai/gateway/tailscale

#### Q: Can I run multiple Gateways on one host?
Yes, but it’s usually unnecessary; one Gateway can run multiple channels and agents.

If you do, you must isolate:
- config path (`OPENCLAW_CONFIG_PATH`)
- state dir (`OPENCLAW_STATE_DIR`)
- workspace (`agents.defaults.workspace`)
- port (`gateway.port`)

See: https://docs.openclaw.ai/gateway/multiple-gateways

#### Q: How do I see what OpenClaw is doing?
Use:

```bash
openclaw status --all
openclaw logs --follow
```

See: https://docs.openclaw.ai/help/faq (log locations)

---

### Advanced FAQ

#### Q: What’s the real security risk: “public bot”, prompt injection, or host compromise?
All three matter, but the practical order is:
1) **Inbound access** (DM/group policies)
2) **Tool blast radius** (exec/browser/web)
3) **Network exposure** (bind modes, proxies, auth)
4) **Host compromise** (OS hardening, keys, patching)

See: https://docs.openclaw.ai/gateway/security

#### Q: How do plugins/extensions affect my threat model?
Plugins run **in-process** with the Gateway. Treat them like installing arbitrary code.

Recommendation:
- only install plugins you trust
- prefer pinned versions
- keep an explicit allowlist if supported

See: https://docs.openclaw.ai/gateway/security ("Plugins/extensions")

#### Q: If I want “maximum privacy”, do I need a local model?
A local model is the strongest privacy posture because it avoids sending content to a third-party provider. However, it changes the safety profile: smaller/weak local models can be easier to prompt-inject and may handle tool policies worse.

See: https://docs.openclaw.ai/gateway/local-models

#### Q: How do I make sure different people’s DMs don’t leak context to each other?
Consider DM session isolation (multi-user mode) so each peer gets an isolated DM session, and use identity linking only where appropriate.

For multi-agent setups, each agent can also be scoped independently: per-agent sandbox isolation, tool allow/deny policies, and workspace access controls prevent one agent's context from leaking into another. See [per-agent access scoping](./04-privacy-safety/threat-model.md#per-agent-access-scoping-multi-agent-setups) for details.

See: https://docs.openclaw.ai/gateway/security ("DM session isolation") and https://docs.openclaw.ai/concepts/session

---

> **See:** [`openclaw security audit` command reference](./08-security-analysis/security-audit-command-reference.md)

---

> **See:** [Official Security Advisories (CVEs/GHSAs)](./08-security-analysis/official-security-advisories.md)

---

> **See:** [Security audit analysis (Issue #1796)](./08-security-analysis/issue-1796-argus-audit.md)

---

> **See:** [Second security audit (Medium article)](./08-security-analysis/medium-article-audit.md)

---

> **See:** [Post-Merge Security Hardening](./08-security-analysis/post-merge-hardening.md)

---

> **See:** [Open Upstream Security Issues](./08-security-analysis/open-upstream-issues.md)

---

> **See:** [Open Upstream Security PRs](./08-security-analysis/open-upstream-prs.md)

---

> **See:** [Ecosystem Security Threats](./08-security-analysis/ecosystem-security-threats.md)

---

> **See:** [Cisco AI Defense Skill Scanner Analysis](./08-security-analysis/cisco-ai-defense-skill-scanner.md)

---

## Worst-Case Security Scenarios

> **Purpose:** This section documents what can go wrong in the worst possible misconfiguration or compromise scenarios for each deployment type.
>
> **Read this if:** You're evaluating OpenClaw for sensitive use cases, want to understand the blast radius of potential failures, or need to build a threat model for your organization.

See the detailed breakdown in [05-worst-case-security/](./05-worst-case-security/).

### Quick Reference: Deployment Risk Profiles

| Deployment | Trust Boundary | Biggest Risk | Recovery Complexity |
|------------|----------------|--------------|---------------------|
| **Mac Mini** | Your hardware | Physical access, cloud sync | Medium (rotate keys) |
| **VPS/1-Click** | Shared infra | Internet exposure, root compromise | High (rebuild VPS) |
| **Moltworker** | Cloudflare | No egress filtering, R2 breach | Very High (no local control) |

### Key Findings from Code Analysis

Based on source code review of:
- `src/gateway/net.ts` - Network binding with fallback chains
- `src/gateway/auth.ts` - Authentication mechanisms
- `src/agents/bash-tools.exec.ts` - Shell execution
- `src/pairing/pairing-store.ts` - Credential storage
- `src/security/audit.ts` - Security audit checks

**Critical vulnerabilities if misconfigured:**

1. **Silent binding fallback** - Loopback failure → 0.0.0.0 exposure (`src/gateway/net.ts:159-164`)
2. **Dangerous auth flags** - `dangerouslyDisableDeviceAuth` bypasses device verification (`src/config/types.gateway.ts:69-72`)
3. **No encryption at rest** - Credentials protected only by file permissions (0o600/0o700)
4. **Egress-free Moltworker** - Sandbox can exfiltrate to any server

### Scenario Documentation

| Document | Coverage |
|----------|----------|
| [Overview](./05-worst-case-security/README.md) | Attack surface comparison, decision guide, severity levels |
| [Mac Mini Risks](./05-worst-case-security/mac-mini-risks.md) | Physical access, cloud sync trap, silent network exposure |
| [VPS Risks](./05-worst-case-security/vps-risks.md) | Internet exposure, multi-tenant risks, credential storage |
| [Moltworker Risks](./05-worst-case-security/moltworker-risks.md) | Trust boundaries, egress filtering, R2 single point of failure |
| [Cross-Cutting](./05-worst-case-security/cross-cutting.md) | Prompt injection, tool execution, channel tokens, supply chain |
| [ClawHub Marketplace Risks](./05-worst-case-security/clawhub-marketplace-risks.md) | Skills marketplace supply chain, ClawHavoc campaign, social engineering |
| [Prompt Injection Attacks](./05-worst-case-security/prompt-injection-attacks.md) | 27 attack examples with data exfiltration scenarios |
| [Misconfiguration Examples](./05-worst-case-security/misconfiguration-examples.md) | 10 real mistakes with step-by-step fixes |
| [Incident Response](./05-worst-case-security/incident-response.md) | Containment, credential rotation, recovery procedures |

📚 **Key resource:** The [Prompt Injection Attacks](./05-worst-case-security/prompt-injection-attacks.md) guide (27 examples with defenses) is referenced throughout this documentation. If you read one security document beyond the threat model, read that one.

---

## Detecting OpenClaw Requests (for hosting services)

> **Purpose:** Documents how third-party services can identify HTTP requests originating from OpenClaw, and what OpenClaw users should know about their request fingerprint.
>
> **Read this if:** You run a hosting service/API and want to identify OpenClaw traffic, or you're an OpenClaw user who wants to understand what your instance reveals about itself.

### Quick Reference: Identifiable Headers

| Request type | Header | Value | Detectable? |
|---|---|---|---|
| Media file fetches | `User-Agent` | `OpenClaw-Gateway/1.0` | Yes — explicitly names OpenClaw |
| GitHub API (signal-cli install) | `User-Agent` | `openclaw` | Yes — explicitly names OpenClaw |
| Anthropic OAuth API | `User-Agent` | `openclaw` | Yes — explicitly names OpenClaw |
| Perplexity/OpenRouter API | `HTTP-Referer` | `https://openclaw.ai` | Yes — domain identifies OpenClaw |
| Perplexity/OpenRouter API | `X-Title` | `OpenClaw` / `OpenClaw Web Search` | Yes — explicitly names OpenClaw |
| MiniMax VLM API | `MM-API-Source` | `OpenClaw` | Yes — custom header |
| ACP protocol | `clientInfo.name` | `openclaw-acp-client` | Yes — protocol-level identification |
| WebFetch (browsing websites) | `User-Agent` | Chrome browser string | No — indistinguishable from real browser |
| Brave Search API | *(no custom UA)* | Default fetch UA | Weak — Node.js fetch fingerprint only |
| xAI Grok API | *(no custom UA)* | Default fetch UA | Weak — Node.js fetch fingerprint only |

The full analysis includes source code references, Cloudflare WAF rules (with regex examples for Business/Enterprise), and a guide for placing Cloudflare as a reverse proxy in front of your Gateway with inbound header protection: [Detecting OpenClaw Requests](./04-privacy-safety/detecting-openclaw-requests.md)

---

> **See:** [AI Model Analysis Comparison](./08-security-analysis/ai-model-analysis-comparison.md)

---

## Official docs (high-signal links)

- Getting started: https://docs.openclaw.ai/start/getting-started
- Install: https://docs.openclaw.ai/install
- Gateway (runbook): https://docs.openclaw.ai/gateway
- Gateway security: https://docs.openclaw.ai/gateway/security
- Remote access: https://docs.openclaw.ai/gateway/remote
- Tailscale: https://docs.openclaw.ai/gateway/tailscale
- Pairing: https://docs.openclaw.ai/start/pairing
- Help / FAQ: https://docs.openclaw.ai/help/faq
- Troubleshooting: https://docs.openclaw.ai/gateway/troubleshooting
- External security guide: https://vibeproof.dev/blog/moltbot-security-setup-guide
