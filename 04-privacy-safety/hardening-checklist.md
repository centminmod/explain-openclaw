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
clawdbot pairing list telegram
clawdbot pairing approve telegram <CODE>
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
- use `clawdbot dashboard` to get a URL that includes the token once
- run `clawdbot security audit` to detect risky exposure

---

## 5) Run the security audit regularly

```bash
clawdbot security audit
clawdbot security audit --deep
clawdbot security audit --fix
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

---

## 8) Protect secrets on disk

- Treat `~/.clawdbot` as sensitive.
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

Docs: https://docs.openclaw.ai/plugin and https://docs.openclaw.ai/gateway/security
