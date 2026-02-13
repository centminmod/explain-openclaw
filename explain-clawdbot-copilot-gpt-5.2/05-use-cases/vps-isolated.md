# Use case: Isolated VPS gateway (remote, locked down)

Goal: run the Gateway on a small Linux VPS (always-on), while keeping access **private**.

The official docs describe the safest default as:
- keep `gateway.bind: "loopback"`
- use **SSH tunneling** or **Tailscale Serve** for remote access

Source: `docs/gateway/remote.md`.

## Baseline architecture

- VPS runs the **Gateway** and owns state (`~/.clawdbot/*`).
- Your laptop/Mac and any nodes connect to it.

## Recommended access pattern (SSH tunnel)

1) On the VPS: install + run onboarding (headless-friendly).

2) From your laptop, create a tunnel:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@vps
```

3) Your local CLI now talks to the VPS gateway via `ws://127.0.0.1:18789`.

## Recommended access pattern (Tailscale Serve)

If you use Tailscale:
- keep `gateway.bind: "loopback"`
- set `gateway.tailscale.mode: "serve"`

Then access:
- Control UI over HTTPS on tailnet

Docs: https://docs.clawd.bot/gateway/tailscale

## Minimum hardening checklist for a VPS

- Use a dedicated non-root user for Clawdbot.
- Keep Gateway loopback-only.
- Run `clawdbot security audit --fix`.
- Make sure `~/.clawdbot` is mode `700` and secrets are not world-readable.
- Don’t install random plugins; treat plugins as in-process code.

See: https://docs.clawd.bot/gateway/security
