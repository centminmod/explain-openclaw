# Using Clawdbot on an Isolated VPS

Running Clawdbot on a VPS (Virtual Private Server) gives you an always-on AI assistant accessible from anywhere. This guide explains how to set it up securely.

## Why use a VPS?

### Advantages

| Benefit | Why it matters |
|---------|----------------|
| **Accessible anywhere** | Connect from home, work, or travel |
| **No local hardware** | Nothing to buy or maintain at home |
| **Isolated** | Separate from your personal devices |
| **Scalable** | Upgrade resources as needed |
| **Low cost** | $5-20/month for basic plans |
| **Reliable** | Professional data center infrastructure |

### Trade-offs vs Mac Mini

| VPS | Mac Mini |
|-----|----------|
| Lower upfront cost | Higher upfront cost |
| Monthly fee | One-time purchase |
| Less control over hardware | Full hardware control |
| Requires Linux knowledge | macOS is familiar |
| No native apps | Has macOS app |
| Good for remote access | Good for home use |

## VPS requirements

### Minimum specs

- **CPU:** 2 vCPUs
- **RAM:** 2GB (4GB recommended)
- **Storage:** 20GB SSD
- **Bandwidth:** 1TB/month (generous for text)
- **OS:** Ubuntu 22.04/24.04 or Debian 12

### Recommended providers

| Provider | Starting price | Notes |
|----------|----------------|-------|
| **Hetzner** | ~€4/month | Germany, great value |
| **DigitalOcean** | $6/month | Easy to use, good docs |
| **Linode** | $5/month | Reliable, global locations |
| **Vultr** | $6/month | Many locations |
| **AWS Lightsail** | $5/month | AWS ecosystem |

For high privacy, consider providers outside the "Fourteen Eyes" if that concerns you.

## Initial server setup

### 1. Create and SSH into your VPS

```bash
# SSH into your new server
ssh root@your-vps-ip

# Or if using a key:
ssh -i ~/.ssh/your-key root@your-vps-ip
```

### 2. Create a non-root user

```bash
# Create user
adduser clawdbot

# Add to sudo group
usermod -aG sudo clawdbot

# Switch to user
su - clawdbot
```

### 3. Secure SSH

```bash
# As clawdbot user, create SSH key
mkdir -p ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t ed25519 -C "clawdbot@vps"

# Copy public key to authorized_keys
cat ~/.ssh/id_ed25519.pub > ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Exit and copy the private key to your local machine
exit
```

From your local machine:

```bash
# Copy the key from server
scp root@your-vps-ip:/home/clawdbot/.ssh/id_ed25519 ~/.ssh/clawdbot-vps

# Add to SSH config
cat >> ~/.ssh/config << 'EOF'
Host clawdbot-vps
    HostName your-vps-ip
    User clawdbot
    IdentityFile ~/.ssh/clawdbot-vps
EOF

# Test SSH
ssh clawdbot-vps
```

### 4. Disable root login and password auth

On the server:

```bash
sudo nano /etc/ssh/sshd_config
```

Change these settings:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart SSH:

```bash
sudo systemctl restart sshd
```

## Installing Clawdbot

### 1. Install Node.js

```bash
# As clawdbot user, install Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify
node --version
npm --version
```

### 2. Install Clawdbot

```bash
# Install globally
sudo npm install -g clawdbot@latest

# Verify
clawdbot --version
```

### 3. Run the onboarding wizard

```bash
clawdbot onboard
```

The wizard will guide you through:
- AI provider selection
- API key configuration
- Channel setup
- Gateway configuration

## Running as a service

### Create systemd service

```bash
sudo nano /etc/systemd/system/clawdbot.service
```

Add this content:

```ini
[Unit]
Description=Clawdbot Gateway
After=network.target

[Service]
Type=simple
User=clawdbot
WorkingDirectory=/home/clawdbot
Environment="PATH=/home/clawdbot/.local/bin:/usr/bin"
ExecStart=/usr/bin/clawdbot gateway --port 18789
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable clawdbot
sudo systemctl start clawdbot

# Check status
sudo systemctl status clawdbot

# View logs
sudo journalctl -u clawdbot -f
```

## Remote access setup

### Option 1: SSH tunnel (simple)

From your local machine:

```bash
# Forward the gateway port
ssh -N -L 18789:127.0.0.1:18789 clawdbot-vps

# Now ws://127.0.0.1:18789 on your machine reaches the VPS
```

### Option 2: Tailscale (recommended)

#### Install Tailscale on VPS

```bash
# Add Tailscale's GPG key
curl -fsSL https://tailscale.com/install.sh | sh

# Connect to your tailnet
sudo tailscale up

# Get your Tailscale IP
tailscale ip -4
```

#### Install Tailscale on your local machine

Download from [tailscale.com](https://tailscale.com) and log in.

#### Enable Tailscale Serve (optional)

On the VPS:

```bash
# Enable HTTPS via Tailscale
sudo tailscale serve --https=443 --bg

# Or enable Funnel for public access (with password)
sudo tailscale funnel --bg
```

Enable in Clawdbot config:

```json5
{
  gateway: {
    tailscale: {
      mode: "serve",  // or "funnel" for public
      resetOnExit: false
    }
  }
}
```

## Security hardening

### 1. Configure firewall

```bash
# Install ufw if not present
sudo apt install -y ufw

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (only after key auth is set up)
sudo ufw allow 22/tcp

# Allow Tailscale
sudo ufw allow 41694/udp

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status
```

### 2. Configure fail2ban

```bash
# Install fail2ban
sudo apt install -y fail2ban

# Enable SSH protection
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### 3. Gateway authentication

Require password for remote access:

```json5
{
  gateway: {
    port: 18789,
    bind: "loopback",
    mode: "local",
    auth: {
      mode: "password",
      password: "your-secure-random-password"
    },
    tailscale: {
      mode: "serve",
      resetOnExit: false
    }
  }
}
```

### 4. Channel security

```json5
{
  channels: {
    telegram: {
      botToken: "your-token",
      allowFrom: ["@your-username"],
      dm: { policy: "pairing" }
    },
    discord: {
      token: "your-token",
      dm: { policy: "pairing" }
    }
  },
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session"
      }
    }
  }
}
```

## Docker setup (alternative)

For better isolation, run in Docker:

### 1. Install Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker clawdbot
```

### 2. Run Clawdbot in Docker

```bash
docker run -d \
  --name clawdbot \
  --restart unless-stopped \
  -p 127.0.0.1:18789:18789 \
  -v ~/.clawdbot:/home/clawdbot/.clawdbot \
  -e ANTHROPIC_API_KEY=your-key \
  ghcr.io/clawdbot/clawdbot:latest \
  gateway --port 18789
```

### 3. Docker Compose (recommended)

Create `docker-compose.yml`:

```yaml
version: "3.8"
services:
  clawdbot:
    image: ghcr.io/clawdbot/clawdbot:latest
    container_name: clawdbot
    restart: unless-stopped
    ports:
      - "127.0.0.1:18789:18789"
    volumes:
      - ./clawdbot-data:/home/clawdbot/.clawdbot
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - TELEGRAM_BOT_TOKEN=${TELEGRAM_BOT_TOKEN}
```

Run with:

```bash
docker-compose up -d
```

## Backup and restore

### Backup

```bash
# Create backup script
cat > ~/backup-clawdbot.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d)
BACKUP_DIR=/home/clawdbot/backups
mkdir -p $BACKUP_DIR
tar -czf $BACKUP_DIR/clawdbot-$DATE.tar.gz ~/.clawdbot
# Keep last 7 days
find $BACKUP_DIR -name "clawdbot-*.tar.gz" -mtime +7 -delete
EOF

chmod +x ~/backup-clawdbot.sh

# Add to crontab (daily at 2am)
crontab -e
# Add: 0 2 * * * /home/clawdbot/backup-clawdbot.sh
```

### Restore

```bash
# Stop service
sudo systemctl stop clawdbot

# Restore from backup
tar -xzf ~/backups/clawdbot-20250127.tar.gz -C ~/

# Restart service
sudo systemctl start clawdbot
```

## Monitoring

### Basic health check

```bash
# Check service status
sudo systemctl status clawdbot

# Check if port is listening
ss -ltnp | grep 18789

# Check logs
sudo journalctl -u clawdbot -n 50 -f
```

### Resource monitoring

```bash
# Install htop
sudo apt install -y htop

# View resource usage
htop
```

### Set up alerts (optional)

Use a service like UptimeRobot or healthchecks.io to monitor the Gateway health endpoint.

## Example configurations

### High-security VPS setup

```json5
{
  gateway: {
    port: 18789,
    bind: "loopback",
    mode: "local",
    auth: {
      mode: "password",
      password: "use-env-var-or-keepass"
    },
    tailscale: {
      mode: "serve"
    }
  },
  agent: {
    model: "anthropic/claude-opus-4-5"
  },
  agents: {
    defaults: {
      sandbox: {
        mode: "all",
        scope: "session"
      }
    }
  },
  channels: {
    telegram: {
      botToken: "your-token",
      allowFrom: ["@your-username"],
      dm: { policy: "pairing" },
      groups: {
        "*": { requireMention: true }
      }
    }
  }
}
```

### Multi-region setup

Run Gateway in multiple regions for redundancy:

1. Set up two VPS in different regions
2. Configure both identically
3. Use DNS failover or load balancer
4. Sessions can be moved between regions

## Troubleshooting

### Gateway won't start

```bash
# Check logs
sudo journalctl -u clawdbot -n 100

# Validate config
clawdbot doctor

# Check for port conflicts
ss -ltnp | grep 18789
```

### Can't connect remotely

```bash
# Verify service is running
sudo systemctl status clawdbot

# Check firewall
sudo ufw status

# Test SSH tunnel from local machine
ssh -N -v -L 18789:127.0.0.1:18789 clawdbot-vps
```

### Out of memory

```bash
# Check memory usage
free -h

# Add swap if needed
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## Comparison: Mac Mini vs VPS

| Aspect | Mac Mini | VPS |
|--------|----------|-----|
| **Privacy** | Data stays in your home | Data in data center |
| **Control** | Full hardware control | Limited by provider |
| **Accessibility** | Home network only | Anywhere |
| **Cost** | $500-800 upfront | $5-20/month |
| **Maintenance** | You handle updates | Provider handles infrastructure |
| **Apps** | Native macOS app | CLI/Web only |
| **Reliability** | Depends on your power/internet | Professional SLA |

## Next steps

- [Configure it](./configuration.md) — Customize your VPS setup
- Learn about [privacy/security](./privacy-security.md) — Secure your server
- See the [reference](./reference.md) — Common commands and troubleshooting

For detailed Docker setup, see the [official Docker docs](https://docs.clawd.bot/install/docker).
