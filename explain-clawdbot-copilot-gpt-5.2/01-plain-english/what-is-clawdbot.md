# What is Clawdbot? (plain English)

Clawdbot is a **personal AI assistant** you run on **your own devices**.
Instead of living only inside a web chat, it can reply to you on messaging apps you already use (WhatsApp, Telegram, Slack, Discord, Signal, iMessage, etc.), and it can optionally use tools (like a browser, reading/writing files, scheduling jobs, or calling paired devices).

The main idea is:

- You run a **Gateway** (a local server).
- Your messages arrive through one or more **channels** (WhatsApp/Telegram/etc.).
- The Gateway runs an **AI agent** (a model like Claude/GPT) to decide what to do.
- The Gateway sends the reply back out to the same channel.

Clawdbot is powerful because it can be *more than chat*: it can act.
That’s also why **privacy and access control** matter a lot.

## What it does (capabilities)

In this repo’s README, Clawdbot is described as a personal assistant that can:

- Answer on real chat surfaces (WhatsApp/Telegram/etc.)
- Run an always-on gateway + optional background service
- Connect companion “nodes” (macOS/iOS/Android) for device actions (camera, screen recording, notifications; macOS can also run commands if you enable it)
- Use tools like browser control and scheduling (cron)

See: `README.md` in the repo root.

## What it is *not*

- Not a “safe by default for public DMs” bot. You must configure who can talk to it.
- Not magic security. If you give it tool access (shell/files/network) you are effectively giving it a *robot hand* on your computer.

Clawdbot’s docs emphasize this: `docs/gateway/security.md` (“Running an AI agent with shell access… is spicy.”).

## Beginner mental model

Think of Clawdbot like:

- **A message router** (many chat apps in, many chat apps out)
- **An agent runtime** (it runs the model + tool calls)
- **A control plane** (one place to manage sessions, pairing, tools, and devices)

If you want a high-privacy setup, the safest default is:

- Keep the Gateway on **localhost** (`127.0.0.1` / “loopback”).
- Only allow **you** to message it (pairing/allowlist).
- Only enable tools you truly need.

Next: [Glossary](./glossary.md) → and then [Threat model](../04-privacy-safety/threat-model.md).
