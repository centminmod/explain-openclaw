# Hardening checklist (high privacy)

This checklist is derived from `docs/gateway/security.md` and related gateway docs.

## 1) Lock down who can talk to the bot

- Prefer `dmPolicy: "pairing"` (default) or `dmPolicy: "allowlist"`.
- Avoid `dmPolicy: "open"` unless you truly want public access.
- Use pairing approvals:

```bash
clawdbot pairing list telegram
clawdbot pairing approve telegram <CODE>
```

Docs: `docs/start/pairing.md`.

## 2) Keep the Gateway loopback-only by default

- Keep `gateway.bind: "loopback"` so it only listens on `127.0.0.1`.
- For remote access, prefer **SSH tunnels** or **Tailscale Serve** (tailnet-only) instead of LAN/public binds.

Docs: `docs/gateway/remote.md` and `docs/gateway/tailscale.md`.

## 3) Run the built-in security audit

```bash
clawdbot security audit
clawdbot security audit --deep
clawdbot security audit --fix
```

`--fix` tightens risky defaults (groupPolicy, redaction, filesystem perms).
Source: `docs/gateway/security.md`.

## 4) Protect secrets on disk

- Treat `~/.clawdbot` as sensitive.
- Ensure permissions are locked down (audit can fix this).
- Never put this folder in cloud sync.

Credential map is documented in `docs/gateway/security.md`.

## 5) Limit tool blast radius

- Disable or restrict high-risk tools (exec, browser, web fetch/search) until you need them.
- Prefer sandboxing for non-main sessions (docs mention `agents.defaults.sandbox.mode: "non-main"`).

Source: `docs/start/getting-started.md` (sandboxing note) and `docs/gateway/security.md`.

## 6) Treat browser control like an admin API

If you expose browser control remotely:
- tailnet-only (Serve)
- token auth
- avoid public funnel unless explicitly intended

Docs: `docs/gateway/tailscale.md` (browser control server section).
