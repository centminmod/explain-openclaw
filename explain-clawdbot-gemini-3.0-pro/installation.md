# Installation & Setup Guide

This guide covers two primary use cases: a standalone **Mac Mini** (for a rich, local ecosystem) and an isolated **Linux VPS** (for a secure, always-on cloud presence).

## Prerequisites
*   **Node.js**: Version 22 or higher.
*   **Package Manager**: `npm`, `pnpm`, or `bun`.
*   **Docker**: Required for sandboxing (highly recommended for VPS).

---

## Use Case 1: The "Home Base" (Standalone Mac Mini)

This setup maximizes functionality, leveraging macOS-specific features like iMessage, Voice Wake, and local Canvas.

### 1. Installation
```bash
# Install globally
npm install -g clawdbot@latest

# Run the onboarding wizard
clawdbot onboard --install-daemon
```

### 2. Configuration for Mac
The wizard will create `~/.clawdbot/clawdbot.json`. We'll tune it for a Mac home server.

**Key Features to Enable:**
*   **iMessage**: Allows you to text your bot from your iPhone without any third-party apps.
*   **Voice Wake**: Enables "Hey Clawd" functionality if you have a microphone connected.

**`~/.clawdbot/clawdbot.json` snippet:**
```json
{
  "channels": {
    "imessage": {
      "enabled": true,
      "allowFrom": ["+15551234567"] // Your phone number
    }
  },
  "nodes": {
    "mac": {
      "voiceWake": true,
      "canvas": true
    }
  }
}
```

### 3. Connect Devices
1.  Run `clawdbot gateway` to start the server.
2.  Install the **Clawdbot Companion App** on your Mac (if available) or use the web dashboard at `http://localhost:18789`.
3.  **iMessage**: Ensure the Mac's Messages app is signed in. The bot will now respond to texts from your allowlisted number.

---

## Use Case 2: The "Fortress" (Isolated VPS)

This setup prioritizes security and isolation. It's perfect for a bot that serves a group chat or runs 24/7 without needing your laptop open.

### 1. Security First
Create a non-root user for Clawdbot.
```bash
useradd -m -s /bin/bash clawd
usermod -aG docker clawd
su - clawd
```

### 2. Installation
```bash
# Install Clawdbot
npm install -g clawdbot@latest

# Initialize
clawdbot onboard
```

### 3. Secure Remote Access (Tailscale)
Instead of opening port `18789` to the wild internet, use Tailscale.

1.  Install Tailscale on your VPS.
2.  Configure Clawdbot to use Tailscale Funnel or Serve.

**`~/.clawdbot/clawdbot.json` snippet:**
```json
{
  "gateway": {
    "bind": "loopback", // Only listen on localhost
    "tailscale": {
      "mode": "funnel", // Public HTTPS URL handled by Tailscale
      "auth": "password" // Require password for the dashboard
    }
  },
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all" // Force ALL sessions (even yours) into Docker
      }
    }
  }
}
```

### 4. Running Headless
Use the systemd service generator included in the CLI (via `onboard --install-daemon`) or use `pm2`.

```bash
# Using PM2
npm install -g pm2
pm2 start clawdbot -- gateway
pm2 save
```

### 5. Docker Sandboxing
Ensure Docker is running. When the bot needs to execute code (e.g., "calculate the 100th fibonacci number"), it will spin up a transient Docker container, execute the code, and destroy the container. This keeps your VPS filesystem clean and safe.
