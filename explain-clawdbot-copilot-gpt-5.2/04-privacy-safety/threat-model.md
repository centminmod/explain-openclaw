# Threat model (beginner-friendly)

Clawdbot is *not just chat*. Depending on what you enable, it can:
- execute commands
- read/write files
- fetch URLs / browse the web
- send messages outward on real chat apps
- call paired devices (camera/screen recording/notifications)

That means the main risk is often simple:

> Someone (or some untrusted content) convinces the agent to do something unsafe.

This is called **prompt injection**.

## Threat surfaces

1) **Inbound messages**
- A stranger DMing your bot
- A group chat member tagging the bot

2) **Untrusted content the bot reads**
- Web pages (browser / web fetch)
- Attachments (docs/images)
- Pasted “instructions” in chat

Even if only *you* can DM the bot, untrusted content can still carry adversarial instructions.
The repo’s security doc calls this out explicitly.

## Your real security boundary

Clawdbot stores session transcripts and credentials on disk. Anyone with access to:
- the Gateway machine
- its filesystem
- its process environment

…can often read sensitive state.
So: **treat the Gateway host as the trust boundary** and harden it.

## Core principle: access control before intelligence

The docs recommend this order:

1) Decide **who can trigger the bot** (pairing / allowlists).
2) Decide **what the bot is allowed to do** (tool allowlists, sandboxing, device approvals).
3) Choose a model last (assume models can be manipulated).

Source: `docs/gateway/security.md`.

Next: [Hardening checklist](./hardening-checklist.md).
