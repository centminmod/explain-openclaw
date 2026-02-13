# Threat model (beginner-friendly)

## Table of contents (Explain OpenClaw)

- [Home (README)](../README.md)
- Plain English
  - [What is OpenClaw?](../01-plain-english/what-is-clawdbot.md)
  - [Glossary](../01-plain-english/glossary.md)
  - [CLI commands](../01-plain-english/cli-commands.md)
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

📚 **For 27 real attack examples with defenses, see: [Prompt Injection Attacks](../05-worst-case-security/prompt-injection-attacks.md)**

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

#### Trusted proxies (reverse proxy configuration)

If you run a reverse proxy (e.g. nginx, Caddy, Traefik) in front of the Gateway, two things can go wrong:

1. **Spoofed local access:** Without trusted proxy config, an attacker can send a fake `X-Forwarded-For: 127.0.0.1` header to trick the Gateway into treating the request as local (bypassing auth).
2. **IP checks see the proxy IP:** The Gateway sees the proxy's IP instead of the real client IP, breaking rate limiting and access controls.

The Gateway solves this with a trust chain:
- `isTrustedProxyAddress()` checks if the connecting IP is in your trusted list (`src/gateway/net.ts:98-104`)
- `resolveGatewayClientIp()` only reads `X-Forwarded-For`/`X-Real-IP` headers when the immediate connection comes from a trusted proxy (`src/gateway/net.ts:106-120`)
- `isLocalDirectRequest()` uses both checks to determine if a request is genuinely local (`src/gateway/auth.ts:96-117`)

**Configuration:**
```bash
openclaw config set gateway.trustedProxies '["127.0.0.1"]'
```

**Rules of thumb:**
- Running a reverse proxy? Set `trustedProxies` to the proxy's IP address(es)
- No reverse proxy? Leave `trustedProxies` empty (the default)
- `openclaw security audit` flags this as `gateway.trusted_proxies_missing` when a proxy is detected but not configured

Source: `src/gateway/auth.ts:73-94` (`resolveTailscaleClientIp()`, `resolveRequestClientIp()`), `src/security/audit.ts` (audit check)

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

#### ClawHub Marketplace

ClawHub is a third-party skills marketplace for OpenClaw. In Feb 2026, **341 malicious skills** were discovered (12% of audited packages). The "ClawHavoc" campaign used social engineering (fake prerequisite commands) to install Atomic Stealer malware.

**Key risk:** Skills run in-process with the Gateway. A malicious skill has full access to credentials, sessions, and network.

**Security improvements (Feb 2026):** ClawHub now scans all published skills through a [VirusTotal partnership](https://openclaw.ai/blog/virustotal-partnership) (6-step pipeline with 70+ AV engines + Gemini LLM code review, daily rescans). OpenClaw also includes a built-in local skill scanner (`src/security/skill-scanner.ts`) that runs pattern-based static analysis at install time, detecting dangerous patterns like shell execution, eval, crypto mining, and credential harvesting.

**Caveat:** Neither scanner can detect social engineering (the ClawHavoc attack vector used fake "prerequisite" commands) or prompt injection payloads. A clean scan is not a guarantee of safety.

> **See also:** [Prompt Injection Attacks](../05-worst-case-security/prompt-injection-attacks.md) -- 27 examples of how injection payloads work in practice.

**Additional mitigations for ClawHub:**
- Check VirusTotal scan status on ClawHub skill page before installing
- Review local scanner warnings shown during skill installation
- Avoid skills less than 30 days old
- Never run "prerequisite" terminal commands from skill docs
- Use [Koi Security Scanner](https://koi.ai/clawhub-scanner) as an independent third-party check
- Inspect skill code in `~/.openclaw/skills/` before enabling

See: [ClawHub Marketplace Risks](../05-worst-case-security/clawhub-marketplace-risks.md)

### 6) mDNS/Bonjour network discovery

The Gateway can advertise itself on the local network via mDNS (Bonjour), using the `_openclaw-gw._tcp` service type. This makes it easy for OpenClaw clients (e.g. the Mac app, mobile apps) to discover the Gateway automatically -- but it also means anyone on the same LAN can see the broadcast.

**What gets broadcast (TXT record fields):**

| Field | Always included | Full-mode only |
|-------|:-:|:-:|
| `gatewayPort` | Yes | Yes |
| `transport` ("gateway") | Yes | Yes |
| `canvasPort` | Yes (if set) | Yes (if set) |
| `tailnetDns` | Yes (if set) | Yes (if set) |
| `gatewayTlsSha256` | Yes (if TLS) | Yes (if TLS) |
| `cliPath` | -- | Yes |
| `sshPort` | -- | Yes |

Source: `src/infra/bonjour.ts:12-26` (opts type), `src/infra/bonjour.ts:130-146` (minimal conditionals), `src/gateway/server-discovery-runtime.ts:19,23-30` (mdnsMode logic)

**Risk:** In "full" mode, the broadcast includes `cliPath` (filesystem structure) and `sshPort` (attack vector). Even in "minimal" mode, the Gateway port and hostname are visible to anyone on the LAN segment.

**Mitigations:**
- Default mode is `minimal` (omits cliPath and sshPort)
- Disable entirely: `openclaw config set discovery.mdns off`
- Environment variable override: `OPENCLAW_DISABLE_BONJOUR=1`
- mDNS is link-local only (not routable beyond the LAN/subnet)
- Use "full" mode only on trusted home networks for debugging

### 7) Persistent memory files

OpenClaw loads nine named workspace `.md` files (AGENTS.md, SOUL.md, TOOLS.md, IDENTITY.md, USER.md, HEARTBEAT.md, BOOTSTRAP.md, MEMORY.md, memory.md) on every agent turn via `loadWorkspaceBootstrapFiles()` (`src/agents/workspace.ts:265-319`). These are injected directly into the system prompt as trusted context — they do **not** carry `<<<EXTERNAL_UNTRUSTED_CONTENT>>>` markers like fetched web pages or webhook payloads do. Each file is truncated at 20,000 characters (`src/agents/pi-embedded-helpers/bootstrap.ts:84`).

Additionally, `memory/*.md` directory files are accessed via `memory_search`/`memory_get` tool calls (4,000-char budget) through a separate pipeline (`src/memory/internal.ts:78-107`).

**Risk:** Any process or user with workspace write access can plant persistent prompt injection that survives across sessions and appears as trusted system instructions. The built-in skill scanner (`src/security/skill-scanner.ts:37-46`) only scans JS/TS files — none of these `.md` files are scanned for malicious content.

Mitigations:
- OS-level file permissions (restrict write access to the workspace directory)
- Periodic content audit: `grep -rn "<!--" .` to detect hidden HTML comments
- Run Cisco AI Defense scanner against workspace directory for deeper LLM-based analysis
- Subagent exposure is limited: `filterBootstrapFilesForSession()` (`src/agents/workspace.ts:323-331`) restricts subagents to only AGENTS.md + TOOLS.md

Source: `src/agents/workspace.ts:23-31` (file list), `src/agents/pi-embedded-helpers/bootstrap.ts:84,162-191` (injection), `src/memory/qmd-manager.ts:346-352` (QMD validation)

See: [Cisco AI Defense gap analysis](../08-security-analysis/cisco-ai-defense-skill-scanner.md#beyond-skillmd-all-persistent-md-files-are-unscanned)

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

## Per-agent access scoping (multi-agent setups)

When running multiple agents on the same Gateway, each agent should have only the access it needs. OpenClaw provides three scoping layers:

### Sandbox isolation per agent

Each agent can have its own sandbox configuration controlling:
- **mode:** `off` (no sandbox), `agent` (per-agent container), or `all` (all sessions sandboxed)
- **scope:** `dedicated` (isolated) or `shared` (shares with other agents)
- **workspaceAccess:** `none`, `ro` (read-only), or `rw` (read-write)
- **Docker:** whether the sandbox runs in a Docker container

Agent-specific settings override global defaults. Resolution order: agent config -> global agent defaults -> built-in defaults.

Source: `src/agents/sandbox/config.ts:126` (`resolveSandboxConfigForAgent()`)

### Tool policies per agent

Each agent can have its own allow/deny tool list. The precedence chain is:
1. Agent-specific allow/deny (`agents.list[].tools.sandbox.tools`)
2. Global agent defaults (`tools.sandbox.tools`)
3. Built-in defaults (all tools allowed)

If agent-specific lists are set, they take priority over global lists.

Source: `src/agents/sandbox/tool-policy.ts:71` (`resolveSandboxToolPolicyForAgent()`)

### Named tool profiles

For convenience, OpenClaw provides named tool profiles that map to common use cases:

| Profile | Description | What it allows |
|---------|-------------|---------------|
| `minimal` | Read-only status | `session_status` only |
| `coding` | Development tasks | Filesystem, runtime, sessions, memory, images |
| `messaging` | Communication tasks | Messaging group, session management |
| `full` | Unrestricted | All tools (empty allow list = no restrictions) |

Source: `src/agents/tool-policy.ts:63-80` (`TOOL_PROFILES`)

### DM session isolation

Each agent's DM sessions are keyed by `<agentId>:<sessionKey>`, so conversations with Agent A cannot leak into Agent B's context -- even when both agents serve the same messaging channel.

### Security recommendation for multi-agent setups

- Apply **least privilege**: each agent gets only the tools and workspace access it needs
- Use `sandbox.mode: "all"` for agents that process untrusted input (e.g. public-facing bots)
- Set `workspaceAccess: "none"` or `"ro"` for agents that don't need to write files
- Use the `minimal` or `messaging` tool profiles for agents that only need communication capabilities

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
