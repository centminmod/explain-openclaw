# Install and Onboard (with Z.AI / GLM-5.0)

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
  - [Install and onboarding](./install-and-onboard.md)
  - [Build from source](./from-source.md)
- Deployment
  - [Standalone Mac mini](../05-use-cases/mac-mini-standalone.md)
  - [Isolated VPS](../05-use-cases/vps-isolated.md)

---

This guide covers installing OpenClaw and setting up GLM-5.0 as your AI model.

Related official docs:
- https://docs.openclaw.ai/install
- https://docs.openclaw.ai/start/wizard
- https://docs.openclaw.ai/gateway/configuration

---

## Prerequisites

### Node.js version

OpenClaw requires Node.js 22.12.0 or later (includes critical security patches).

Verify your version:

```bash
node --version  # Should be v22.12.0 or later
```

If you need to upgrade:
```bash
# macOS
brew install node

# Linux (Ubuntu/Debian)
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
```

### Platform support

| Platform | Support | Notes |
|----------|----------|-------|
| macOS | Full support including native iMessage integration |
| Linux | Full support (Ubuntu 20.04+, Debian 11+) |
| Windows | WSL2 support; native Windows in development |

---

## Step 1: Install OpenClaw

### Quick install (recommended)

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

This installs the `openclaw` CLI and its dependencies.

### Alternative: npm global install

```bash
npm install -g openclaw@latest
```

### Verify installation

```bash
openclaw --version
```

---

## Step 2: Get Z.AI API Key

You'll need a Z.AI API key to use GLM-5.0 models.

### Get your API key

1. Visit the Z.AI console
2. Sign up or log in
3. Create a new API key
4. Copy the key for use in OpenClaw setup

**Security note:** Treat your Z.AI API key like a password:
- Never commit it to git
- Don't share it in screenshots
- Don't log it in plaintext
- Use environment variables where possible

### API key formats

Z.AI API keys typically start with a prefix like:
- `sk-xxxxxxxxxxxxxxxxxxxxxxxxxx`

---

## Step 3: Run the onboarding wizard

The onboarding wizard will guide you through:

1. **Provider selection** - Choose "Z.AI" when prompted
2. **Model selection** - Select GLM-5.0 or your preferred variant
3. **API key entry** - Enter your Z.AI API key
4. **Channel setup** - Configure messaging channels (optional)
5. **Background service** - Install the Gateway as a background service

```bash
openclaw onboard --install-daemon
```

### What the wizard does

- Creates `~/.openclaw/` directory structure
- Writes `openclaw.json` configuration file
- Sets up agent defaults with your selected provider
- Installs launchd/systemd service for always-on Gateway
- Optionally starts the Gateway immediately

---

## Step 4: Configure GLM-5.0 provider

After onboarding, you may want to customize your GLM-5.0 configuration.

### Provider configuration via CLI

```bash
# Set the default provider to Z.AI
openclaw config set agents.defaults.provider zai

# Set the default model
openclaw config set agents.defaults.model glm-5.0

# Or set both at once
openclaw config set agents.defaults.provider zai agents.defaults.model glm-5.0
```

### Model variants

GLM-5.0 offers several variants for different use cases:

| Model | Context | Best For | Pricing |
|-------|---------|-----------|----------|
| `glm-5.0` | ~200K tokens | General purpose | Standard |
| `glm-5.0-flash` | ~128K tokens | Speed/cost sensitive | Lower cost |
| `glm-5.0-long` | ~1M tokens | Long documents | Premium |

### Temperature and sampling

For creative tasks (brainstorming, creative writing):
```bash
openclaw config set agents.defaults.temperature 1.0
```

For analytical/precise tasks (coding, analysis):
```bash
openclaw config set agents.defaults.temperature 0.3
```

---

## Step 5: Verify installation

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

If the audit finds issues, it can auto-fix many of them:

```bash
# Automatically apply recommended security fixes
openclaw security audit --fix
```

---

## Step 6: Open the dashboard

Once the Gateway is running, access the Control UI:

### Local access

Default URL: http://127.0.0.1:18789/

If auth is enabled and you don't have the token in the browser yet:

```bash
openclaw dashboard
```

This prints and opens a URL with the authentication token included.

### Remote access (Mac mini / VPS)

See the deployment guides for remote access options:
- [Standalone Mac mini](../05-use-cases/mac-mini-standalone.md) — SSH tunnel, Tailscale
- [Isolated VPS](../05-use-cases/vps-isolated.md) — Tailscale, reverse proxy

---

## Troubleshooting

### "Command not found" after install

If `openclaw` command isn't found:
- Close and reopen your terminal
- Ensure `npm bin -g` output is in your PATH
- Try using `npx openclaw` as an alternative

### "Provider not recognized" error

If Z.AI provider isn't available:
- Ensure you're running the latest OpenClaw version
- Check `openclaw models list` to see available providers

### Gateway won't start

```bash
# Check for config errors
openclaw gateway status

# View logs for errors
openclaw logs --follow

# Verify config syntax
openclaw config validate
```

---

## Upgrading

```bash
# Check current version
openclaw --version

# Upgrade to latest
npm update -g openclaw@latest

# Or reinstall
curl -fsSL https://openclaw.ai/install.sh | bash
```

After upgrading, run `openclaw security audit --deep` to check for any new security recommendations.

---

## Uninstalling

```bash
# Stop the service
openclaw gateway stop

# Remove the service
openclaw gateway uninstall

# Optionally remove configuration and state (CAUTION: this deletes your config and sessions)
rm -rf ~/.openclaw
```

---

Next: [Build from source](./from-source.md) for advanced installation options
