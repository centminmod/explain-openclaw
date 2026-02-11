# Deployment runbook: Standalone Mac mini (local-first, high privacy)

> **Note:** This guide is for OpenClaw (formerly Moltbot/Clawdbot).

## Table of contents (Explain OpenClaw)

- [Home (README)](../README.md)
- Plain English
  - [What is OpenClaw?](../01-plain-english/what-is-clawdbot.md)
  - [Glossary](../01-plain-english/glossary.md)
- Technical
  - [Architecture](../02-technical/architecture.md)
  - [Repo map](../02-technical/repo-map.md)
- Privacy + safety
  - [Threat model](../04-privacy-safety/threat-model.md)
  - [Hardening checklist](../04-privacy-safety/hardening-checklist.md)
- Deployment
  - [Standalone Mac mini](./standalone-mac-mini.md)
  - [Isolated VPS](./isolated-vps.md)
  - [Cloudflare Moltworker](./cloudflare-moltworker.md)
  - [Docker Model Runner](./docker-model-runner.md)
- Optimizations
  - [Cost + token optimization](../06-optimizations/cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

Goal: run OpenClaw on a dedicated Mac mini at home with **minimal network exposure**.

If you can, this is the safest default deployment: you control the hardware, disk encryption is easy, and “remote exposure” can be optional.

Related official docs:
- https://docs.openclaw.ai/start/getting-started
- https://docs.openclaw.ai/gateway/security
- https://docs.openclaw.ai/gateway/remote
- https://docs.openclaw.ai/gateway/tailscale

---

## Recommended posture (summary)

- `gateway.bind: "loopback"` (localhost only)
- DM policy: `pairing` or `allowlist`
- Only enable the channels you actually need
- Avoid exposing browser control remotely
- Run `openclaw security audit --deep` after setup and after any config change

---

## Step-by-step setup

### 1) Create a dedicated user (optional but recommended)
If you treat this Mac mini as an "assistant appliance", create a dedicated macOS user (e.g. `moltbot`) and run the service under that user. This reduces accidental data leakage into your main user's home directory.

Create the user via System Settings, or use `dscl` for scripted setup:

```bash
sudo dscl . -create /Users/moltbot
sudo dscl . -create /Users/moltbot UserShell /bin/zsh
sudo dscl . -create /Users/moltbot UniqueID 550
sudo dscl . -create /Users/moltbot PrimaryGroupID 20
sudo dscl . -create /Users/moltbot NFSHomeDirectory /Users/moltbot
sudo mkdir -p /Users/moltbot
sudo chown moltbot:staff /Users/moltbot
```

### 2) Install OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Or:

```bash
npm install -g openclaw@latest
```

Verify Node.js version (22.12.0+ recommended for security patches):

```bash
node --version  # Should be v22.12.0 or later
```

### 3) Onboard and install the background service

```bash
openclaw onboard --install-daemon
```

This typically sets up a per-user service (launchd) and writes config under `~/.openclaw/`.

For automated/headless setup with a custom LLM provider (Ollama, LM Studio, etc.):

```bash
export CUSTOM_API_KEY="your-api-key-here"
openclaw onboard --non-interactive --install-daemon \
  --custom-base-url "http://localhost:11434/v1" \
  --custom-model-id "llama3" \
  --custom-compatibility openai
```

See [Non-interactive onboarding flags](../99-reference/commands-and-troubleshooting.md#non-interactive-custom-provider-onboarding) for all options.

### 4) Verify basics

```bash
openclaw gateway status
openclaw status
openclaw health
openclaw security audit --deep
```

If the audit suggests fixes:

```bash
openclaw security audit --fix
```

### 5) Open the dashboard (Control UI)

Local (same machine):
- http://127.0.0.1:18789/

If auth is enabled and you don’t have the token in the browser yet:

```bash
openclaw dashboard
```

---

## Connecting messaging channels (high-level guidance)

OpenClaw supports many channels; two common ones:

### WhatsApp
- Uses WhatsApp Web / Baileys.
- Login flow typically uses QR code:

```bash
openclaw channels login
```

Docs: https://docs.openclaw.ai/channels/whatsapp

### Telegram
- Uses a bot token created via @BotFather.
- DM pairing is commonly enabled by default; approve yourself:

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Docs: https://docs.openclaw.ai/channels/telegram

---

## Optional: remote access (still private)

### Option A (universal): SSH tunnel
From your laptop:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@mac-mini
```

Then open:
- http://127.0.0.1:18789/

### Option B (best UX): Tailscale Serve
Keep `gateway.bind: "loopback"` and use Tailscale Serve to publish the Control UI to your tailnet over HTTPS.

Docs: https://docs.openclaw.ai/gateway/tailscale

---

## Host hardening checklist (Mac mini)

Based on [VibeProof Security Guide](https://vibeproof.dev/blog/moltbot-security-setup-guide) (uses legacy "Moltbot" name) and code review.

### Operating System
- [ ] Enable FileVault (full disk encryption)
- [ ] Keep macOS updated: `softwareupdate -ia`
- [ ] Enable firewall: System Settings → Network → Firewall → Turn On

### User Isolation
- [ ] Create a dedicated user (see Step 1 above)
- [ ] Run the gateway service under that user

### Node.js Version
Ensure Node.js 22.12.0+ (includes critical security patches):

```bash
node --version  # Should be v22.12.0 or later
```

### Gateway Security
Set a gateway auth token for production:

```bash
export GATEWAY_AUTH_TOKEN="$(openssl rand -hex 32)"
# Add to ~/.zprofile or pass via config
```

### Credential Protection
- [ ] Treat `~/.openclaw/` as secret material (mode 0700)
- [ ] Avoid installing random global npm packages

Protect shell history from credential leakage:

```bash
# Add to ~/.zshrc or ~/.zprofile
export HISTCONTROL=ignoreboth
export HISTFILESIZE=0
```

### Sandbox Configuration
Enable Docker sandbox for code execution tools:

```bash
# In openclaw.json
# "agents.defaults.sandbox": "docker"
# "agents.defaults.sandboxNetwork": "none"
```

This isolates any successful prompt injection to the container environment.

> **See:** [Prompt Injection Attacks](../05-worst-case-security/prompt-injection-attacks.md) for 27 examples of the attack patterns sandbox isolation defends against.

---

## Backups (privacy-first)

If you back up anything, back up only what you understand, and encrypt it.

Consider backing up:
- your `openclaw.json` config
- only the credentials you are comfortable restoring

Avoid backing up:
- session transcripts (unless you explicitly need them)

Docs: https://docs.openclaw.ai/gateway/security
