# Deployment: Standalone Mac mini (local-first, high privacy)

> **Note:** This guide is for OpenClaw (formerly Moltbot/Clawdbot).

## Table of contents (Explain OpenClaw - GLM-5.0 Edition)

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
  - [High privacy config example](../04-privacy-safety/high-privacy-config.example.json5.md)
- Installation
  - [Install and onboarding](../03-install/install-and-onboard.md)
  - [Build from source](../03-install/from-source.md)
- Deployment
  - [Standalone Mac mini](./mac-mini-standalone.md)
  - [Isolated VPS](../05-use-cases/vps-isolated.md)

---

Goal: run OpenClaw on a dedicated Mac mini at home with **minimal network exposure**.

If you can, this is the safest default deployment: you control the hardware, disk encryption is easy, and "remote exposure" can be optional.

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
- Use GLM-5.0 with appropriate temperature settings
- Run `openclaw security audit --deep` after setup and after any config change

---

## Step-by-step setup

### 1) Create a dedicated user (optional but recommended)

If you treat this Mac mini as an "assistant appliance", create a dedicated macOS user (e.g. `openclaw`) and run the service under that user. This reduces accidental data leakage into your main user's home directory.

Create the user via System Settings, or use `dscl` for scripted setup:

```bash
# Create a dedicated user account
sudo dscl . -create /Users/openclaw
sudo dscl . -append /Groups/admin openclaw
```

### 2) Install OpenClaw

```bash
# Quick install script
curl -fsSL https://openclaw.ai/install.sh | bash

# Or via npm
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

### 4) Select GLM-5.0 during onboarding

When prompted for provider selection:
1. Choose "Zhipu AI"
2. Select "GLM-5.0" or your preferred variant
3. Enter your Zhipu AI API key when prompted

**Alternative: Configure via CLI after onboarding**

```bash
# Set provider to Zhipu AI
openclaw config set agents.defaults.provider zhipu

# Set GLM-5.0 as default model
openclaw config set agents.defaults.model glm-5.0
```

### 5) Verify basics

```bash
# Check if gateway process is running and listening
openclaw gateway status
# Show overall OpenClaw status (all components)
openclaw status
# Run a health check — confirms service is responding
openclaw health
# Deep security scan — checks config, permissions, and known issues
openclaw security audit --deep
```

If the audit suggests fixes:

```bash
# Automatically apply recommended security fixes
openclaw security audit --fix
```

### 6) Open the dashboard (Control UI)

Local (same machine):
- http://127.0.0.1:18789/

If auth is enabled and you don't have the token in the browser yet:

```bash
openclaw dashboard
```

This will open your default browser with a token-authenticated URL.

---

## Connecting messaging channels (high-level guidance)

OpenClaw supports many channels; two common ones for GLM-5.0 users:

### Telegram

1. Create a bot via @BotFather:
   - Send `/newbot`
   - Follow prompts to set name and description
   - Copy the bot token

2. Add your Telegram user ID to allowlist:

```bash
# Find your Telegram user ID - message @userinfobot on Telegram
openclaw config set channels.telegram.allowFrom '["YOUR_TELEGRAM_USER_ID"]'
```

3. For pairing (if enabled), approve yourself:

```bash
# List pending pairing requests
openclaw pairing list telegram

# Approve a pairing code
openclaw pairing approve telegram <CODE>
```

**Note:** GLM-5.0's improved multilingual support works especially well with Telegram's international user base.

Docs: https://docs.openclaw.ai/channels/telegram

### WhatsApp

1. Run the login flow:

```bash
openclaw channels login
```

2. Select WhatsApp
3. Scan QR code with your phone
4. Approve paired devices as needed

Docs: https://docs.openclaw.ai/channels/whatsapp

---

## GLM-5.0 specific considerations

### Model selection for different use cases

| Use Case | Recommended Model | Why |
|-----------|------------------|-----|
| General chatbot | `glm-5.0` | Full ~200K context, best reasoning |
| Speed/cost sensitive | `glm-5.0-flash` | Faster responses, lower cost |
| Document analysis | `glm-5.0-long` | Extended ~1M context for long documents |
| Code generation | `glm-5.0` with `temperature: 0.3` | More focused, fewer errors |

### Temperature configuration

For GLM-5.0, temperature settings significantly affect output style:

```bash
# For analytical/precise work (coding, analysis)
openclaw config set agents.defaults.temperature 0.3

# For creative work (brainstorming, writing)
openclaw config set agents.defaults.temperature 0.8
```

### Function calling with GLM-5.0

GLM-5.0 has native function calling support. Enable with caution:

```bash
# Enable function calling at agent level
openclaw config set agents.defaults.functionCalling true

# Set specific tools allowed for function calling
openclaw config set agents.defaults.tools.allowlist '["web_search", "memory_search"]'
```

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

Keep Gateway on loopback and use Tailscale Serve to publish the Control UI to your tailnet over HTTPS.

Docs: https://docs.openclaw.ai/gateway/tailscale

---

## Host hardening checklist (Mac mini)

Based on security best practices and code review.

### Operating System
- [ ] Enable FileVault (full disk encryption)
- [ ] Keep macOS updated: `softwareupdate -ia`
- [ ] Enable firewall: System Settings → Network → Firewall → Turn On

### User Isolation
- [ ] Create a dedicated user (see Step 1 above)
- [ ] Run gateway service under that user
- [ ] Ensure separate user doesn't have admin access

### Node.js Version
Ensure Node.js 22.12.0+ (includes critical security patches):

```bash
node --version  # Should be v22.12.0 or later
```

### Gateway Security
Set a gateway auth token for production:

```bash
# Generate and set a secure token
export GATEWAY_AUTH_TOKEN="$(openssl rand -hex 32)"
# Add to ~/.zprofile or pass via config
```

### Credential Protection
- [ ] Treat `~/.openclaw/` as secret material (mode 0700)
- [ ] Avoid installing random global npm packages
- [ ] Protect shell history from credential leakage

Protect shell history from credential leakage:

```bash
# Add to ~/.zshrc or ~/.zprofile
export HISTCONTROL=ignoreboth
export HISTFILESIZE=0
```

### Sandbox Configuration

Enable Docker sandbox for code execution tools:

```bash
# In openclaw.json or via CLI
# "agents.defaults.sandbox": "docker"
# "agents.defaults.sandbox.sandboxNetwork": "none"
```

This isolates any successful prompt injection to the container environment.

---

## Backups (privacy-first)

If you back up anything, back up only what you understand, and encrypt it.

Consider backing up:
- your `openclaw.json` config
- only the credentials you are comfortable restoring

Avoid backing up:
- session transcripts (unless you explicitly need them)
- workspace memory files (may contain sensitive data)

### Create a backup

```bash
# Full backup — compresses entire .openclaw/ directory
tar czf openclaw-backup-$(date +%Y%m%d).tar.gz -C ~/ .openclaw/

# Privacy-conscious backup — skips chat logs
tar czf openclaw-backup-$(date +%Y%m%d).tar.gz \
  --exclude='.openclaw/sessions' \
  --exclude='.openclaw/workspace' \
  -C ~/ .openclaw/
```

### Restore from backup

```bash
# Stop the gateway so files aren't being written while we restore
launchctl stop com.openclaw.gateway

# Extract backup archive into your home directory
tar xzf openclaw-backup-YYYYMMDD.tar.gz -C ~/

# Lock down permissions
chmod 700 ~/.openclaw
chmod 600 ~/.openclaw/openclaw.json

# Start gateway again with restored config
launchctl start com.openclaw.gateway
```

### Recommendations

- Store backups in an **encrypted location** (e.g., external drive, Time Machine backup with encryption)
- Schedule weekly backups via launchd:
  ```bash
  # Create a weekly backup plist
  sudo tee /Library/LaunchDaemons/com.openclaw.backup.plist << 'EOF'
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
  <dict>
    <key>Label</key>
    <string>OpenClaw Weekly Backup</string>
    <key>ProgramArguments</key>
    <array>
      <string>/usr/local/bin/openclaw-backup.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <integer>604800</integer>
    <key>WorkingDirectory</key>
    <string>~</string>
  </dict>
  </plist>
  EOF
  ```

---

## Common problems

### Control UI says unauthorized

- Run `openclaw dashboard` and open the printed tokenized URL
- Ensure you are connecting to the correct Gateway instance/profile

### Port already in use (18789)

- Stop the supervised service (`launchctl stop com.openclaw.gateway`)
- Foreground reclaim: `openclaw gateway --force`

### Nothing responds in Telegram/WhatsApp

- Check channel status and logs
- Confirm pairing/allowlists aren't blocking
- Verify model auth is present on **gateway host**
- Check Zhipu AI API key is valid

---

Docs: https://docs.openclaw.ai/help/faq
