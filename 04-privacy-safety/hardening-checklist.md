# Hardening checklist (high privacy)

## Table of contents (Explain OpenClaw)

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
- Deployment
  - [Standalone Mac mini](../03-deploy/standalone-mac-mini.md)
  - [Isolated VPS](../03-deploy/isolated-vps.md)
  - [Cloudflare Moltworker](../03-deploy/cloudflare-moltworker.md)
- Optimizations
  - [Cost + token optimization](../06-optimizations/cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

This is an actionable checklist to get to a strong “high privacy / low exposure” posture.

Source of truth:
- https://docs.openclaw.ai/gateway/security
- https://docs.openclaw.ai/gateway/remote
- https://docs.openclaw.ai/gateway/tailscale
- https://docs.openclaw.ai/start/pairing

---

## 0) Decide your trust boundary (recommended)

- Treat the **Gateway host** as the trust boundary.
- If you can’t harden the host, don’t enable high-power tools.

---

## 1) Lock down who can trigger the bot (DMs)

### Prefer DM policy: pairing (or allowlist)
- `pairing`: unknown senders get a code; bot ignores them until approved
- `allowlist`: unknown senders are blocked
- avoid `open`

Approve pairing requests:

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Docs: https://docs.openclaw.ai/start/pairing

---

## 2) Lock down group behavior

Common group-safe defaults:
- require mention
- restrict which groups the bot will respond in
- restrict who can trigger commands in groups

Docs:
- https://docs.openclaw.ai/concepts/groups
- https://docs.openclaw.ai/gateway/security

---

## 3) Keep the Gateway loopback-only unless you have a reason

Recommended default:
- `gateway.bind: "loopback"`

Remote access patterns:

### SSH tunnel (universal)
```bash
ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
```

### Tailscale Serve (best UX; tailnet-only)
Keep Gateway on loopback and expose the UI via HTTPS Serve.

Docs: https://docs.openclaw.ai/gateway/tailscale

---

## 4) Require auth for any non-loopback access

- Token/password auth is required when binding beyond loopback.
- The wizard generates a token by default.

If you suspect auth is misconfigured:
- use `openclaw dashboard` to get a URL that includes the token once
- run `openclaw security audit` to detect risky exposure

---

## 5) Run the security audit regularly

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

`--fix` tightens common footguns (group policy, redaction, file perms).

Docs: https://docs.openclaw.ai/gateway/security

---

## 6) Treat browser control like an admin API

If you enable browser control remotely:
- tailnet-only preferred
- token auth required
- avoid Funnel unless you *explicitly* want public exposure

Docs: https://docs.openclaw.ai/gateway/security and https://docs.openclaw.ai/gateway/tailscale

---

## 7) Minimize tool blast radius

Practical guidance:
- Start with tools disabled or minimal.
- Add one tool category at a time.
- Prefer sandboxing for non-main sessions.

Docs:
- https://docs.openclaw.ai/tools
- https://docs.openclaw.ai/gateway/sandboxing

> **Why this matters for prompt injection:** Limiting tools reduces the damage a successful injection can do. See [Prompt Injection Attacks](../05-worst-case-security/prompt-injection-attacks.md) for 27 examples of how attackers exploit tool access.

---

## 8) Protect secrets on disk

- Treat `~/.openclaw` as sensitive.
- Don’t sync it to iCloud/Dropbox/etc.
- Ensure permissions are tight (audit can fix).

Docs: https://docs.openclaw.ai/gateway/security

---

## 9) Be cautious with plugins/extensions

Plugins run **in-process**.

Recommendations:
- install only trusted plugins
- pin versions
- inspect code on disk

### ClawHub Skills Warning

In Feb 2026, **341 malicious skills** (12% of audited packages) were found on ClawHub. The attack used social engineering, not code exploits.

**Scanning improvements (Feb 2026):** ClawHub now scans all published skills via a [VirusTotal partnership](https://openclaw.ai/blog/virustotal-partnership) (automated analysis + daily rescans). OpenClaw also includes a built-in local skill scanner that runs at install time and detects dangerous code patterns. However, neither scanner can catch social engineering (the actual ClawHavoc attack vector) or prompt injection — manual review remains essential.

**Before installing any ClawHub skill:**
- [ ] Check VirusTotal scan status on the ClawHub skill page
- [ ] Review local scanner warnings shown during skill installation
- [ ] Check skill age (avoid < 30 days old)
- [ ] Verify publisher reputation
- [ ] Read the actual code, not just documentation
- [ ] **NEVER** run "prerequisite" terminal commands from skill docs
- [ ] Use [Koi Security Scanner](https://koi.ai/clawhub-scanner) as an independent third-party check
- [ ] Be especially suspicious of crypto-related skills

See: [ClawHub Marketplace Risks](../05-worst-case-security/clawhub-marketplace-risks.md)

Docs: https://docs.openclaw.ai/plugin and https://docs.openclaw.ai/gateway/security

---

## 10) Control mDNS/Bonjour discovery

The Gateway can broadcast its presence on the local network via mDNS. Check your current mode:

```bash
openclaw config get discovery.mdns
```

| Network environment | Recommended mode | Why |
|---------------------|------------------|-----|
| Home LAN (trusted) | `minimal` (default) | Convenient discovery; sensitive fields omitted |
| Shared/public network | `off` | Don't advertise the gateway at all |
| Debugging on trusted network | `full` | Exposes cliPath + sshPort for diagnostics |

To disable mDNS entirely:
```bash
openclaw config set discovery.mdns off
```

Or via environment variable:
```bash
export OPENCLAW_DISABLE_BONJOUR=1
```

Source: `src/gateway/server-discovery-runtime.ts:19,23-30`, `src/infra/bonjour.ts:28-38`

See: [Threat model - mDNS/Bonjour](./threat-model.md#6-mdnsbonjour-network-discovery)

---

## 11) Configure trusted proxies

If you run a reverse proxy (nginx, Caddy, Traefik) in front of the Gateway, you **must** configure trusted proxies so the Gateway can correctly identify client IPs and enforce local-access checks.

```bash
openclaw config set gateway.trustedProxies '["127.0.0.1"]'
```

Verify your configuration:
```bash
openclaw security audit
# Look for: gateway.trusted_proxies_missing
```

If you're not using a reverse proxy, leave `trustedProxies` empty (the default).

Source: `src/gateway/net.ts:98-120`

See: [Threat model - Trusted proxies](./threat-model.md#trusted-proxies-reverse-proxy-configuration)

---

## 12) Audit workspace .md files for hidden content

OpenClaw loads nine workspace bootstrap `.md` files directly into the agent's system prompt on every turn. These appear as trusted context — not wrapped with untrusted content markers. The built-in skill scanner does not scan `.md` files, so malicious content in these files is invisible to automated checks.

**Scan for hidden HTML comments** (the most common injection vector in `.md` files):

```bash
grep -rn "<!--" ~/your-workspace-dir/
```

**Scan for suspicious instruction patterns:**

```bash
grep -rniE "(ignore previous|system override|you are now|execute the following|curl.*base64|wget.*credentials)" ~/your-workspace-dir/*.md
```

**Run Cisco AI Defense scanner** for deeper LLM-based analysis:

```bash
skill-scanner scan ~/your-workspace-dir/
```

Files to audit: `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`, `MEMORY.md`, `memory.md`, and any files in the `memory/` directory.

See: [Cisco AI Defense gap analysis](../08-security-analysis/cisco-ai-defense-skill-scanner.md#beyond-skillmd-all-persistent-md-files-are-unscanned), [Threat model #7](./threat-model.md#7-persistent-memory-files), [Attack #27](../05-worst-case-security/prompt-injection-attacks.md#-attack-27-persistent-memory-injection)
