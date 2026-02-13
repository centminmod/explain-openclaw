# Deployment Scenarios

> **Last Updated:** January 2026  
> **Applies to:** Moltbot 2026.1.27-beta.1+

This guide covers three primary deployment scenarios for Moltbot, each suited to different use cases, security requirements, and technical expertise levels.

---

## Table of Contents

1. [Scenario 1: Standalone Mac Mini](#scenario-1-standalone-mac-mini)
2. [Scenario 2: Isolated VPS](#scenario-2-isolated-vps)
3. [Scenario 3: Cloudflare Moltworker](#scenario-3-cloudflare-moltworker)
4. [Comparison Matrix](#comparison-matrix)

---

## Scenario 1: Standalone Mac Mini

### Overview

Running Moltbot on a dedicated Mac Mini provides maximum privacy and control. The Mac acts as both the Gateway server and potentially a node for local tools.

### Use Cases

- Personal AI assistant for home/office
- Privacy-focused deployment (all data stays local)
- Integration with macOS-specific features (iMessage, Voice Wake)
- Development and testing environment

### Architecture

```
┌─────────────────────────────────────────────────┐
│              MAC MINI (Gateway Host)            │
│  ┌──────────────┐    ┌──────────────────────┐  │
│  │   Gateway    │    │   macOS Node         │  │
│  │   (Node.js)  │◄──►│   - system.run       │  │
│  │   Port 18789 │    │   - system.notify    │  │
│  └──────┬───────┘    │   - canvas           │  │
│         │            └──────────────────────┘  │
│         │                                       │
│    ┌────┴────┐                                  │
│    │Channels │◄── WhatsApp, Telegram, etc.     │
│    └─────────┘                                  │
└─────────────────────────────────────────────────┘
            │
            │ Tailscale/VPN (optional)
            ▼
┌─────────────────────────────────────────────────┐
│         REMOTE DEVICES (iOS/Android)            │
│         - Voice Wake                            │
│         - Camera/Location                       │
└─────────────────────────────────────────────────┘
```

### Setup Instructions

#### Step 1: Prepare the Mac

```bash
# Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Node.js 22
brew install node@22

# Verify installation
node --version  # Should show v22.x.x
```

#### Step 2: Install Moltbot

```bash
# Install Moltbot
curl -fsSL https://molt.bot/install.sh | bash

# Or via npm
npm install -g moltbot@latest
```

#### Step 3: Configure for Local-Only Operation

```json5
// ~/.moltbot/moltbot.json
{
  gateway: {
    mode: "local",
    bind: "loopback",     // Only localhost can connect
    port: 18789,
    auth: {
      mode: "token",
      token: "your-secure-random-token"  // Generate with: openssl rand -hex 32
    }
  },
  discovery: {
    mdns: {
      mode: "minimal"     // Don't expose sensitive info via mDNS
    }
  },
  agents: {
    defaults: {
      workspace: "~/clawd",
      sandbox: {
        mode: "non-main"  // Sandbox group/channel sessions
      }
    }
  }
}
```

#### Step 4: Install as Service

```bash
# Install launchd service
moltbot onboard --install-daemon

# Or manually create service
mkdir -p ~/Library/LaunchAgents
cat > ~/Library/LaunchAgents/com.moltbot.gateway.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.moltbot.gateway</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/moltbot</string>
        <string>gateway</string>
        <string>--bind</string>
        <string>loopback</string>
        <string>--port</string>
        <string>18789</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/moltbot-gateway.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/moltbot-gateway.error.log</string>
</dict>
</plist>
EOF

launchctl load ~/Library/LaunchAgents/com.moltbot.gateway.plist
```

#### Step 5: Optional - macOS App

For GUI control and Voice Wake:

```bash
# Build macOS app (requires Xcode)
cd /path/to/moltbot
pnpm mac:package

# Or download pre-built from releases
open dist/Moltbot.app
```

### Security Hardening

```bash
# 1. Enable Firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on

# 2. Restrict file permissions
chmod 700 ~/.moltbot
chmod 600 ~/.moltbot/moltbot.json
chmod -R 600 ~/.moltbot/credentials/*

# 3. Enable FileVault (full disk encryption)
sudo fdesetup enable

# 4. Disable unnecessary services
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.apsd.plist 2>/dev/null || true

# 5. Run security audit
moltbot security audit --deep --fix
```

### Pros & Cons

| Pros | Cons |
|------|------|
| ✅ Complete data privacy | ❌ Requires always-on Mac |
| ✅ Easy macOS integration | ❌ Limited to local network (without VPN) |
| ✅ iMessage support | ❌ Single point of failure |
| ✅ Voice Wake on macOS | ❌ Higher power consumption than VPS |
| ✅ No recurring cloud costs | ❌ Hardware maintenance |

---

## Scenario 2: Isolated VPS

### Overview

Deploying Moltbot on a Virtual Private Server (VPS) provides 24/7 availability and network accessibility without keeping personal devices powered on.

### Use Cases

- 24/7 AI assistant
- Multiple user support
- Integration with cloud services
- Geographic distribution (choose server location)
- Separation from personal devices

### Recommended VPS Providers

| Provider | Minimum Spec | Cost/Month | Notes |
|----------|--------------|------------|-------|
| **Hetzner** | 2 vCPU, 4GB RAM | ~€5 | Great value, EU/Germany |
| **DigitalOcean** | 1 vCPU, 1GB RAM | ~$6 | Easy to use, good docs |
| **Linode** | 1 vCPU, 1GB RAM | ~$5 | Reliable, good support |
| **AWS Lightsail** | 1 vCPU, 1GB RAM | ~$5 | AWS ecosystem |
| **Vultr** | 1 vCPU, 1GB RAM | ~$5 | Fast provisioning |

### Architecture

```
┌─────────────────────────────────────────────────┐
│              VPS (Cloud Provider)               │
│  ┌─────────────────────────────────────────┐   │
│  │           Gateway (Docker)              │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐ │   │
│  │  │Channels │  │ Sessions│  │  Tools  │ │   │
│  │  │         │  │         │  │         │ │   │
│  │  │WhatsApp │  │ Routing │  │ bash    │ │   │
│  │  │Telegram │  │ Storage │  │ browser │ │   │
│  │  │Discord  │  │         │  │ cron    │ │   │
│  │  └────┬────┘  └────┬────┘  └────┬────┘ │   │
│  │       └─────────────┴─────────────┘    │   │
│  │              Port 18789                │   │
│  └─────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────┘
                         │
           ┌─────────────┼─────────────┐
           │             │             │
           ▼             ▼             ▼
      ┌─────────┐  ┌─────────┐  ┌─────────┐
      │   You   │  │ Family  │  │  Tail   │
      │ (Phone) │  │(Devices)│  │  scale  │
      └─────────┘  └─────────┘  └─────────┘
```

### Setup Instructions

#### Step 1: Provision VPS

**Hetzner Example:**

```bash
# Create server (via web UI or hcloud CLI)
hcloud server create --name moltbot-gateway \
  --type cx21 \
  --image ubuntu-22.04 \
  --location nbg1

# Get IP
hcloud server ip moltbot-gateway
```

#### Step 2: Initial Server Setup

```bash
# SSH to server
ssh root@your-server-ip

# Create non-root user
adduser moltbot
usermod -aG sudo moltbot

# Configure firewall
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp    # SSH
ufw allow 18789/tcp # Moltbot (if exposing directly)
ufw enable

# Install Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt-get install -y nodejs

# Install pnpm
npm install -g pnpm

# Install Docker (optional but recommended)
curl -fsSL https://get.docker.com | sh
usermod -aG docker moltbot
```

#### Step 3: Install Moltbot

```bash
# Switch to moltbot user
su - moltbot

# Install Moltbot
npm install -g moltbot@latest

# Create state directory
mkdir -p ~/.moltbot
```

#### Step 4: Configure for Remote Access

```json5
// ~/.moltbot/moltbot.json
{
  gateway: {
    mode: "local",
    bind: "loopback",     // Still bind to loopback for security
    port: 18789,
    auth: {
      mode: "password",   // Use password for remote
      password: "${CLAWDBOT_GATEWAY_PASSWORD}"  // Set via env var
    }
  },
  // Tailscale recommended for remote access
  tailscale: {
    enabled: true,
    mode: "serve"  // or "funnel" for public HTTPS
  },
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "agent"
      }
    }
  }
}
```

#### Step 5: Docker Deployment (Recommended)

```bash
# Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  moltbot:
    image: moltbot/moltbot:latest
    container_name: moltbot-gateway
    restart: unless-stopped
    
    # Security: Run as non-root
    user: "1000:1000"
    
    # Network: Isolated bridge
    networks:
      - moltbot-network
    
    # Ports: Only expose Gateway
    ports:
      - "127.0.0.1:18789:18789"
    
    # Volumes: Persistent state
    volumes:
      - ./data:/app/data
      - /etc/localtime:/etc/localtime:ro
    
    # Environment
    environment:
      - NODE_ENV=production
      - CLAWDBOT_STATE_DIR=/app/data
      - CLAWDBOT_GATEWAY_PASSWORD=${CLAWDBOT_GATEWAY_PASSWORD}
    
    # Security options
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - SETUID
      - SETGID
    read_only: true
    tmpfs:
      - /tmp:noexec,nosuid,size=100m

networks:
  moltbot-network:
    driver: bridge
EOF

# Create data directory
mkdir -p data
chmod 700 data

# Set environment
echo "CLAWDBOT_GATEWAY_PASSWORD=$(openssl rand -hex 32)" > .env
chmod 600 .env

# Start
docker-compose up -d
```

#### Step 6: systemd Service (Non-Docker)

```bash
# Create systemd service
sudo tee /etc/systemd/system/moltbot-gateway.service > /dev/null << 'EOF'
[Unit]
Description=Moltbot Gateway
After=network.target

[Service]
Type=simple
User=moltbot
Group=moltbot
WorkingDirectory=/home/moltbot
Environment="NODE_ENV=production"
Environment="CLAWDBOT_STATE_DIR=/home/moltbot/.moltbot"
Environment="CLAWDBOT_GATEWAY_PASSWORD_FILE=/home/moltbot/.moltbot/gateway-password"
ExecStart=/usr/bin/moltbot gateway --bind loopback --port 18789
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

# Security
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/home/moltbot/.moltbot
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true
RestrictSUIDSGID=true
RestrictRealtime=true
RestrictNamespaces=true
LockPersonality=true
MemoryDenyWriteExecute=true
SystemCallFilter=@system-service
SystemCallErrorNumber=EPERM

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable moltbot-gateway
sudo systemctl start moltbot-gateway
```

#### Step 7: Remote Access via Tailscale

```bash
# Install Tailscale on VPS
curl -fsSL https://tailscale.com/install.sh | sh

# Authenticate
sudo tailscale up

# Get Tailscale IP
tailscale ip -4

# On your local machine, connect via Tailscale
moltbot config set gateway.remote.host "100.x.x.x"  # VPS Tailscale IP
moltbot config set gateway.remote.port 18789
moltbot config set gateway.remote.token "your-gateway-token"
```

### Security Hardening

```bash
# 1. Keep system updated
sudo apt update && sudo apt upgrade -y

# 2. Configure fail2ban
sudo apt install fail2ban
sudo tee /etc/fail2ban/jail.local << 'EOF'
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
EOF

# 3. Disable root SSH
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# 4. Enable automatic security updates
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# 5. Setup log monitoring
sudo apt install logwatch
```

### Pros & Cons

| Pros | Cons |
|------|------|
| ✅ 24/7 availability | ❌ Requires trusting VPS provider |
| ✅ Accessible from anywhere | ❌ Recurring hosting costs |
| ✅ Better for multiple users | ❌ Network latency |
| ✅ Isolated from personal devices | ❌ More complex setup |
| ✅ Professional IP reputation | ❌ No iMessage support |

---

## Scenario 3: Cloudflare Moltworker

See dedicated document: [Cloudflare Moltworker](./cloudflare-moltworker.md)

---

## Comparison Matrix

| Feature | Mac Mini | VPS | Cloudflare Moltworker |
|---------|----------|-----|----------------------|
| **Privacy** | ⭐⭐⭐ Excellent | ⭐⭐ Good | ⭐⭐ Good |
| **Uptime** | ⭐⭐ Moderate | ⭐⭐⭐ Excellent | ⭐⭐⭐ Excellent |
| **Setup Difficulty** | ⭐⭐ Easy | ⭐⭐⭐ Moderate | ⭐⭐⭐ Moderate |
| **Cost** | Hardware only | $5-20/month | Free tier available |
| **iMessage** | ✅ Yes | ❌ No | ❌ No |
| **Voice Wake** | ✅ Yes | ❌ No | ❌ No |
| **24/7 Operation** | ❌ No* | ✅ Yes | ✅ Yes |
| **Global Edge** | ❌ No | ❌ No | ✅ Yes |
| **Shell Access** | ✅ Full | ✅ Full | ⭐ Limited |

*Unless you keep Mac always on

### Decision Guide

**Choose Mac Mini if:**
- Privacy is your top priority
- You want iMessage support
- You're the only user
- You already have a Mac to use

**Choose VPS if:**
- You need 24/7 availability
- Multiple people will use it
- You want geographic flexibility
- You're comfortable with server admin

**Choose Cloudflare Moltworker if:**
- You want zero server management
- You need global edge deployment
- Cost minimization is important
- You're comfortable with serverless limitations

---

## Related Documentation

- [Installation & Setup](./installation-and-setup.md)
- [Cloudflare Moltworker](./cloudflare-moltworker.md)
- [Security Analysis](./security-analysis.md)
- [Configuration Reference](./configuration-reference.md)