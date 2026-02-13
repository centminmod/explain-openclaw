# Installation Guide

This guide walks you through installing Clawdbot from scratch.

## Prerequisites

### Required

| Requirement | Version | Check Command |
|-------------|---------|---------------|
| Node.js | 22+ | `node --version` |
| npm/pnpm/bun | Any recent | `npm --version` |

### Optional

| Tool | Purpose |
|------|---------|
| Ollama | Local AI models |
| signal-cli | Signal support |
| Docker | Containerized deployment |

---

## Installation Methods

### Method 1: npm (Recommended)

```bash
npm install -g clawdbot@latest
```

Verify installation:
```bash
clawdbot --version
```

### Method 2: pnpm

```bash
pnpm add -g clawdbot@latest
```

### Method 3: From Source

For development or customization:

```bash
# Clone repository
git clone https://github.com/clawdbot/clawdbot.git
cd clawdbot

# Install dependencies
pnpm install

# Build
pnpm build

# Run from source
pnpm clawdbot --version
```

---

## First-Time Setup

### Interactive Wizard (Recommended)

The onboarding wizard guides you through setup:

```bash
clawdbot onboard --install-daemon
```

This will:
1. Check system requirements
2. Configure AI provider (API key)
3. Set up messaging channels
4. Install background daemon
5. Start the gateway

### Manual Setup

If you prefer manual configuration:

```bash
# 1. Check system health
clawdbot doctor

# 2. Authenticate with AI provider
clawdbot login
# Select provider, enter API key

# 3. Start gateway
clawdbot gateway run --port 18789
```

---

## Configuration File

After setup, your config is at `~/.clawdbot/clawdbot.json`:

```json5
{
  // AI model settings
  "agent": {
    "model": "claude-opus-4-5",
    "provider": "anthropic"
  },

  // Gateway settings
  "gateway": {
    "port": 18789
  },

  // Messaging channels
  "channels": {
    "telegram": {
      "enabled": false
    }
  }
}
```

---

## Setting Up a Channel

### Telegram Example

1. Create a bot via [@BotFather](https://t.me/BotFather):
   ```
   /newbot
   Name: My AI Bot
   Username: my_ai_bot
   ```

2. Copy the token (looks like: `123456789:ABCdefGHI...`)

3. Set environment variable:
   ```bash
   export TELEGRAM_BOT_TOKEN="123456789:ABCdefGHI..."
   ```

4. Edit config:
   ```json5
   {
     "channels": {
       "telegram": {
         "enabled": true,
         "accounts": {
           "default": {
             "botToken": "${TELEGRAM_BOT_TOKEN}",
             "allowFrom": ["@yourusername"]
           }
         }
       }
     }
   }
   ```

5. Restart gateway:
   ```bash
   clawdbot daemon restart
   # or
   clawdbot gateway run
   ```

6. Message your bot on Telegram

---

## Environment Variables

Create a `.env` file or add to your shell profile (`~/.zshrc`, `~/.bashrc`):

```bash
# AI Providers
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."
export GEMINI_API_KEY="..."

# Channels
export TELEGRAM_BOT_TOKEN="..."
export DISCORD_BOT_TOKEN="..."
export SLACK_BOT_TOKEN="xoxb-..."
export SLACK_APP_TOKEN="xapp-..."

# Gateway Security
export CLAWDBOT_GATEWAY_TOKEN="your-secret-token"
```

---

## Running the Gateway

### Foreground (Development)

```bash
clawdbot gateway run --port 18789
```

Press Ctrl+C to stop.

### Background (Daemon)

```bash
# Start daemon
clawdbot daemon start

# Check status
clawdbot daemon status

# Stop daemon
clawdbot daemon stop

# Restart
clawdbot daemon restart
```

### System Service (Production)

On Linux, create a systemd service:

```ini
# /etc/systemd/system/clawdbot.service
[Unit]
Description=Clawdbot Gateway
After=network.target

[Service]
Type=simple
User=clawdbot
ExecStart=/usr/bin/clawdbot gateway run --port 18789
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable clawdbot
sudo systemctl start clawdbot
```

---

## Verifying Installation

### Health Check

```bash
clawdbot doctor
```

Expected output:
```
✓ Node.js version 22.x
✓ Config file found
✓ Credentials configured
✓ Gateway reachable
```

### Channel Status

```bash
clawdbot channels status --probe
```

Expected output:
```
CHANNEL    STATUS    ACCOUNT
telegram   ✓ OK      default
whatsapp   ✓ OK      default
discord    ○ disabled
```

### Test Message

```bash
clawdbot message send --to "telegram:@yourusername" --text "Hello from Clawdbot!"
```

---

## Updating Clawdbot

```bash
# Via npm
npm update -g clawdbot

# Via clawdbot
clawdbot update

# Check version
clawdbot --version
```

---

## Uninstalling

```bash
# Stop daemon
clawdbot daemon stop

# Uninstall package
npm uninstall -g clawdbot

# Remove data (optional)
rm -rf ~/.clawdbot
```

---

## Troubleshooting

### "Command not found: clawdbot"

Ensure npm global bin is in PATH:
```bash
export PATH="$PATH:$(npm config get prefix)/bin"
```

### "Port 18789 already in use"

Another instance is running:
```bash
clawdbot daemon stop
# or
lsof -i :18789
kill <PID>
```

### "API key invalid"

1. Check the key is correct (no extra spaces)
2. Verify environment variable is set:
   ```bash
   echo $ANTHROPIC_API_KEY
   ```
3. Re-run `clawdbot login`

### "Permission denied"

On Linux/macOS:
```bash
# If installed globally as root
sudo chown -R $(whoami) ~/.clawdbot
chmod 700 ~/.clawdbot
chmod 600 ~/.clawdbot/clawdbot.json
```

---

## Next Steps

- [Configuration](./06-configuration.md) - Detailed config options
- [Security & Privacy](./07-security-privacy.md) - Secure your setup
- [Mac Mini Deployment](./08-mac-mini-deployment.md) - Home server setup
- [VPS Deployment](./09-vps-deployment.md) - Cloud server setup

---

*Continue to [06 - Configuration](./06-configuration.md) →*
