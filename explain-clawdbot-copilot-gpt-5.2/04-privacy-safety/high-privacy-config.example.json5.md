# Example high-privacy config (JSON5)

Clawdbot reads config from `~/.clawdbot/clawdbot.json` (JSON5).
Source: `docs/gateway/configuration.md`.

This example focuses on **high privacy** defaults:
- loopback-only gateway
- strict DM access
- minimal discovery

> Note: config surface evolves; verify exact keys against `docs/gateway/configuration.md` for your version.

```json5
{
  gateway: {
    bind: "loopback",

    // Optional but recommended even on localhost.
    // Prefer setting the token via env for services.
    auth: {
      mode: "token",
      token: "${CLAWDBOT_GATEWAY_TOKEN}",
    },

    discovery: {
      mdns: { mode: "off" },
      wideArea: { enabled: false },
    },
  },

  // DM isolation: if multiple people can DM you, consider per-sender or per-peer scoping
  session: {
    dmScope: "per-channel-peer",
  },

  channels: {
    telegram: {
      dmPolicy: "pairing",
      // Once paired, approvals are stored under ~/.clawdbot/credentials/telegram-allowFrom.json
      groups: { "*": { groupPolicy: "disabled" } },
    },

    // Example pattern; fill in only the channels you actually use.
    whatsapp: {
      dmPolicy: "pairing",
      groups: { "*": { groupPolicy: "allowlist", requireMention: true } },
    },
  },
}
```
