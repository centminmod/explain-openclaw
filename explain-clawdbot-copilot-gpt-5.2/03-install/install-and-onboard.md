# Install + onboard (fast path)

This follows the repo’s canonical install docs (`docs/install/index.md`) and the beginner flow (`docs/start/getting-started.md`).

## Prereqs

- Node **>= 22** (see `package.json` + `docs/install/index.md`)

## Install (recommended)

```bash
curl -fsSL https://clawd.bot/install.sh | bash
```

Alternative (manual global install):

```bash
npm install -g clawdbot@latest
# or: pnpm add -g clawdbot@latest
```

## Run onboarding (recommended)

```bash
clawdbot onboard --install-daemon
```

The wizard sets up:
- model/auth
- gateway settings
- channels
- pairing defaults
- optional background service

See: `docs/start/getting-started.md` and https://docs.clawd.bot/start/wizard

## Start / verify

```bash
clawdbot gateway status
clawdbot status
clawdbot health
clawdbot security audit --deep
```

## First chat (no channels yet)

```bash
clawdbot dashboard
# then open http://127.0.0.1:18789/
```

## Pairing (DM safety)

If your first DM to a channel returns a pairing code (or you get no reply), approve:

```bash
clawdbot pairing list telegram
clawdbot pairing approve telegram <CODE>
```

Docs: `docs/start/pairing.md`.
