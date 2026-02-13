# Use case: Standalone Mac mini (local-first, high privacy)

Goal: run Clawdbot on a Mac mini that stays at home, with **minimal network exposure**.

## Recommended posture

- Keep Gateway bound to loopback (`gateway.bind: "loopback"`).
- Use local Control UI (`http://127.0.0.1:18789/`).
- Use DM pairing / allowlists.
- Only enable tools you need.

## Setup steps

1) Install + onboard:

```bash
curl -fsSL https://clawd.bot/install.sh | bash
clawdbot onboard --install-daemon
```

2) Verify:

```bash
clawdbot status
clawdbot health
clawdbot security audit --deep
```

3) Connect a channel (optional):

```bash
clawdbot channels login  # WhatsApp QR login
```

4) Approve your DM pairing if needed:

```bash
clawdbot pairing list whatsapp
clawdbot pairing approve whatsapp <CODE>
```

## Remote access (optional, still private)

If you want to access the Mac mini gateway from your laptop while traveling:

- Prefer **SSH tunnel** (universal) OR **Tailscale Serve** (tailnet-only, HTTPS).

SSH tunnel (from your laptop):

```bash
ssh -N -L 18789:127.0.0.1:18789 user@mac-mini
```

Then open `http://127.0.0.1:18789/` locally.

Docs:
- https://docs.clawd.bot/gateway/remote
- https://docs.clawd.bot/gateway/tailscale
