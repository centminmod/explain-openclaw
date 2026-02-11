# Deployment: Isolated VPS (remote + locked down)

> **Note:** This guide is for OpenClaw (formerly Moltbot/Clawdbot).

## Skill level key

| Tag | Meaning |
| ----- | --------- |
| `[All]` | Everyone should do this |
| `[Intermediate]` | Recommended for users comfortable with Linux CLI |
| `[Advanced]` | Optional hardening for security-focused deployments |

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
  - [Isolated VPS](./vps-isolated.md)

---

Goal: run Gateway on a Linux VPS while keeping access **private** and host hardened.

This is a great setup when:
- you want assistant always-on
- your laptop sleeps often
- you want predictable networking

But security bar is higher: a VPS is an internet-connected machine.

> **Recommended path:** For most users, follow sections 1-6 (baseline setup), then configure **Tailscale** for secure remote access. Sections 7-13 are for advanced users who need public-facing access or additional hardening.

Related official docs:
- https://docs.openclaw.ai/gateway/remote
- https://docs.openclaw.ai/gateway/security
- https://docs.openclaw.ai/platforms/linux

---

## Recommended posture (summary)

- Keep Gateway **loopback-only** (`gateway.bind: "loopback"`)
- Access via **Tailscale** (recommended) or SSH tunnel (fallback)
- Use token/password auth — or Tailscale identity-based auth
- Run as a dedicated non-root user
- Lock down file permissions
- Disable mDNS discovery on VPS
- Run `openclaw security audit --deep` after setup and after any config change

---

## 1) Provision VPS (baseline hardening) `[All]`

Based on security best practices.

> **⚠️ Provider Isolation:** Use a **separate VPS provider** (or at minimum a separate account) from any provider you already use for production services. If OpenClaw triggers TOS violations — outbound spam, misconfigured gateway relaying abuse, or API abuse — your provider may **suspend your entire account**, not just the OpenClaw VPS. That means all your other instances, snapshots, backups, and DNS records on that account go down too.

### Choose a Provider

- AWS EC2: t3.small, Ubuntu 24.04 LTS
- DigitalOcean: Basic $6/month Droplet, Ubuntu 24.04
- Linode: Nanode 1GB, Ubuntu 24.04
- Hetzner: CX11, Ubuntu 24.04

### SSH Key Setup (if you don't have one)

Generate one on your **local machine** (not server):

```bash
# Generate an Ed25519 key (recommended — stronger than RSA, shorter keys)
ssh-keygen -t ed25519 -C "your-email@example.com"

# Or RSA 4096-bit if Ed25519 is not supported
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

When prompted for a passphrase, **set one** — it protects your key if your laptop is compromised.

Most cloud providers let you upload your public key (`~/.ssh/id_ed25519.pub`) during VPS creation.

If your provider doesn't, copy it after first login:

```bash
# Copies your public key to server so you can log in without a password
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@YOUR_SERVER_IP
```

### Initial Setup

```bash
# Log in to your VPS using your SSH key
ssh -i ~/.ssh/id_ed25519 ubuntu@YOUR_SERVER_IP

# Download and install all available updates
sudo apt update && sudo apt upgrade -y

# Create a dedicated user account for running OpenClaw (not root)
sudo adduser openclaw
# Give that user permission to run admin commands with "sudo"
sudo usermod -aG sudo openclaw
```

### Firewall Configuration (Critical)

Only allow SSH from your IP. **Never allow `0.0.0.0/0` (anywhere) access.

```bash
# Block ALL incoming connections by default (nothing gets in unless you allow it)
sudo ufw default deny incoming
# Allow all outgoing connections (your server can still reach the internet)
sudo ufw default allow outgoing
# Punch a hole for SSH (port 22) so you don't lock yourself out
sudo ufw allow ssh
# Turn on firewall — defaults must be set FIRST
sudo ufw enable

# Show current firewall rules to confirm everything looks right
sudo ufw status
```

**Cloud firewall:** Also configure your provider's firewall (Security Groups on AWS, etc.) to only allow SSH from your IP. Do **not** open port 18789 to the public internet.

### SSH Hardening (Critical)

Disable password authentication and root login to prevent brute-force attacks:

```bash
# Open SSH server config file in a text editor
sudo nano /etc/ssh/sshd_config
```

Find and set (or uncomment) these lines:

```
PermitRootLogin no              # Nobody can SSH in as "root" directly
PasswordAuthentication no       # Only key-based login allowed (no password guessing)
PubkeyAuthentication yes        # Enable SSH key authentication
```

**Before reloading**, validate config to avoid locking yourself out:

```bash
# Dry-run SSH config — catches typos before they take effect
sudo sshd -t
```

If test passes with no output, reload SSH to apply changes:

```bash
# Tell SSH service to re-read its config (existing connections stay open)
sudo systemctl reload ssh
```

**Lockout prevention:** Open a **second terminal** and test SSH login before closing your current session. If new connection works, you're safe. If it fails, fix `sshd_config` in the still-open session.

> **Beginner tip:** If you do get locked out, use your provider's web console (DigitalOcean "Access" tab, AWS "EC2 Instance Connect", etc.) to fix `sshd_config`.

### Swap File Setup

Small VPS instances (1-2 GB RAM) can run out of memory during agent sessions with GLM-5.0's larger context, causing process to be killed (OOM). A swap file acts as emergency overflow:

```bash
# Allocate a 2GB file on disk to use as swap (virtual memory)
sudo fallocate -l 2G /swapfile
# Only root should read/write swap file (security best practice)
sudo chmod 600 /swapfile
# Format as swap space
sudo mkswap /swapfile
# Activate swap immediately
sudo swapon /swapfile

# Add it to /etc/fstab so it activates automatically after a reboot
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Confirm swap is active — you should see "Swap: 2.0G" in output
free -h
```

---

## 2) Install OpenClaw `[All]`

On VPS:

```bash
# Download and run official OpenClaw installer
curl -fsSL https://openclaw.ai/install.sh | bash
```

Verify Node.js version (22.12.0+ recommended for security patches):

```bash
node --version  # Should be v22.12.0 or later
```

Then onboard — this walks you through initial setup and creates a background service:

```bash
# Interactive setup wizard + installs a systemd service
openclaw onboard --install-daemon
```

During onboarding, select **Zhipu AI** as your provider when prompted.

### Set up Zhipu AI API key

You'll need a Zhipu AI API key to use GLM-5.0 models. Get one from: https://open.bigmodel.cn/usercenter/apikeys

Set as environment variable to avoid storing in config file:

```bash
# Add to ~/.bashrc or ~/.zshrc
export ZHIPU_API_KEY="your-api-key-here"
```

---

## 3) Keep Gateway loopback-only `[All]`

This is the safest remote pattern:

- Gateway listens only on `127.0.0.1:18789`.
- You forward it when you need access.

---

## 4) Access it remotely (recommended) `[All]`

### Option A: Tailscale (recommended)

Tailscale is the recommended way to access your VPS. It provides encrypted transport, identity-based authentication, and auto-TLS — with zero ports exposed to the public internet.

#### 4.1) VPS-side: Install and authenticate `[All]`

```bash
# Download and install Tailscale VPN client
curl -fsSL https://tailscale.com/install.sh | sh

# Connect to your Tailscale account — prints a URL you open in a browser to log in
sudo tailscale up
```

For a **headless VPS** without a browser, generate an auth key from the [Tailscale admin console](https://login.tailscale.com/admin/settings/keys), then:

```bash
# Log in without a browser using a pre-generated auth key (replace XXXXX with your key)
sudo tailscale up --auth-key=tskey-auth-XXXXX
```

Verify connection:

```bash
# Shows all devices on your tailnet and their connection status
tailscale status
# Prints just this machine's tailnet IP address (starts with 100.x.y.z)
tailscale ip -4
```

#### 4.2) Client-side: Install on your personal devices `[All]`

Install Tailscale on every device you'll use to access the VPS:

| Platform | Install Method |
|----------|---------------|
| macOS | Download from https://tailscale.com/download or `brew install tailscale` |
| Windows | Download from https://tailscale.com/download |
| Linux | `curl -fsSL https://tailscale.com/install.sh | sh` |
| iOS/Android | App Store / Play Store |

After installing on each device:
1. Open Tailscale and log in with **same identity provider** (Google, Microsoft, Apple) you used for the VPS
2. Verify device appears on the [Machines page](https://login.tailscale.com/admin/machines) in admin console
3. Confirm you can reach the VPS by its tailnet IP: `ping 100.x.y.z`

#### 4.3) Shields-up on personal devices `[All]`

`shields-up` blocks **all** incoming Tailscale connections to a device. Enable it on laptops and phones that only need to *initiate* connections to the VPS:

```bash
# Run this on your LAPTOP/PHONE (not on VPS):
# Blocks all incoming connections from other tailnet devices to this machine
tailscale set --shields-up
```

This prevents other devices on your tailnet from connecting to your laptop — reducing attack surface if another tailnet device is compromised.

#### 4.4) UFW firewall for Tailscale-only VPS `[Intermediate]`

If you **only** access the VPS via Tailscale (no public SSH), lock the firewall to the `tailscale0` interface:

```bash
# Block everything from public internet by default
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Only allow traffic arriving through the Tailscale VPN tunnel
# "tailscale0" is the virtual network interface Tailscale creates
sudo ufw allow in on tailscale0 to any

# Activate firewall (must set defaults before enabling)
sudo ufw enable

# Review rules — you should see "ALLOW IN" only for tailscale0
sudo ufw status verbose
```

This means port 22 is only reachable via tailnet, not via public internet. Your cloud provider's firewall should also deny public SSH.

#### 4.5) Configure Tailscale Serve (tailnet-only HTTPS) `[Intermediate]`

`tailscale serve` exposes a local port to your tailnet with automatic TLS — no certbot needed:

```bash
# Expose your local gateway (port 18789) as HTTPS on your private tailnet
# --bg runs it in background; --https=443 means it listens on standard HTTPS port
sudo tailscale serve --bg --https=443 127.0.0.1:18789

# See what's currently being served
tailscale serve status

# Remove the serve config (stops exposing the port)
tailscale serve reset
```

Tailscale handles TLS certificate provisioning automatically. Access at `https://<machine-name>.<tailnet>.ts.net/`.

Configure OpenClaw to accept Tailscale identity:

```bash
# In openclaw.json or via CLI
openclaw config set gateway.auth.allowTailscale true
```

Or run Gateway with Tailscale mode:

```bash
openclaw gateway --tailscale serve
```

---

## 5) Verify and lock down `[All]`

On VPS:

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

## 6) Logging + troubleshooting on a server `[All]`

Common troubleshooting workflow:

```bash
# Show status of all OpenClaw components (gateway, agents, channels)
openclaw status --all
# Stream live logs — useful for watching what happens in real time (Ctrl+C to stop)
openclaw logs --follow
```

If service appears running but probe fails:
- you may have a profile/config mismatch
- or process is alive but not listening

Docs: https://docs.openclaw.ai/help/faq

---

## 7) Operational advice for VPS safety `[All]`

- Keep plugins to a minimum.
- Use separate messaging accounts for bot.
- Treat browser control endpoints like admin APIs.
- Rotate tokens and API keys if you suspect exposure.
- Keep `~/.openclaw/` permissions tight (`chmod 700 ~/.openclaw`).
- Run `openclaw security audit --deep` regularly.

### Disable mDNS/Bonjour discovery `[All]`

By default, OpenClaw Gateway broadcasts its presence on the local network using mDNS (Bonjour). On a VPS, this is unnecessary and potentially leaks information.

Disable it by adding to your `~/.profile` or `~/.bashrc`:

```bash
export OPENCLAW_DISABLE_BONJOUR=1
```

Or set in `openclaw.json`:

```json
{
  "discovery": {
    "mdns": { "mode": "off" }
  }
}
```

### Protect shell history from credential leakage

```bash
# Add to ~/.profile or ~/.bashrc
# ignoreboth = don't save commands that start with a space OR duplicate commands
export HISTCONTROL=ignoreboth
# Set history file size to 0 = don't write any commands to on-disk history file
export HISTFILESIZE=0
```

Docs: https://docs.openclaw.ai/gateway/security

---

## 8) Automatic Security Updates `[All]`

Keep your system patched automatically:

```bash
# Install automatic update tool
sudo apt install unattended-upgrades

# Interactive setup — select "Yes" to enable automatic security updates
sudo dpkg-reconfigure unattended-upgrades
```

This ensures critical security patches are applied without manual intervention.

---

## 9) SSH Hardening with Fail2ban `[All]`

Protect against SSH brute force attacks:

```bash
# Install fail2ban — monitors SSH login attempts and bans IPs after too many failures
sudo apt install fail2ban
# Start fail2ban on boot
sudo systemctl enable fail2ban
# Start it now
sudo systemctl start fail2ban
```

Check status:

```bash
# Shows how many IPs are currently banned and total failed attempts
sudo fail2ban-client status sshd
```

---

## 10) GLM-5.0 Performance Considerations `[All]`

GLM-5.0's larger context window (~200K tokens) means:

### Memory requirements

| VPS Size | Recommended RAM | Minimum | Reason |
|------------|----------------|---------|---------|
| 1 GB | 2 GB | 2 GB | GLM-5.0's ~200K context + processing overhead |
| 2 GB | 4 GB | 3 GB | Headroom for model + OS |
| 4 GB | 8 GB | 6 GB | Comfortable for long conversations |

### Streaming support

GLM-5.0 supports streaming responses. Ensure your network connection is stable for the best experience.

---

## Advanced: Reverse Proxy Hardening (nginx) `[Advanced]`

> Skip this section if you chose Tailscale Serve (section 4) — Tailscale handles TLS and doesn't need a reverse proxy.

OpenClaw has **no built-in rate limiting** and only adds security headers on Control UI endpoints. A reverse proxy fills both gaps.

### Install nginx

```bash
sudo apt install nginx
```

### Full hardened config

Create `/etc/nginx/sites-available/openclaw`:

```nginx
# Rate limiting zones
limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=auth:10m    rate=3r/s;

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    # TLS (certbot fills these in — see TLS section below)
    ssl_certificate     /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    # Security headers
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Content-Type-Options    "nosniff" always;
    add_header X-Frame-Options           "DENY" always;
    add_header Referrer-Policy           "no-referrer" always;
    add_header Permissions-Policy        "camera=(), microphone=(), geolocation=()" always;
    add_header Content-Security-Policy   "default-src 'self'; frame-ancestors 'none'" always;

    # Request body limit (mitigates large-payload DoS)
    client_max_body_size 10m;

    # General rate limit
    limit_req zone=general burst=20 nodelay;

    # Stricter rate limit for auth endpoints
    location /api/auth {
        limit_req zone=auth burst=5 nodelay;
        proxy_pass http://127.0.0.1:18789;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Main proxy with WebSocket support
    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket upgrade
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$host$request_uri;
}
```

Enable the site:

```bash
# Create a symbolic link to activate the config
sudo ln -s /etc/nginx/sites-available/openclaw /etc/nginx/sites-enabled/
# Remove the default "Welcome to nginx" site
sudo rm /etc/nginx/sites-enabled/default
```

**Note:** Run `sudo nginx -t && sudo systemctl reload nginx` only **after** obtaining your TLS certificate in the TLS section below. The config above references certificate paths that won't exist until certbot creates them.

### Configure trusted proxies (mandatory)

When using a reverse proxy, you **must** tell OpenClaw to trust the proxy's `X-Forwarded-For` header. Add to `openclaw.json`:

```json
{
  "gateway": {
    "trustedProxies": ["127.0.0.1"]
  }
}
```

Without this, IP-based rate limiting and logging will see the proxy IP instead of the real client IP.

---

## Advanced: TLS Termination `[Advanced]`

> Skip this section if you chose Tailscale Serve (section 4) — TLS is automatic.

### Install certbot

```bash
sudo apt install certbot python3-certbot-nginx
```

### Obtain certificate

```bash
# Request a free TLS certificate for your domain and auto-configure nginx
# You'll be asked for an email address and to agree to terms of service
sudo certbot --nginx -d your-domain.com
```

Certbot will automatically update your nginx config with certificate paths.

### Auto-renewal

```bash
# Set up a cron job that checks for certificate renewal twice daily
# The random sleep prevents millions of servers hitting Let's Encrypt at the same second
SLEEPTIME=$(awk 'BEGIN{srand(); print int(rand()*(3600+1))}')
echo "0 0,12 * * * root sleep $SLEEPTIME && certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```

---

## Security Checklist (VPS)

Based on security best practices and GLM-5.0 deployment.

### Baseline Hardening

- [ ] SSH password authentication disabled (`PasswordAuthentication no`)
- [ ] SSH root login disabled (`PermitRootLogin no`)
- [ ] SSH config validated with `sshd -t` before reload
- [ ] Swap file configured (prevents OOM kills)
- [ ] mDNS/Bonjour broadcasting disabled (`OPENCLAW_DISABLE_BONJOUR=1`)
- [ ] NTP time synchronization active (chrony or systemd-timesyncd)
- [ ] Configuration backup schedule active

### Network

- [ ] Security group inbound is SSH only from your IP
- [ ] Gateway port 18789 is NOT public
- [ ] Host firewall (UFW) is enabled
- [ ] Fail2ban is active for SSH protection
- [ ] If using Tailscale: UFW restricted to `tailscale0` interface

### Authentication & Access

- [ ] Gateway auth token is set
- [ ] DM policy is `allowlist` or `pairing`
- [ ] Only approved user IDs can trigger actions
- [ ] If using Tailscale: `allowTailscale: true` is set

### GLM/Zhipu AI Specific

- [ ] Zhipu AI API key stored in env var (not in config file)
- [ ] API endpoint configured correctly
- [ ] Function calling audited and logged
- [ ] Model context size appropriate for VPS memory

### Execution Safety

- [ ] Docker sandbox enabled for execution tools
- [ ] Sandbox has `network: none` or strict isolation
- [ ] Tools restricted to minimum needed
- [ ] Gateway tool removed from tool allowlist

### Secrets

- [ ] Secrets stored in env vars (not in shell history)
- [ ] Sensitive files set to `chmod 600`
- [ ] Shell history protected (`HISTCONTROL=ignoreboth`)
- [ ] `~/.openclaw/` on encrypted volume (LUKS or provider-level)

### System Maintenance

- [ ] Automatic security updates enabled
- [ ] Node.js 22.12.0+ installed
- [ ] Systemd resource limits configured
- [ ] Backup schedule active

### Tailscale (if using)

- [ ] Tailscale installed and authenticated on VPS
- [ ] `tailscale.mode: "serve"` configured (tailnet-only)
- [ ] `gateway.auth.allowTailscale: true` set
- [ ] Shields-up enabled on personal devices
- [ ] UFW restricted to `tailscale0` interface (if Tailscale-only)

---

See also: [Mac Mini deployment](../05-use-cases/mac-mini-standalone.md) for local-first alternative
