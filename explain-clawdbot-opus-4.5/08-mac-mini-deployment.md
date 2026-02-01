# Mac Mini Deployment

This guide covers deploying Clawdbot on a dedicated Mac Mini as a personal AI assistant server.

## Use Case

A Mac Mini running 24/7 as your personal AI gateway:
- Always available AI assistant
- Accessible from any messaging app
- Your data stays on your hardware
- Optional: fully local AI with Ollama

---

## Hardware Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| Mac Mini | M1 | M2/M4 |
| RAM | 8GB | 16GB+ (for Ollama) |
| Storage | 128GB | 256GB+ |
| Network | Wi-Fi | Ethernet |

---

## Initial Setup

### 1. macOS Configuration

```bash
# Enable remote login (SSH)
sudo systemsetup -setremotelogin on

# Prevent sleep
sudo pmset -a sleep 0
sudo pmset -a disksleep 0
sudo pmset -a displaysleep 0

# Set to restart after power failure
sudo pmset -a autorestart 1
```

### 2. Install Prerequisites

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Node.js 22
brew install node@22

# Verify
node --version  # Should be 22.x
```

### 3. Install Clawdbot

```bash
npm install -g clawdbot@latest
clawdbot --version
```

### 4. Install Ollama (Optional, for Local AI)

```bash
# Install Ollama
brew install ollama

# Start Ollama service
brew services start ollama

# Pull a model
ollama pull llama3.2
```

---

## Configuration

### Privacy-First Config

Create `~/.clawdbot/clawdbot.json`:

```json5
{
  // Use local Ollama model - no cloud API calls
  "agent": {
    "model": "ollama:llama3.2",
    "provider": "ollama"
  },

  // Gateway settings
  "gateway": {
    "port": 18789,
    "bind": "loopback",  // localhost only
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "${CLAWDBOT_GATEWAY_TOKEN}"
    }
  },

  // Only enable channels you need
  "channels": {
    "telegram": {
      "enabled": true,
      "accounts": {
        "default": {
          "botToken": "${TELEGRAM_BOT_TOKEN}",
          "allowFrom": ["@yourusername"]  // Only you
        }
      }
    },
    "imessage": {
      "enabled": true,
      "accounts": {
        "default": {
          "allowFrom": ["+1234567890"]  // Your phone number
        }
      }
    }
  },

  // Minimal logging
  "logging": {
    "level": "warn"
  }
}
```

### Cloud AI Config (If Not Using Ollama)

```json5
{
  "agent": {
    "model": "claude-opus-4-5",
    "provider": "anthropic"
  }
  // ... rest same as above
}
```

---

## Environment Variables

Create `~/.zshrc` additions:

```bash
# Clawdbot secrets
export CLAWDBOT_GATEWAY_TOKEN="$(openssl rand -hex 32)"
export TELEGRAM_BOT_TOKEN="your-telegram-token"
export ANTHROPIC_API_KEY="sk-ant-..."  # If using cloud AI
```

Reload:
```bash
source ~/.zshrc
```

---

## Auto-Start on Boot

### Method 1: Clawdbot Daemon

```bash
clawdbot daemon start
```

This uses launchd automatically.

### Method 2: Manual Launch Agent

Create `~/Library/LaunchAgents/com.clawdbot.gateway.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.clawdbot.gateway</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/clawdbot</string>
        <string>gateway</string>
        <string>run</string>
        <string>--port</string>
        <string>18789</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/clawdbot-gateway.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/clawdbot-gateway.error.log</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/usr/local/bin:/usr/bin:/bin</string>
    </dict>
</dict>
</plist>
```

Load it:
```bash
launchctl load ~/Library/LaunchAgents/com.clawdbot.gateway.plist
```

---

## iMessage Setup (macOS Only)

The Mac Mini can respond to iMessages:

1. Sign into iMessage on the Mac Mini

2. Grant Full Disk Access:
   - System Preferences → Privacy & Security → Full Disk Access
   - Add Terminal (or your terminal app)

3. Enable in config:
   ```json5
   {
     "channels": {
       "imessage": {
         "enabled": true,
         "accounts": {
           "default": {
             "allowFrom": ["+1234567890"]
           }
         }
       }
     }
   }
   ```

---

## Remote Access

### Option 1: Tailscale (Recommended)

Secure, zero-config VPN:

```bash
# Install Tailscale
brew install tailscale

# Connect
tailscale up

# Get your Tailscale IP
tailscale ip
```

Now access your Mac Mini from anywhere on your Tailscale network.

Update config for Tailscale auth:
```json5
{
  "gateway": {
    "bind": "lan",  // Allow network access
    "auth": {
      "mode": "tailscale"  // Use Tailscale identity
    }
  }
}
```

### Option 2: SSH Tunnel

From your remote machine:
```bash
ssh -L 18789:localhost:18789 user@mac-mini.local
```

Then access `localhost:18789` locally.

---

## Monitoring

### Check Status

```bash
# Gateway status
clawdbot daemon status

# Channel connections
clawdbot channels status --probe

# View logs
tail -f /tmp/clawdbot-gateway.log
```

### Health Check Script

Create `~/check-clawdbot.sh`:

```bash
#!/bin/bash
if ! pgrep -f "clawdbot gateway" > /dev/null; then
    echo "Clawdbot not running, restarting..."
    clawdbot daemon start
fi
```

Add to crontab:
```bash
crontab -e
# Add:
*/5 * * * * ~/check-clawdbot.sh
```

---

## Maintenance

### Updates

```bash
# Update Clawdbot
npm update -g clawdbot
clawdbot daemon restart

# Update Ollama models
ollama pull llama3.2
```

### Log Rotation

Logs can grow large. Set up rotation:

```bash
# Create /etc/newsyslog.d/clawdbot.conf
/tmp/clawdbot-gateway.log 644 5 1000 * J
/tmp/clawdbot-gateway.error.log 644 5 1000 * J
```

### Backup

Important files to backup:
- `~/.clawdbot/clawdbot.json` - Configuration
- `~/.clawdbot/credentials/` - API keys
- `~/.clawdbot/agents/` - Conversation history

---

## Firewall Configuration

### Allow Only Local + Tailscale

```bash
# Block all incoming
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on

# Allow SSH (if needed)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /usr/sbin/sshd

# Tailscale handles its own traffic
```

### Verify

```bash
# Check what's listening
lsof -i :18789
# Should show only loopback or Tailscale
```

---

## Troubleshooting

### "Ollama not responding"

```bash
# Check Ollama service
brew services list | grep ollama

# Restart
brew services restart ollama

# Verify
curl http://localhost:11434/api/tags
```

### "iMessage not working"

1. Verify Full Disk Access granted
2. Check Messages app is signed in
3. Run: `clawdbot doctor`

### "Gateway won't start"

```bash
# Check port in use
lsof -i :18789

# Kill existing process
pkill -f "clawdbot gateway"

# Start fresh
clawdbot daemon start
```

---

## Complete Setup Checklist

- [ ] macOS configured (no sleep, auto-restart)
- [ ] Node.js 22+ installed
- [ ] Clawdbot installed
- [ ] Ollama installed (if using local AI)
- [ ] Config file created
- [ ] Environment variables set
- [ ] Daemon auto-start configured
- [ ] Channels enabled and tested
- [ ] Remote access configured (Tailscale)
- [ ] Monitoring set up
- [ ] Backup plan in place

---

*Continue to [09 - VPS Deployment](./09-vps-deployment.md) →*
