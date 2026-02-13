# Using Clawdbot on a Mac Mini

A Mac Mini is one of the best ways to run Clawdbot — it's quiet, energy-efficient, and can run 24/7 in your home. This guide explains how to set it up.

## Why Mac Mini is ideal

### Advantages

| Benefit | Why it matters |
|---------|----------------|
| **Always on** | Your AI is always available |
| **Low power** | Uses ~10-15W when idle |
| **Small size** | Fits anywhere |
| **Quiet** | Fanless M1/M2/M4 models are silent |
| **Reliable** | macOS is stable and well-supported |
| **Privacy** | Data stays in your home |
| **macOS app** | Nice menu bar interface |

### Recommended specs

- **M1, M2, or M4 Mac Mini** — Apple Silicon is efficient and fast
- **8GB RAM minimum** — 16GB or more recommended for heavy use
- **256GB SSD minimum** — 512GB preferred for storage
- **Ethernet** — More reliable than Wi-Fi for a server

## Installation steps

### 1. Initial macOS setup

When setting up a new Mac Mini for Clawdbot:

1. **Create a dedicated user account** (optional but recommended)
   - System Settings → Users & Groups → Add Account
   - Call it something like "clawdbot" or "ai"

2. **Enable auto-login** (for easy access)
   - System Settings → Users & Groups → Automatic login
   - Not recommended if you need high security

3. **Disable sleep** (so it stays running)
   - System Settings → Energy Saver
   - Set "Turn display off after" to Never
   - Enable "Prevent automatic sleeping"

4. **Enable Remote Login** (for SSH access)
   - System Settings → General → Sharing
   - Enable "Remote Login"

### 2. Install Node.js

```bash
# Install Homebrew if not present
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Node.js 22+
brew install node@22

# Verify installation
node --version
```

### 3. Install Clawdbot

```bash
# Install globally
npm install -g clawdbot@latest

# Verify
clawdbot --version
```

### 4. Run the onboarding wizard

```bash
clawdbot onboard --install-daemon
```

This will:
- Set up your AI provider (Anthropic/OpenAI)
- Configure channels (WhatsApp, Telegram, etc.)
- Install the launchd service for auto-start

## macOS app setup

The Clawdbot macOS app provides a nice menu bar interface:

### Installation

```bash
# Download the latest macOS app from GitHub releases
# Or build from source:
pnpm mac:package

# Open the app
open dist/Clawdbot.app
```

### What the app gives you

- **Menu bar control** — Quick access to status and controls
- **Voice Wake** — Say "Hey Clawd" to trigger the assistant
- **Talk Mode** — Continuous voice conversation
- **WebChat** — Browser-based chat interface
- **Debug tools** — Logs and diagnostics
- **Remote control** — Manage remote gateways

### macOS permissions

On first run, the app will request permissions:

| Permission | Why it's needed | How to grant |
|------------|----------------|--------------|
| **Accessibility** | Keyboard shortcuts, Voice Wake | System Settings → Privacy → Accessibility |
| **Full Disk Access** | Reading/writing files | System Settings → Privacy → Full Disk Access |
| **Notifications** | Sending alerts | System Settings → Notifications |
| **Screen Recording** | Screen capture tools | System Settings → Privacy → Screen Recording |
| **Microphone** | Voice input | System Settings → Privacy → Microphone |
| **Camera** | Camera tools | System Settings → Privacy → Camera |

Grant all requested permissions for full functionality.

## Running as a service

### Using launchd (recommended)

The onboarding wizard can set this up automatically:

```bash
# Check service status
launchctl print gui/$UID | grep clawdbot

# View logs
log show --predicate 'subsystem == "com.clawdbot.gateway"' --last 1h

# Restart the service
clawdbot service restart
```

### Manual control

```bash
# Start the gateway in foreground
clawdbot gateway --port 18789 --verbose

# Start in background
clawdbot service start

# Stop
clawdbot service stop

# Restart
clawdbot service restart
```

## TCC (Transparency, Consent, and Control)

macOS has strict security. Here's how to handle it:

### What TCC does

- Restricts access to sensitive data (contacts, photos, etc.)
- Requires user approval for certain operations
- Can reset permissions on app updates

### Best practices

1. **Keep the app signed** — Unsigned apps lose permissions on restart
2. **Use System Settings** — Grant permissions there, not via alerts
3. **Restart after changes** — Some permissions require app restart

### Diagnosing TCC issues

```bash
# Check if Clawdbot has necessary permissions
clawdbot doctor

# View macOS logs for TCC denials
log show --predicate 'subsystem == "com.apple.TCC"' --last 1h | grep clawdbot
```

## Privacy tips for Mac Mini

### 1. Network isolation

Keep your Mac Mini on a separate VLAN or guest network:

- Creates isolation from your main devices
- Limits blast radius if compromised
- Still accessible via SSH/Tailscale

### 2. Firewall settings

```bash
# Enable firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on

# Block all incoming, then allow exceptions
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setblockall on

# Allow SSH (if needed)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /usr/sbin/sshd
```

### 3. FileVault encryption

Enable FileVault to protect data at rest:

- System Settings → Privacy & Security → FileVault
- Requires password on boot
- Protects data if the Mac Mini is stolen

### 4. Local-only AI (optional)

For maximum privacy, use local models:

```bash
# Install Ollama
brew install ollama

# Start Ollama
ollama serve

# Pull a model
ollama pull llama3

# Configure Clawdbot
clawdbot config set agent.model ollama/llama3
```

## Remote access from other devices

### From your Mac laptop

```bash
# SSH into the Mac Mini
ssh user@mac-mini.local

# Or forward the gateway port
ssh -N -L 18789:127.0.0.1:18789 user@mac-mini.local
```

Now your laptop can connect to `ws://127.0.0.1:18789` and it reaches the Mac Mini.

### Using Tailscale (recommended)

1. Install Tailscale on both devices
2. Log in to the same account
3. Connect using the Tailscale IP:

```bash
# Find the Tailscale IP on Mac Mini
tailscale ip -4

# Connect from another device
clawdbot gateway --url ws://100.x.x.x:18789
```

### Using the macOS app remotely

The macOS app can control remote gateways:

1. Open the app
2. Click "Connect to Remote Gateway"
3. Enter SSH details or Tailscale IP
4. Control the remote Mac Mini as if local

## Example configurations

### Home assistant setup

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-5"
  },
  agents: {
    defaults: {
      workspace: "~/clawd"
    }
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550100"],
      groups: {
        "*": { requireMention: true }
      }
    },
    telegram: {
      botToken: "your-token",
      allowFrom: ["@family"]
    }
  },
  gateway: {
    port: 18789,
    bind: "loopback",
    mode: "local"
  }
}
```

### Voice-first setup

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-5"
  },
  channels: {
    imessage: {
      enabled: true,
      groups: { "*": { requireMention: false } }
    }
  },
  gateway: {
    mode: "local"
  },
  voiceWake: {
    enabled: true,
    phrase: "Hey Clawd"
  }
}
```

## Troubleshooting

### Gateway won't start

```bash
# Check logs
log show --predicate 'subsystem == "com.clawdbot.gateway"' --last 10m

# Check config
clawdbot doctor

# Restart
clawdbot service restart
```

### Permission denied errors

```bash
# Reset TCC permissions (last resort)
tccutil reset All com.clawdbot.Clawdbot

# Then reopen the app and re-grant permissions
```

### High memory usage

- Consider using a smaller model
- Reduce session history
- Enable session pruning

## Maintenance

### Updates

```bash
# Update Clawdbot
npm update -g clawdbot

# Update macOS app
# Download latest release or rebuild
pnpm mac:package
```

### Backups

```bash
# Back up configuration and sessions
rsync -av ~/.clawdbot/ ~/Backup/clawdbot/

# Or use Time Machine (includes ~/.clawdbot)
```

## Next steps

- [Configure it](./configuration.md) — Customize your setup
- Learn about [privacy/security](./privacy-security.md) — Secure your Mac Mini
- See the [reference](./reference.md) — Common commands

For detailed macOS setup, see the [official macOS docs](https://docs.clawd.bot/platforms/macos).
