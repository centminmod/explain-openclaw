# VPS Deployment

This guide covers deploying Clawdbot on an isolated VPS (Virtual Private Server) for 24/7 availability with strong security.

## Use Case

A cloud server running Clawdbot:
- 24/7 availability without home hardware
- Accessible from anywhere
- Isolated from personal devices
- Professional-grade uptime

---

## VPS Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| RAM | 1GB | 2GB+ |
| CPU | 1 vCPU | 2 vCPU |
| Storage | 20GB | 40GB+ |
| OS | Ubuntu 22.04 / Debian 12 | Ubuntu 24.04 |
| Network | IPv4 | IPv4 + IPv6 |

**Recommended Providers:**
- Hetzner (EU, affordable)
- DigitalOcean
- Linode
- Vultr
- AWS Lightsail

---

## Initial Server Setup

### 1. Create Dedicated User

```bash
# SSH as root initially
ssh root@your-vps-ip

# Create clawdbot user
adduser clawdbot
usermod -aG sudo clawdbot

# Switch to new user
su - clawdbot
```

### 2. Install Node.js 22

```bash
# Install NodeSource repo
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -

# Install Node.js
sudo apt install -y nodejs

# Verify
node --version  # Should be 22.x
```

### 3. Install Clawdbot

```bash
sudo npm install -g clawdbot@latest
clawdbot --version
```

---

## Security Hardening

### Firewall (UFW)

```bash
# Install UFW
sudo apt install -y ufw

# Default deny incoming
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (change port if using non-standard)
sudo ufw allow 22/tcp

# If using Tailscale, allow its interface
# sudo ufw allow in on tailscale0

# Enable firewall
sudo ufw enable
sudo ufw status
```

**Note:** We do NOT expose port 18789 publicly. Access via Tailscale or SSH tunnel only.

### SSH Hardening

Edit `/etc/ssh/sshd_config`:

```bash
# Disable root login
PermitRootLogin no

# Disable password auth (use keys only)
PasswordAuthentication no

# Change default port (optional)
Port 2222
```

Restart SSH:
```bash
sudo systemctl restart sshd
```

### Fail2Ban

```bash
sudo apt install -y fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

---

## Install Tailscale (Recommended)

Zero-trust networking for secure access:

```bash
# Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# Connect
sudo tailscale up

# Get Tailscale IP
tailscale ip
```

Now you can access the VPS via its Tailscale IP from any device on your Tailscale network.

---

## Configuration

### Create Config Directory

```bash
mkdir -p ~/.clawdbot
chmod 700 ~/.clawdbot
```

### Create Config File

Create `~/.clawdbot/clawdbot.json`:

```json5
{
  "agent": {
    "model": "claude-opus-4-5",
    "provider": "anthropic",
    "thinking": "medium"
  },

  "gateway": {
    "port": 18789,
    "bind": "loopback",  // IMPORTANT: localhost only
    "mode": "local",
    "auth": {
      "mode": "tailscale"  // Use Tailscale identity
    }
  },

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
  },

  "logging": {
    "level": "info"
  }
}
```

### Set Permissions

```bash
chmod 600 ~/.clawdbot/clawdbot.json
```

---

## Environment Variables

Create `~/.clawdbot-env`:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export TELEGRAM_BOT_TOKEN="123456789:ABC..."
export CLAWDBOT_GATEWAY_TOKEN="$(openssl rand -hex 32)"
```

```bash
chmod 600 ~/.clawdbot-env
```

Add to `~/.bashrc`:
```bash
source ~/.clawdbot-env
```

---

## Systemd Service

Create `/etc/systemd/system/clawdbot.service`:

```ini
[Unit]
Description=Clawdbot Gateway
After=network.target

[Service]
Type=simple
User=clawdbot
Group=clawdbot
WorkingDirectory=/home/clawdbot
EnvironmentFile=/home/clawdbot/.clawdbot-env
ExecStart=/usr/bin/clawdbot gateway run --port 18789
Restart=always
RestartSec=10

# Security hardening
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=read-only
ReadWritePaths=/home/clawdbot/.clawdbot
PrivateTmp=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable clawdbot
sudo systemctl start clawdbot
```

Check status:

```bash
sudo systemctl status clawdbot
```

---

## Access Methods

### Via Tailscale (Recommended)

From any device on your Tailscale network:

```bash
# Access gateway
curl http://100.x.x.x:18789/health

# SSH
ssh clawdbot@100.x.x.x
```

### Via SSH Tunnel

If not using Tailscale:

```bash
# From your local machine
ssh -L 18789:localhost:18789 clawdbot@your-vps-ip

# Now access localhost:18789 locally
```

---

## Monitoring

### View Logs

```bash
# Systemd logs
sudo journalctl -u clawdbot -f

# Or if logging to file
tail -f ~/.clawdbot/logs/clawdbot.log
```

### Health Checks

```bash
# Check service
sudo systemctl status clawdbot

# Check channels
clawdbot channels status --probe

# Check gateway
curl http://localhost:18789/health
```

### Uptime Monitoring

Set up external monitoring with:
- UptimeRobot (free tier)
- Healthchecks.io
- Custom script with alerts

---

## Automatic Updates

Create `/home/clawdbot/update-clawdbot.sh`:

```bash
#!/bin/bash
set -e

echo "Updating Clawdbot..."
sudo npm update -g clawdbot

echo "Restarting service..."
sudo systemctl restart clawdbot

echo "Checking status..."
sleep 5
sudo systemctl status clawdbot --no-pager
```

```bash
chmod +x ~/update-clawdbot.sh
```

Run manually or schedule via cron.

---

## Backup Strategy

### What to Backup

- `~/.clawdbot/clawdbot.json` - Configuration
- `~/.clawdbot/credentials/` - API credentials
- `~/.clawdbot/agents/` - Conversation history
- `~/.clawdbot-env` - Environment variables

### Automated Backup Script

Create `/home/clawdbot/backup-clawdbot.sh`:

```bash
#!/bin/bash
BACKUP_DIR="/home/clawdbot/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/clawdbot_$DATE.tar.gz"

mkdir -p "$BACKUP_DIR"

tar -czf "$BACKUP_FILE" \
    ~/.clawdbot/clawdbot.json \
    ~/.clawdbot/credentials/ \
    ~/.clawdbot-env

# Keep only last 7 backups
ls -t "$BACKUP_DIR"/clawdbot_*.tar.gz | tail -n +8 | xargs -r rm

echo "Backup created: $BACKUP_FILE"
```

Add to crontab:
```bash
crontab -e
# Add:
0 2 * * * /home/clawdbot/backup-clawdbot.sh
```

---

## Resource Limits

Prevent runaway processes:

Edit `/etc/systemd/system/clawdbot.service`, add to `[Service]`:

```ini
# Memory limit
MemoryMax=1G

# CPU limit (50% of one core)
CPUQuota=50%

# File descriptor limit
LimitNOFILE=4096
```

Reload:
```bash
sudo systemctl daemon-reload
sudo systemctl restart clawdbot
```

---

## Troubleshooting

### "Service won't start"

```bash
# Check logs
sudo journalctl -u clawdbot -n 50

# Check permissions
ls -la ~/.clawdbot/

# Verify config
clawdbot doctor
```

### "Can't connect from Tailscale"

```bash
# Verify Tailscale is running
tailscale status

# Check firewall allows Tailscale
sudo ufw status

# Test local connection first
curl http://localhost:18789/health
```

### "Out of memory"

```bash
# Check memory usage
free -h

# Consider adding swap
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent: add to /etc/fstab
/swapfile none swap sw 0 0
```

### "Channels not connecting"

```bash
# Check credentials
cat ~/.clawdbot-env | grep TOKEN

# Verify with probe
clawdbot channels status --probe

# Check network
curl -I https://api.telegram.org
```

---

## Complete Setup Checklist

- [ ] VPS created with dedicated user
- [ ] Node.js 22+ installed
- [ ] Clawdbot installed
- [ ] Firewall configured (deny all except SSH)
- [ ] SSH hardened (keys only, no root)
- [ ] Tailscale installed
- [ ] Config file created with proper permissions
- [ ] Environment variables set
- [ ] Systemd service enabled
- [ ] Monitoring configured
- [ ] Backup script scheduled
- [ ] Resource limits set

---

## Security Summary

| Layer | Protection |
|-------|------------|
| Network | Firewall blocks all incoming except SSH |
| Access | Tailscale for zero-trust networking |
| Auth | Tailscale identity or token |
| Process | Runs as unprivileged user |
| Filesystem | Read-only except state dir |
| Data | Allowlists restrict who can interact |

---

*Continue to [10 - Commands Reference](./10-commands-reference.md) →*
