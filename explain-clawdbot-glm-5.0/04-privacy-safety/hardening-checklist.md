# Hardening checklist (high privacy)

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

This is an actionable checklist to get to a strong "high privacy / low exposure" posture with GLM-5.0.

Source of truth:
- https://docs.openclaw.ai/gateway/security
- https://docs.openclaw.ai/gateway/remote
- https://docs.openclaw.ai/gateway/tailscale

---

## 0) Decide your trust boundary (recommended)

- Treat the **Gateway host** as your trust boundary.
- If you can't harden the host, don't enable high-power tools.

---

## 1) Lock down who can trigger bot (DMs)

### Prefer DM policy: pairing (or allowlist)

- `pairing`: unknown senders get a code; bot ignores them until approved
- `allowlist`: unknown senders are blocked
- Avoid `open`

Approve pairing requests:

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Docs: https://docs.openclaw.ai/start/pairing

---

## 2) Lock down group behavior

Common group-safe defaults:
- Require mention
- Restrict which groups bot will respond in
- Restrict who can trigger commands in groups

**GLM-5.0 consideration:** GLM-5.0's larger context window may include more group history, increasing the risk of sensitive information leakage through prompt injection.

Docs:
- https://docs.openclaw.ai/concepts/groups
- https://docs.openclaw.ai/gateway/security

---

## 3) Keep Gateway loopback-only unless you have a reason

Recommended default:
- `gateway.bind: "loopback"`

Remote access patterns:

### SSH tunnel (universal)

```bash
ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
```

### Tailscale Serve (best UX)

Keep Gateway on loopback and expose the UI via HTTPS Serve to your tailnet.

Docs: https://docs.openclaw.ai/gateway/tailscale

---

## 4) Require auth for any non-loopback access

- Token/password auth is required when binding beyond loopback.
- Generate a secure token: `openclaw config set gateway.auth.token "$(openssl rand -hex 32)"`

If using Tailscale:
- Set `gateway.auth.allowTailscale: true` to accept Tailscale identity headers

---

## 5) Run security audit regularly

```bash
# Read-only scan of config + filesystem permissions
openclaw security audit

# Everything above + live WebSocket probe
openclaw security audit --deep

# Apply safe auto-fixes
openclaw security audit --fix
```

`--fix` tightens common issues and sets safe defaults.

---

## 6) Minimize tool blast radius

~~GLM-5.0's function calling capabilities make tool policies especially important.~~

> **Opus 4.6 audit:** OpenClaw has no separate "function calling" mechanism. All models (including GLM-5.0) use the same tool invocation pipeline. The relevant controls are `tools.profile` and `tools.allow`/`tools.deny`. Source: `src/agents/pi-tools.ts`, `src/agents/tool-policy.ts`.

Tool policies are especially important for any model with tool access.

Practical guidance:
- Start with tools disabled or minimal
- Add one tool category at a time
- Use tool allow/deny policies per agent
- Prefer sandboxing for non-main sessions

Docs:
- https://docs.openclaw.ai/tools
- https://docs.openclaw.ai/gateway/sandboxing

### ~~Function calling controls (GLM-5.0 specific)~~

~~GLM-5.0 supports structured function calling. To control this:~~

~~`openclaw config set agents.defaults.functionCalling true`~~

~~`openclaw config set agents.defaults.tools.allowlist '["web_search", "memory_search"]'`~~

> **Opus 4.6 audit:** Both config options are fabricated. `agents.defaults.functionCalling` does not exist in OpenClaw's config schema (`src/config/types.agent-defaults.ts`). The correct tool allowlist option is `agents.defaults.tools.allow` (not `allowlist`). OpenClaw has no separate "function calling" toggle — all models use the same tool invocation pipeline controlled by `tools.profile` and `tools.allow`/`tools.deny`.
>
> Corrected commands:
> ```bash
> # Set tool profile (minimal, coding, messaging, or null for all)
> openclaw config set agents.defaults.tools.profile minimal
>
> # Set tool allow list
> openclaw config set agents.defaults.tools.allow '["web_search", "memory_search"]'
> ```

---

## 7) Protect secrets on disk

- Treat `~/.openclaw` as sensitive
- Don't sync it to iCloud/Dropbox/etc.
- Ensure permissions are tight (audit can fix)

### Zhipu AI API key protection

```bash
# Store in environment variable (not in shell history)
export ZHIPU_API_KEY="your-key-here"

# Or use a secure file manager
pass insert zhipu-api-key <key>
```

---

## 8) Control mDNS/Bonjour discovery

Check your current mode:

```bash
openclaw config get discovery.mdns
```

| Network environment | Recommended mode | Why |
|---------------------|-----------|-----|
| Home LAN (trusted) | `minimal` (default) | Convenient discovery; sensitive fields omitted |
| Shared/public network | `off` | Don't advertise gateway at all |

Disable entirely:

```bash
openclaw config set discovery.mdns off
```

Or via environment variable:

```bash
export OPENCLAW_DISABLE_BONJOUR=1
```

---

## 9) Configure trusted proxies (if using reverse proxy)

If you run nginx, Caddy, or Traefik in front of the Gateway:

```bash
openclaw config set gateway.trustedProxies '["127.0.0.1"]'
```

Without this, IP-based rate limiting and local-access checks will see the proxy IP instead of the real client IP.

Verify:

```bash
openclaw security audit
# Look for: gateway.trusted_proxies_missing
```

---

## 10) Audit workspace .md files for hidden content

OpenClaw loads workspace bootstrap `.md` files directly into the agent's system prompt as trusted context. The built-in skill scanner does NOT scan these files.

**Files to audit:** `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`, `MEMORY.md`, `memory.md`

**Scan for hidden HTML comments:**

```bash
# Search for suspicious HTML comments
grep -rn "<!--" ~/your-workspace-dir/
```

**Scan for suspicious instruction patterns:**

```bash
grep -rniE "(ignore previous|system override|you are now|execute the following|curl.*base64|wget.*credentials)" ~/your-workspace-dir/*.md
```

---

## 11) Never let AI modify security-critical config

The gateway tool's `config.apply` and `config.patch` actions have **zero permission checks**.

**Primary defense - Remove gateway tool:**

```bash
# Option A: Use coding tool profile (excludes gateway tool)
openclaw config set tools.profile coding

# Option B: Deny gateway tool specifically
openclaw config set tools.deny '["gateway"]'
```

**Keep config commands disabled:**

```bash
# commands.config should be false by default — verify it stays that way
openclaw config get commands.config
# Should be: false (or unset)
```

---

## ~~12) Function calling audit trail~~

~~GLM-5.0's function calling feature creates a new audit surface. Enable logging of all tool invocations:~~

~~`openclaw config set logging.toolCalls true`~~

> **Opus 4.6 audit:** `logging.toolCalls` does not exist in OpenClaw's `LoggingConfig` (`src/config/types.base.ts`). There is no dedicated "function calling audit" toggle. Tool output visibility is controlled by `logging.redactSensitive` (valid values: `false`, `"tools"`, `"all"`). Setting `logging.redactSensitive: false` keeps full tool output in logs for auditing; `"tools"` redacts tool input/output.
>
> Corrected command:
> ```bash
> # Keep tool output visible in logs for auditing (less private, more auditable)
> openclaw config set logging.redactSensitive false
> ```

Regularly review:

```bash
# View tool call logs
openclaw logs | grep -i "tool_call"
```

Look for:
- Unexpected tool invocations
- Tool parameters that seem malicious
- Patterns suggesting prompt injection

---

## 13) Be cautious with plugins/extensions

Plugins run **in-process** with Gateway.

Recommendations:
- Only install trusted plugins
- Pin versions
- Inspect code on disk

---

## 14) Secure Zhipu AI API key rotation

If you suspect your Zhipu AI API key has been compromised:

1. Generate a new API key from Zhipu AI console
2. Update the key in your configuration
3. Revoke the old key from Zhipu AI console

```bash
# Update key via CLI
openclaw config set agents.defaults.authProfile.apiKey "your-new-key-here"
```

---

## 15) GLM-5.0 specific hardening

### Context window management

GLM-5.0 supports ~200K token context. Larger context means:
- More data in each request (if compromised)
- Higher memory usage
- More session storage on disk

**Recommendation:** For sensitive deployments, consider using `glm-5.0-flash` with ~128K context.

### Model isolation

For high-security deployments, consider using separate agents for different trust levels:

```json
{
  "agents": {
    "lists": {
      "public-bot": {
        "provider": "zhipu",
        "model": "glm-5.0-flash",
        "tools": { "profile": "minimal" }
      },
      "private-assistant": {
        "provider": "zhipu",
        "model": "glm-5.0",
        "tools": { "profile": "coding" },
        "systemPrompt": "You are a private assistant with elevated tool access. Be cautious with tool requests."
      }
    }
  }
}
```

### Temperature controls

GLM-5.0 works well with specific temperature ranges:

```bash
# For analytical work (lower temperature for more focused outputs)
openclaw config set agents.defaults.temperature 0.3

# For creative work
openclaw config set agents.defaults.temperature 0.8
```

---

See also: [High privacy config example](./high-privacy-config.example.json5.md) for a complete hardened configuration.
