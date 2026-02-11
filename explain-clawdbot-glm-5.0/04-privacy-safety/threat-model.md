# Threat model (beginner-friendly)

## Table of contents (Explain OpenClaw - GLM-5.0 Edition)

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
  - [High privacy config example](./high-privacy-config.example.json5.md)
- Installation
  - [Install and onboarding](../03-install/install-and-onboard.md)
  - [Build from source](../03-install/from-source.md)
- Deployment
  - [Standalone Mac mini](../05-use-cases/mac-mini-standalone.md)
  - [Isolated VPS](../05-use-cases/vps-isolated.md)

---

This page explains *why* privacy and safety configuration matters for OpenClaw with GLM-5.0.

If you only take one idea from this:

> Treat the **Gateway host** (and anything it can reach) as your security boundary.

---

## Why OpenClaw is "different-risk" than a normal chat bot

OpenClaw can be configured to do much more than respond with text. When using GLM-5.0's enhanced capabilities, this becomes even more powerful.

Depending on what you enable, it can:
- send outbound messages on real messaging accounts
- fetch or browse untrusted websites
- run scheduled jobs (automation)
- invoke tools that touch files/network
- invoke node/device actions (camera/audio/canvas)

That means an attacker doesn't need to "hack" the codebase. Often they just need to:

1) get a message (or web page) into the agent's context, and
2) convince the model to take an unsafe action

This is the practical form of **prompt injection**.

📚 **For detailed attack examples, see:** [Prompt Injection Attacks](../05-worst-case-security/prompt-injection-attacks.md)

Official security doc (source of truth): https://docs.openclaw.ai/gateway/security

---

## Threat surfaces

### 1) Inbound messages (DMs + groups)

**Risk:** Strangers sending prompts that convince GLM-5.0 to take unsafe actions.

Mitigations:
- DM pairing / allowlists
- group allowlists + require-mention defaults
- command gating/access groups

**GLM-5.0 consideration:** GLM-5.0's improved reasoning makes it potentially more susceptible to sophisticated prompt injection attacks. Be especially careful with public-facing deployments.

### 2) Untrusted content bot reads

Even if only you can message the bot, it may read untrusted content:
- web pages (fetch/search/browser)
- attachments (PDFs, docs)
- pasted logs
- forwarded messages

**GLM-5.0 consideration:** Better code analysis capabilities can help identify malicious patterns in untrusted content, but may also enable more sophisticated attacks if tools are enabled.

Mitigations:
- Treat untrusted content like email attachments
- Use a "reader" agent with tools disabled
- Avoid combining "reads random web pages" with "can run exec"

### 3) Network exposure

If the Gateway is reachable from more networks than intended, risk goes up fast.

Mitigations:
- Keep `gateway.bind: "loopback"` where possible
- Use SSH tunnel / Tailscale Serve
- Require auth (token/password) for non-loopback binds
- Be careful with reverse proxies; configure trusted proxies

### 4) Local disk + secrets

OpenClaw stores transcripts and credentials on disk under `~/.openclaw/`.

If another user/process on the host can read that directory, privacy is gone.

Mitigations:
- File permissions (audit fixes these)
- Avoid syncing `~/.openclaw` to cloud drives
- OS-level hardening (separate user, disk encryption)

**Zhipu AI API keys:** Store these in env vars or encrypted storage where possible.

### 5) Supply chain / plugins

Plugins run **in-process** with Gateway.

Mitigations:
- Only install trusted plugins
- Pin versions
- Inspect code on disk
- Keep an allowlist (if supported)

### 6) Persistent memory files

OpenClaw loads workspace `.md` files (AGENTS.md, TOOLS.md, etc.) on every agent turn. These are injected directly into the system prompt as trusted context — they do **not** carry untrusted content markers.

**Risk:** Any process or user with workspace write access can plant persistent prompt injection that survives across sessions.

Mitigations:
- OS-level file permissions (restrict write access)
- Periodic content audit
- Run security scanners against workspace directory

See: [Hardening checklist - Audit workspace .md files](../04-privacy-safety/hardening-checklist.md#12-audit-workspace-md-files-for-hidden-content)

### 7) Function calling abuse (GLM-5.0 specific)

GLM-5.0 has native function calling support. While this is a powerful feature, it also creates new attack vectors:

**Attack pattern:**
1. Attacker gets GLM-5.0 to call a tool with malicious parameters
2. Tool returns results to the model
3. Attacker uses results in follow-up prompt to exfiltrate data

**Defenses:**
- Tool parameter validation and sanitization
- Audit logging of all tool invocations
- Restrict which tools are available to each agent
- Use tool allow/deny policies

Source: `src/agents/pi-tools.ts`, `src/agents/tool-policy.ts`

### 8) mDNS/Bonjour network discovery

The Gateway can advertise itself on local network via mDNS (Bonjour), using `_openclaw-gw._tcp` service type.

**Risk:** Anyone on the same LAN segment can see that OpenClaw is running and learn your hostname.

**GLM-5.0 consideration:** Larger context windows and more sessions increase potential information leakage through mDNS broadcasts.

Mitigations:
- Default mode is `minimal` (omits sensitive fields)
- Disable entirely: `openclaw config set discovery.mdns off`
- Environment variable: `export OPENCLAW_DISABLE_BONJOUR=1`

Source: `src/infra/bonjour.ts:12-26`, `src/gateway/server-discovery-runtime.ts:19,23-30`

---

## A practical attacker model

Ask: "who could realistically cause trouble?"

- Random internet users (if you open DMs/groups)
- People in your group chats
- A compromised plugin/package
- A compromised model provider account / API key
- A compromised host (VPS intrusion, stolen laptop)

---

## Core principle: access control before intelligence

OpenClaw's docs emphasize an ordering that works in practice:

1) **Identity first** — who is allowed to trigger bot (pairing/allowlists)
2) **Scope next** — what the bot is allowed to do (tool policy/sandboxing/nodes)
3) **Model last** — assume GLM-5.0 can be manipulated; design so manipulation has limited blast radius

---

## What "high privacy" means in OpenClaw terms

High privacy usually implies:
- Gateway host is locked down
- Gateway binds to localhost (or tailnet-only)
- Control UI is not public
- Only you (or a small allowlist) can trigger bot
- Tool surface area is minimal
- Sensitive logs are redacted
- Credentials are stored in env vars or encrypted stores where possible

**For GLM-5.0 specifically:**
- Zhipu AI API keys stored securely
- Model output (which may contain sensitive data) is redacted from logs
- Function calling is audited and logged

Next: [Hardening checklist](./hardening-checklist.md)
