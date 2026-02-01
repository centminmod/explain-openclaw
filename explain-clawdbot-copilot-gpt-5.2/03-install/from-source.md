# Running from source (development)

This mirrors the repo’s `README.md` “From source (development)” section.

```bash
git clone https://github.com/clawdbot/clawdbot.git
cd clawdbot

pnpm install
pnpm ui:build
pnpm build

pnpm clawdbot onboard --install-daemon

# dev loop (auto-reload on TS changes)
pnpm gateway:watch
```

Notes:
- `pnpm clawdbot ...` runs TypeScript directly (via `tsx`).
- `pnpm build` generates `dist/`.

For WhatsApp + Telegram, the docs warn that Bun has issues; prefer Node for the Gateway.
See: `docs/start/getting-started.md`.
