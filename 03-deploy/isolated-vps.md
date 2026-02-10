# Deployment runbook: Isolated VPS (remote + locked down)

> **Note:** This guide is for OpenClaw (formerly Moltbot/Clawdbot).

## Skill level key

| Tag | Meaning |
|-----|---------|
| `[All]` | Everyone should do this |
| `[Intermediate]` | Recommended for users comfortable with Linux CLI |
| `[Advanced]` | Optional hardening for security-focused deployments |

## Table of contents (Explain OpenClaw)

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
- Deployment
  - [Standalone Mac mini](./standalone-mac-mini.md)
  - [Isolated VPS](./isolated-vps.md)
    - [DigitalOcean 1-Click Deploy](#11-digitalocean-1-click-deploy-all)
    - [Tailscale Setup](#12-tailscale-setup-recommended-intermediate)
    - [Reverse Proxy Hardening](#13-reverse-proxy-hardening-nginx-advanced)
    - [TLS Termination](#14-tls-termination-advanced)
    - [Encrypted Storage for Secrets](#15-encrypted-storage-for-secrets-advanced)
    - [Docker Deployment Hardening](#16-docker-deployment-hardening-advanced)
    - [OpenClaw Config Hardening](#17-openclaw-config-hardening-advanced)
    - [Gateway Token Rotation](#18-gateway-token-rotation-advanced)
  - [Cloudflare Moltworker](./cloudflare-moltworker.md)
  - [Docker Model Runner](./docker-model-runner.md)
- Optimizations
  - [Cost + token optimization](../06-optimizations/cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

Goal: run the Gateway on a Linux VPS while keeping access **private** and the host hardened.

This is a great setup when:
- you want the assistant always-on
- your laptop sleeps often
- you want predictable networking

But the security bar is higher: a VPS is an internet-connected machine.

> **Recommended path:** For most users, follow sections 1-6 (baseline setup), then jump to [section 12 (Tailscale)](#12-tailscale-setup-recommended-intermediate). Tailscale gives you encrypted access, auto-TLS, and zero exposed ports — without needing nginx, certbot, or manual TLS. Sections 13-18 are for advanced users who need public-facing access or additional hardening.

Related official docs:
- https://docs.openclaw.ai/gateway/remote
- https://docs.openclaw.ai/gateway/security
- https://docs.openclaw.ai/platforms/linux
- https://docs.openclaw.ai/help/faq

---

## Recommended posture (summary)

- Keep Gateway **loopback-only** (`gateway.bind: "loopback"`).
- Access via **Tailscale** (recommended) or SSH tunnel (fallback).
- Use token/password auth — or Tailscale identity-based auth.
- Run as a dedicated non-root user.
- Lock down file permissions.
- Disable mDNS discovery on the VPS.

---

## 1) Provision the VPS (baseline hardening) `[All]`

Based on [VibeProof Security Guide](https://vibeproof.dev/blog/moltbot-security-setup-guide) (uses legacy "Moltbot" name).

### Choose a Provider
- AWS EC2: t3.small, Ubuntu 24.04 LTS
- DigitalOcean: Basic $6/month Droplet, Ubuntu 24.04
- Linode: Nanode 1GB, Ubuntu 24.04
- Hetzner: CX11, Ubuntu 24.04

### SSH Key Setup (if you don't have one)

If you don't already have an SSH key pair, generate one on your **local machine** (not the server):

```bash
# Generate an Ed25519 key (recommended — stronger than RSA, shorter keys)
ssh-keygen -t ed25519 -C "your-email@example.com"

# Or RSA 4096-bit if Ed25519 is not supported
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

When prompted for a passphrase, **set one** — it protects your key if your laptop is compromised.

Most cloud providers let you upload your public key (`~/.ssh/id_ed25519.pub`) during VPS creation. If your provider doesn't, copy it after first login:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@YOUR_SERVER_IP
```

### Initial Setup

```bash
# Connect to your VPS
ssh -i ~/.ssh/id_ed25519 ubuntu@YOUR_SERVER_IP

# Update system
sudo apt update && sudo apt upgrade -y

# Create dedicated user
sudo adduser moltbot
sudo usermod -aG sudo moltbot
```

### Firewall Configuration (Critical)

Only allow SSH from your IP. **Never allow `0.0.0.0/0` (anywhere) access.**

```bash
# Set defaults FIRST, then enable (correct order)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw enable

# Verify
sudo ufw status
```

**Cloud firewall**: Also configure your provider's firewall (Security Groups on AWS, etc.) to only allow SSH from your IP. Do **not** open 18789 to the public internet.

### SSH Hardening (Critical)

Disable password authentication and root login to prevent brute-force attacks:

```bash
sudo nano /etc/ssh/sshd_config
```

Set or uncomment these lines:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

**Before reloading**, validate the config to avoid locking yourself out:

```bash
# Test config syntax (catches typos that would break sshd)
sudo sshd -t
```

If the test passes with no output, reload SSH:

```bash
sudo systemctl reload ssh
```

**Lockout prevention:** Open a **second terminal** and test SSH login before closing your current session. If the new connection works, you're safe. If it fails, fix `sshd_config` in the still-open session.

> **Beginner tip:** If you do get locked out, use your provider's web console (DigitalOcean "Access" tab, AWS "EC2 Instance Connect", etc.) to fix the config.

### Swap File Setup

Small VPS instances (1-2 GB RAM) can run out of memory during agent sessions, causing the process to be killed (OOM). A swap file acts as emergency overflow:

```bash
# Create 2GB swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Verify
free -h
```

### NTP Time Synchronization `[Intermediate]`

Accurate timestamps are essential for security log correlation and debugging. Most cloud providers sync time automatically, but verify and install `chrony` if needed:

```bash
# Check current time sync status
timedatectl status

# If "NTP service: inactive", install chrony
sudo apt install chrony
sudo systemctl enable chrony

# Verify
chronyc tracking
```

---

## 2) Install OpenClaw `[All]`

On the VPS:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Verify Node.js version (22.12.0+ recommended for security patches):

```bash
node --version  # Should be v22.12.0 or later
```

Then onboard:

```bash
openclaw onboard --install-daemon
```

Set a gateway auth token for production:

```bash
export GATEWAY_AUTH_TOKEN="$(openssl rand -hex 32)"
echo "export GATEWAY_AUTH_TOKEN='$GATEWAY_AUTH_TOKEN'" >> ~/.profile
```

If you're headless and need OAuth-style auth, do the auth step on a trusted machine first and copy the required credential files as documented.

Docs: https://docs.openclaw.ai/start/getting-started

---

## 3) Keep the Gateway loopback-only `[All]`

This is the safest remote pattern:
- Gateway listens only on `127.0.0.1:18789`.
- You forward it when you need access.

---

## 4) Access it remotely (recommended) `[All]`

### Option A: Tailscale (recommended)

Tailscale is the recommended way to access your VPS. It provides encrypted transport, identity-based authentication, and auto-TLS — with zero ports exposed to the public internet. No nginx or certbot needed.

Install Tailscale on both the VPS and your personal devices, then use Tailscale Serve to publish the Gateway over HTTPS to your private tailnet. Full walkthrough in [section 12](#12-tailscale-setup-recommended-intermediate).

Docs: https://docs.openclaw.ai/gateway/tailscale

### Option B: SSH tunnel (universal fallback)

If you can't use Tailscale (e.g., corporate policy, no Tailscale account), SSH tunnelling works everywhere:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
```

Now your local browser can open:
- http://127.0.0.1:18789/

…and your local CLI can talk to the Gateway at the forwarded URL. The downside is you must keep the SSH session open, and there's no auto-TLS or identity-based auth.

Docs: https://docs.openclaw.ai/gateway/remote

---

## 5) Verify and lock down `[All]`

On the VPS:

```bash
openclaw gateway status
openclaw status
openclaw health
openclaw security audit --deep
```

If needed:

```bash
openclaw security audit --fix
```

---

## 6) Logging + troubleshooting on a server `[All]`

Common troubleshooting workflow:

```bash
openclaw status --all
openclaw logs --follow
```

If the service appears running but the probe fails:
- you may have a profile/config mismatch
- or the process is alive but not listening

Docs: https://docs.openclaw.ai/help/faq

---

## 7) Operational advice for VPS safety `[All]`

- Keep plugins to a minimum.
- Use separate messaging accounts for the bot.
- Treat browser control endpoints as admin APIs.
- Rotate tokens and API keys if you suspect exposure.
- Keep `~/.openclaw/` permissions tight (`chmod 700 ~/.openclaw`).

### Disable mDNS/Bonjour discovery

By default, the OpenClaw Gateway broadcasts its presence on the local network using mDNS (Multicast DNS, also called Bonjour). It advertises a `_openclaw-gw._tcp` service so that OpenClaw clients (desktop app, mobile apps) can auto-discover the Gateway without manual configuration.

On a VPS, this is a problem:
- **Shared hosting**: other tenants on the same network segment can see that OpenClaw is running and learn your hostname.
- **Dedicated VPS**: mDNS is link-local only (doesn't cross routers), so the risk is lower, but there's still no benefit — you're accessing via SSH tunnel or Tailscale, not local discovery.

Disable it by adding to your `~/.profile` or `~/.bashrc`:

```bash
export OPENCLAW_DISABLE_BONJOUR=1
```

Or set it in `openclaw.json`:

```json
{
  "discovery": {
    "mdns": { "mode": "off" }
  }
}
```

Source: `src/infra/bonjour.ts:29`, `src/gateway/server-discovery-runtime.ts:27`. Cross-reference: [Threat model - mDNS/Bonjour](../04-privacy-safety/threat-model.md#6-mdnsbonjour-network-discovery), [Hardening checklist section 10](../04-privacy-safety/hardening-checklist.md#10-control-mdnsbonjour-discovery).

### Protect shell history from credential leakage

```bash
# Add to ~/.profile or ~/.bashrc
export HISTCONTROL=ignoreboth
export HISTFILESIZE=0
```

Docs: https://docs.openclaw.ai/gateway/security

---

## 8) Automatic Security Updates `[All]`

Keep your system patched automatically:

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

This ensures critical security patches are applied without manual intervention.

---

## 9) SSH Hardening with Fail2ban `[All]`

Protect against SSH brute force attacks:

```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Check status:

```bash
sudo fail2ban-client status sshd
```

---

## 10) Systemd Service and Resource Limits `[Intermediate]`

The `openclaw onboard --install-daemon` command creates a systemd user service. If you need to customize it or create one manually, here is a complete service file.

### Complete systemd user service file

Create `~/.config/systemd/user/openclaw-gateway.service`:

```ini
[Unit]
Description=OpenClaw Gateway
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=%h/.openclaw/bin/openclaw gateway --foreground
Restart=on-failure
RestartSec=5
Environment=NODE_ENV=production

# Resource limits (adjust for your VPS size)
MemoryMax=1G
CPUQuota=80%

# Security hardening
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=read-only
ReadWritePaths=%h/.openclaw

[Install]
WantedBy=default.target
```

Enable and start:

```bash
systemctl --user daemon-reload
systemctl --user enable openclaw-gateway
systemctl --user start openclaw-gateway

# Allow service to run when you're not logged in
loginctl enable-linger $(whoami)
```

### System-level service (alternative)

If you run OpenClaw as a system service instead (e.g., `/etc/systemd/system/moltbot.service`), add the same resource limits:

```ini
[Service]
MemoryMax=1G
CPUQuota=80%
```

Then reload:

```bash
sudo systemctl daemon-reload
sudo systemctl restart moltbot
```

This prevents runaway processes from consuming all system resources.

---

## 10a) Backup and Restore `[All]`

OpenClaw stores configuration, credentials, and session data in `~/.openclaw/`. Back it up regularly.

### Create a backup

```bash
# Full backup (config + credentials + sessions)
tar czf openclaw-backup-$(date +%Y%m%d).tar.gz -C ~/ .openclaw/

# Privacy-conscious backup (exclude session transcripts)
tar czf openclaw-backup-$(date +%Y%m%d).tar.gz \
  --exclude='.openclaw/sessions' \
  --exclude='.openclaw/workspace' \
  -C ~/ .openclaw/
```

### Restore from backup

```bash
# Stop the service first
systemctl --user stop openclaw-gateway

# Restore
tar xzf openclaw-backup-YYYYMMDD.tar.gz -C ~/

# Fix permissions
chmod 700 ~/.openclaw
chmod 600 ~/.openclaw/openclaw.json

# Restart
systemctl --user start openclaw-gateway
```

### Recommendations

- Store backups in an **encrypted location** (e.g., LUKS volume, encrypted S3 bucket). The backup contains API keys and tokens in plaintext.
- Schedule weekly backups via cron:

```bash
echo "0 2 * * 0 $(whoami) tar czf /home/$(whoami)/backups/openclaw-backup-\$(date +\%Y\%m\%d).tar.gz -C /home/$(whoami)/ .openclaw/" | sudo tee -a /etc/crontab > /dev/null
```

---

## 10b) Browser Agent on Headless VPS `[Intermediate]`

If you want OpenClaw's browser agent to work on a headless VPS, you need a headless Chromium installation. OpenClaw manages the browser process internally — do **not** run a standalone Chrome/CDP service.

### Install headless Chromium

```bash
# Install from Ubuntu repos (NOT a standalone .deb)
sudo apt install chromium-browser

# Verify
chromium-browser --version
```

### Configure OpenClaw

Add to `openclaw.json`:

```json
{
  "browser": {
    "evaluateEnabled": false
  }
}
```

`evaluateEnabled: false` prevents arbitrary JavaScript execution in the browser context — see [section 17](#17-openclaw-config-hardening-advanced) for full config hardening.

### What NOT to do

- Do **not** install Chrome via Homebrew, snap, or standalone `.deb` packages — `apt` is simpler and gets security updates automatically.
- Do **not** run a standalone Chrome DevTools Protocol (CDP) service on a fixed port (e.g., `--remote-debugging-port=18800`). This exposes a high-privilege debug interface. OpenClaw launches and manages its own browser instance.
- Do **not** install third-party browser management packages (`agent-browser`, etc.) — OpenClaw's gateway handles browser lifecycle internally.

---

## 11) DigitalOcean 1-Click Deploy `[All]`

> **Fastest path to a hardened VPS.** The DigitalOcean Marketplace app pre-configures security best practices automatically.

Official resources:
- Marketplace: https://marketplace.digitalocean.com/apps/moltbot
- Tutorial: https://www.digitalocean.com/community/tutorials/how-to-run-moltbot

### What 1-Click handles for you

The 1-Click deployment configures these security controls out of the box:

| Control | Description |
|---------|-------------|
| Authenticated gateway token | Auto-generated; protects against unauthorized access |
| Hardened firewall rules | Rate-limiting on OpenClaw ports to prevent DoS |
| Non-root execution | OpenClaw runs as a non-root user, limiting attack surface |
| Docker container isolation | Sandboxed execution environment |
| Private DM pairing | Enabled by default; prevents unauthorized messaging |

### System requirements

| Usage Level | RAM | CPU | Recommended For |
|-------------|-----|-----|-----------------|
| Personal (1-5 users) | 4GB | 2 | Individual use, few channels |
| Small Team (5-20) | 8GB | 4 | Multiple channels |
| Medium Team (20-50) | 16GB | 8 | Heavy usage |
| Large Team (50+) | 32GB | 16 | High-volume deployment |

### Step 1: Create the Droplet

**Via DigitalOcean Console:**
1. Sign in to DigitalOcean -> **Create Droplet**
2. Under **Choose an Image** -> **Marketplace** tab
3. Search for "OpenClaw" and select it
4. Choose at least **4GB RAM** (s-2vcpu-4gb or higher)
5. Add your SSH key under **Authentication**
6. Set a hostname (e.g., `moltbot-server`)
7. Click **Create Droplet**

**Via API:**

```bash
curl -X POST -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer '$TOKEN'' -d \
  '{"name":"moltbot-server","region":"nyc3","size":"s-2vcpu-4gb","image":"moltbot"}' \
  "https://api.digitalocean.com/v2/droplets"
```

### Step 2: Connect and configure

Wait for the Droplet to finish provisioning (the dashboard may say "ready" before SSH is available-retry after 60 seconds if needed).

```bash
ssh root@your_droplet_ip
```

The welcome message displays your **Dashboard URL**-save it for browser access.

**Interactive setup:**
1. Choose LLM Provider: Anthropic (recommended), Gradient, or OpenAI (coming soon)
2. Paste your API key when prompted
3. The clawdbot service restarts automatically

### Step 3: Access OpenClaw

**Terminal UI (TUI):**

```bash
/opt/clawdbot-tui.sh
```

**Web Dashboard:**

Open the Dashboard URL from the welcome message in your browser. The URL includes a gateway token for authentication.

### Step 4: Add messaging channels

**WhatsApp:**

```bash
/opt/clawdbot-cli.sh channels add
# Select WhatsApp -> scan the QR code with your phone
```

**Telegram:**
1. Run `/opt/clawdbot-cli.sh channels add` and select Telegram
2. Open Telegram -> chat with @BotFather -> send `/newbot`
3. Follow prompts to create your bot and get a token
4. Paste the bot token back into the CLI
5. Open the Dashboard URL and add your Telegram user ID to the allowlist
6. Start chatting with your bot

### Manual configuration

Edit `/opt/clawdbot.env` for provider, gateway, and channel settings:

```bash
nano /opt/clawdbot.env
systemctl restart clawdbot
```

### Troubleshooting commands

| Task | Command |
|------|---------|
| Check service status | `systemctl status clawdbot` |
| View live logs | `journalctl -u clawdbot -f` |
| Edit environment | `nano /opt/clawdbot.env` |
| Start TUI | `/opt/clawdbot-tui.sh` |

### What's different from manual VPS setup

The 1-Click handles sections 1-3 of this guide automatically:
- VPS provisioning with Ubuntu 24.04 + Node.js 22 + Docker
- OpenClaw installation and service setup
- Gateway configuration with auth token

You still need to:
- Configure messaging channels (Step 4 above)
- Optionally add SSH tunnel or Tailscale for remote access (see [section 4](#4-access-it-remotely-recommended-all))
- Review the [Security Checklist](#security-checklist-vps) below

### Resources

- **OpenClaw Documentation:** https://docs.openclaw.ai/
- **Gateway Configuration:** https://docs.openclaw.ai/gateway/configuration
- **Channel Setup:** https://docs.openclaw.ai/channels
- **Security Guide:** https://docs.openclaw.ai/gateway/security
- **Discord Community:** https://discord.gg/molt
- **GitHub:** https://github.com/openclaw/openclaw

---

## 12) Tailscale Setup (recommended) `[Intermediate]`

> If you use Tailscale, you can skip sections 13 (Reverse Proxy) and 14 (TLS) entirely — Tailscale handles encrypted transport and identity-based auth automatically.

For most single-user VPS deployments, Tailscale is simpler and more secure than the nginx reverse proxy + certbot path. It provides built-in TLS, identity-based authentication, and zero port exposure.

### 12.1) VPS-side: Install and authenticate `[All]`

```bash
# Install Tailscale on VPS
curl -fsSL https://tailscale.com/install.sh | sh

# Interactive login (opens a browser URL to authenticate)
sudo tailscale up
```

For a **headless VPS** without a browser, generate an auth key from the [Tailscale admin console Keys page](https://login.tailscale.com/admin/settings/keys), then:

```bash
sudo tailscale up --auth-key=tskey-auth-XXXXX
```

Verify the connection:

```bash
tailscale status
tailscale ip -4    # Shows your VPS's tailnet IP (e.g. 100.x.y.z)
```

### 12.2) Client-side: Install on your personal devices `[All]`

Install Tailscale on every device you'll use to access the VPS:

| Platform | Install Method |
|----------|---------------|
| macOS | Download from https://tailscale.com/download or `brew install tailscale` |
| Windows | Download from https://tailscale.com/download |
| Linux | `curl -fsSL https://tailscale.com/install.sh \| sh` |
| iOS/Android | App Store / Play Store |

After installing on each device:
1. Open Tailscale and log in with the **same identity provider** (Google, Microsoft, Apple) you used for the VPS
2. Verify the device appears on the [Machines page](https://login.tailscale.com/admin/machines) in the admin console
3. Confirm you can reach the VPS by its tailnet IP: `ping 100.x.y.z`

### 12.3) Shields-up on personal devices `[All]`

`shields-up` blocks **all** incoming Tailscale connections to a device. Enable it on laptops and phones that only need to *initiate* connections to the VPS, never receive them:

```bash
# On your laptop/personal device (NOT the VPS):
tailscale set --shields-up

# To disable later if needed:
tailscale set --shields-up=false
```

This prevents other devices on your tailnet from connecting to your laptop — reducing attack surface if another tailnet device is compromised.

### 12.4) UFW firewall for Tailscale-only VPS `[Intermediate]`

If you **only** access the VPS via Tailscale (no public SSH), lock the firewall to the `tailscale0` interface:

```bash
# Set defaults FIRST (correct order — some blog guides get this wrong)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow all traffic on the Tailscale interface only
sudo ufw allow in on tailscale0

# Enable AFTER setting defaults
sudo ufw enable
sudo ufw status verbose
```

This means port 22 is only reachable via tailnet, not the public internet. Your cloud provider's firewall should also deny public SSH.

### 12.5) Configure Tailscale Serve (tailnet-only HTTPS) `[Intermediate]`

`tailscale serve` exposes a local port to your tailnet with automatic TLS — no certbot needed:

```bash
# Serve OpenClaw gateway (port 18789) over HTTPS to your tailnet
sudo tailscale serve --bg --https=443 127.0.0.1:18789

# Check status
tailscale serve status

# Reset/remove serve config
tailscale serve reset
```

Tailscale handles TLS certificate provisioning automatically. Access at `https://<machine-name>.<tailnet>.ts.net/`.

Configure OpenClaw:

```json
{
  "gateway": {
    "bind": "loopback",
    "tailscale": { "mode": "serve" },
    "auth": { "allowTailscale": true }
  }
}
```

Or via CLI: `openclaw gateway --tailscale serve`

### 12.6) Funnel (public internet) — use with caution `[Advanced]`

Funnel exposes your service to the **public internet**, not just your tailnet:

```bash
# Expose gateway publicly (DANGEROUS for browser control endpoints)
sudo tailscale funnel --https=443 127.0.0.1:18789
```

Funnel ports are restricted to 443, 8443, or 10000.

**Warning:** If using Funnel, you **must** set `gateway.auth.mode: "password"` — Tailscale identity headers are NOT available for public internet requests. Avoid Funnel for browser control endpoints entirely.

### 12.7) ACL hardening (optional) `[Advanced]`

Restrict tailnet access so each user can only reach their own devices. Set this in the [Tailscale admin console ACL editor](https://login.tailscale.com/admin/acls):

```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["autogroup:member"],
      "dst": ["autogroup:self:*"]
    }
  ]
}
```

This is the "users access own devices" pattern — prevents other tailnet members from reaching your VPS.

### Configuration reference

| Key | Value | Purpose |
|-----|-------|---------|
| `gateway.bind` | `"loopback"` | Keep Gateway on 127.0.0.1 |
| `gateway.tailscale.mode` | `"serve"` / `"funnel"` / `"off"` | Exposure level |
| `gateway.auth.allowTailscale` | `true` | Accept Tailscale identity headers |
| `gateway.tailscale.resetOnExit` | `true` | Clean up Serve config on shutdown |

### Security notes

- **Serve** = tailnet-only (recommended). **Funnel** = public internet (forces `auth.mode: "password"`).
- `allowTailscale: true` lets requests from the Tailscale proxy auto-authenticate via identity headers — no manual token entry needed.
- Avoid Funnel for browser control endpoints.
- Tailscale handles TLS automatically — no certbot needed.

**Prerequisites:** Tailscale CLI installed, logged in, HTTPS enabled on your tailnet (for Serve).

Docs: [gateway/tailscale](https://docs.openclaw.ai/gateway/tailscale), [gateway/security](https://docs.openclaw.ai/gateway/security)

---

## 13) Reverse Proxy Hardening (nginx) `[Advanced]`

> Skip this section if you chose Tailscale Serve ([section 12](#12-tailscale-setup-recommended-intermediate)) — Tailscale handles TLS and doesn't need a reverse proxy.

OpenClaw has **no built-in rate limiting** ([#8594](https://github.com/openclaw/openclaw/issues/8594)) and only adds security headers on Control UI endpoints. A reverse proxy fills both gaps.

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

    # TLS (certbot fills these in — see section 14)
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
        proxy_read_timeout 86400s;
    }

    # Optional: IP allowlist (uncomment and replace with your IP)
    # allow YOUR_IP;
    # deny all;
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
sudo ln -s /etc/nginx/sites-available/openclaw /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
```

**Note:** Run `sudo nginx -t && sudo systemctl reload nginx` only **after** obtaining your TLS certificate in [section 14](#14-tls-termination-advanced). The config above references certificate paths that won't exist until certbot creates them.

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

Cross-reference: [Hardening checklist section 11](../04-privacy-safety/hardening-checklist.md#11-configure-trusted-proxies)

Docs: [gateway/security](https://docs.openclaw.ai/gateway/security), [nginx rate limiting](https://nginx.org/en/docs/http/ngx_http_limit_req_module)

---

## 14) TLS Termination `[Advanced]`

> Skip this section if you chose Tailscale Serve ([section 12](#12-tailscale-setup-recommended-intermediate)) — TLS is automatic.

OpenClaw does not set HSTS headers. A reverse proxy with Let's Encrypt TLS fixes this.

### Install certbot

```bash
sudo apt install certbot python3-certbot-nginx
```

### Obtain certificate

```bash
sudo certbot --nginx -d your-domain.com
```

Certbot will automatically update your nginx config with certificate paths.

### Auto-renewal

```bash
SLEEPTIME=$(awk 'BEGIN{srand(); print int(rand()*(3600+1))}')
echo "0 0,12 * * * root sleep $SLEEPTIME && certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```

This checks for renewal twice daily with a random delay to avoid Let's Encrypt traffic spikes.

Docs: [certbot](https://certbot.eff.org), [gateway/tailscale](https://docs.openclaw.ai/gateway/tailscale)

---

## 15) Encrypted Storage for Secrets `[Advanced]`

OpenClaw stores **all secrets** (API keys, OAuth tokens, gateway tokens) in **plaintext** on disk. Filesystem permissions (`0600`/`0700`) are the only protection. No built-in encryption-at-rest exists.

### Option A: LUKS encrypted partition

```bash
# Install cryptsetup
sudo apt install cryptsetup

# Create encrypted volume (DESTRUCTIVE — choose an empty disk/partition)
sudo cryptsetup luksFormat /dev/sdX
sudo cryptsetup open /dev/sdX openclaw-vault
sudo mkfs.ext4 /dev/mapper/openclaw-vault

# Mount as OpenClaw data directory
sudo mount /dev/mapper/openclaw-vault /home/moltbot/.openclaw
sudo chown moltbot:moltbot /home/moltbot/.openclaw
```

Add a systemd mount dependency so the OpenClaw service waits for decryption:

```ini
# In /etc/systemd/system/moltbot.service
[Unit]
RequiresMountsFor=/home/moltbot/.openclaw
```

**Note:** Requires manual unlock on reboot (or a key file, which trades convenience for security).

### Option B: Provider-level encryption

| Provider | Encryption-at-rest |
|----------|-------------------|
| DigitalOcean | Block storage volumes encrypted by default |
| AWS | Enable EBS encryption on the volume |
| Hetzner | Enable LUKS at provisioning time |

### What this protects

- Disk theft, decommissioned VPS, snapshot exposure.

### What this does NOT protect

- Running process memory, authenticated SSH access.

Docs: [gateway/security](https://docs.openclaw.ai/gateway/security) (confirms no built-in encryption)

---

## 16) Docker Deployment Hardening `[Advanced]`

The default `docker-compose.yml` binds to `0.0.0.0` (lan mode) and has no security options enabled.

### Override the bind address

Add to your `.env` file:

```bash
OPENCLAW_GATEWAY_BIND=loopback
```

This overrides the default `${OPENCLAW_GATEWAY_BIND:-lan}` in `docker-compose.yml`.

### Docker Compose security overlay

Create a `docker-compose.override.yml` or add to your existing compose file:

```yaml
services:
  openclaw-gateway:
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETUID
      - SETGID
    read_only: true
    tmpfs:
      - /tmp
      - /home/node/.openclaw/workspace
    pids_limit: 256
    mem_limit: 1g
```

**Note:** The `--no-sandbox` Chrome flag is required in containers (Dockerfile default) and cannot be avoided.

### OpenClaw sandbox Docker config

These settings control the Docker sandbox that OpenClaw creates for agent sessions. Add to `openclaw.json`:

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all",
        "docker": {
          "network": "none",
          "readOnlyRoot": true,
          "capDrop": ["ALL"],
          "pidsLimit": 100,
          "memory": "512m"
        }
      }
    }
  }
}
```

**Caveat:** `readOnlyRoot`, `capDrop`, `pidsLimit`, and `memory` are defined in source code (`src/config/types.sandbox.ts`) but not yet in official docs. They work (tested in `src/config/config.sandbox-docker.test.ts`) but may change between releases.

Docs: [Docker Compose services reference](https://docs.docker.com/reference/compose-file/services/), [gateway/sandboxing](https://docs.openclaw.ai/gateway/sandboxing)

---

## 17) OpenClaw Config Hardening `[Advanced]`

Several security-relevant config options exist but aren't enabled by default. This section consolidates recommended settings for a hardened VPS deployment.

### Recommended `openclaw.json`

```json
{
  "gateway": {
    "bind": "loopback",
    "auth": {
      "mode": "token"
    },
    "trustedProxies": ["127.0.0.1"],
    "controlUi": {
      "dangerouslyDisableDeviceAuth": false
    }
  },
  "browser": {
    "evaluateEnabled": false
  },
  "plugins": {
    "enabled": false
  },
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all",
        "workspaceAccess": "ro"
      }
    }
  },
  "logging": {
    "redactSensitive": "tools"
  }
}
```

### Setting explanations

| Setting | Why |
|---------|-----|
| `gateway.bind: "loopback"` | Only listen on 127.0.0.1 — access via SSH tunnel or Tailscale |
| `gateway.auth.mode: "token"` | Require bearer token for all requests ([docs](https://docs.openclaw.ai/gateway/security)) |
| `gateway.trustedProxies: ["127.0.0.1"]` | Required if using reverse proxy — prevents X-Forwarded-For spoofing ([docs](https://docs.openclaw.ai/gateway/security)) |
| `dangerouslyDisableDeviceAuth: false` | Prevents silently weakening auth |
| `browser.evaluateEnabled: false` | Disables arbitrary JS execution in browser context (`src/config/types.browser.ts:18`) |
| `plugins.enabled: false` | Avoids plugin HTTP route auth bypass ([#8512](https://github.com/openclaw/openclaw/issues/8512)) |
| `agents.defaults.sandbox.mode: "all"` | All sessions run in Docker sandbox ([docs](https://docs.openclaw.ai/gateway/sandboxing)) |
| `agents.defaults.sandbox.workspaceAccess: "ro"` | Read-only workspace prevents file tampering ([docs](https://docs.openclaw.ai/gateway/security)) |
| `logging.redactSensitive: "tools"` | Redact secrets from tool output logs ([docs](https://docs.openclaw.ai/gateway/security)) |

### Gateway token

Set via environment variable (>= 32 characters):

```bash
export OPENCLAW_GATEWAY_TOKEN="$(openssl rand -hex 32)"
```

### Verify

After applying changes, run:

```bash
openclaw security audit --deep
```

Docs: [gateway/security](https://docs.openclaw.ai/gateway/security), [gateway/sandboxing](https://docs.openclaw.ai/gateway/sandboxing)

---

## 18) Gateway Token Rotation `[Advanced]`

OpenClaw has no auto-rotation for gateway tokens. A simple cron script fills the gap.

### Rotation script

The script depends on how OpenClaw was installed. The default `openclaw onboard --install-daemon` stores the token in `~/.openclaw/openclaw.json` and runs as a **systemd user service** (`openclaw-gateway.service`). The DO 1-Click uses a system service with `/opt/clawdbot.env`.

**Default install** (systemd user service — most manual VPS setups):

Create `/usr/local/bin/rotate-openclaw-token.sh`:

```bash
#!/bin/bash
# Default OpenClaw install: token in ~/.openclaw/openclaw.json
# Service: systemd user unit "openclaw-gateway.service"
OPENCLAW_USER="moltbot"
OPENCLAW_HOME="/home/${OPENCLAW_USER}"
CONFIG="${OPENCLAW_HOME}/.openclaw/openclaw.json"
NEW_TOKEN=$(openssl rand -hex 32)

# Update token in openclaw.json using jq
jq --arg t "$NEW_TOKEN" '.gateway.auth.token = $t' "$CONFIG" > "${CONFIG}.tmp" \
  && mv "${CONFIG}.tmp" "$CONFIG" \
  && chown "${OPENCLAW_USER}:${OPENCLAW_USER}" "$CONFIG"

# Restart the systemd user service (runs as the openclaw user)
sudo -u "$OPENCLAW_USER" XDG_RUNTIME_DIR="/run/user/$(id -u $OPENCLAW_USER)" \
  systemctl --user restart openclaw-gateway

echo "$(date): Token rotated" >> /var/log/openclaw-token-rotation.log
```

**DO 1-Click install** (system service with env file):

```bash
#!/bin/bash
# DO 1-Click: token in /opt/clawdbot.env, service name "moltbot"
NEW_TOKEN=$(openssl rand -hex 32)
sed -i "s/^OPENCLAW_GATEWAY_TOKEN=.*/OPENCLAW_GATEWAY_TOKEN=${NEW_TOKEN}/" /opt/clawdbot.env
systemctl restart moltbot
echo "$(date): Token rotated" >> /var/log/openclaw-token-rotation.log
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/rotate-openclaw-token.sh
```

**Prerequisite:** The default install script uses `jq` — install with `sudo apt install jq` if not present.

### Schedule monthly rotation

```bash
echo "0 3 1 * * root /usr/local/bin/rotate-openclaw-token.sh" | sudo tee -a /etc/crontab > /dev/null
```

Runs at 3 AM on the first of each month.

**Important:** After rotation, update any bookmarked dashboard URLs or SSH tunnel configs that embed the old token.

Docs: [gateway/security](https://docs.openclaw.ai/gateway/security) (token auth docs)

---

## Security Checklist (VPS)

Based on [VibeProof Security Guide](https://vibeproof.dev/blog/moltbot-security-setup-guide) (uses legacy "Moltbot" name).

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

### Authentication & Access
- [ ] Gateway auth token is set
- [ ] DM policy is `allowlist` or `pairing`
- [ ] Only approved user IDs can trigger actions

### Execution Safety
- [ ] Docker sandbox enabled for execution tools
- [ ] Sandbox has `network: none` or strict isolation
- [ ] Dangerous command patterns blocked (`rm -rf`, `curl | bash`, etc.)
- [ ] Tools restricted to minimum needed

### Secrets
- [ ] Secrets stored in env vars (not in shell history)
- [ ] Sensitive files set to `chmod 600`
- [ ] Shell history protected (`HISTCONTROL=ignoreboth`)
- [ ] `~/.openclaw/` on encrypted volume (LUKS or provider-level)

### System Maintenance
- [ ] Automatic security updates enabled
- [ ] Node.js 22.12.0+ installed
- [ ] Systemd resource limits configured

### Observability
- [ ] Session logging enabled
- [ ] Log rotation active (`/var/log/moltbot/`)
- [ ] Weekly review habit

### Tailscale (if using Tailscale path — [section 12](#12-tailscale-setup-recommended-intermediate))
- [ ] Tailscale installed and authenticated on VPS
- [ ] Tailscale installed on all client devices (same identity provider)
- [ ] `tailscale.mode: "serve"` configured (tailnet-only)
- [ ] `gateway.auth.allowTailscale: true` set
- [ ] Shields-up enabled on personal devices (`tailscale set --shields-up`)
- [ ] UFW restricted to `tailscale0` interface (if Tailscale-only)
- [ ] Funnel NOT enabled unless explicitly needed (forces password auth)
- [ ] ACL policy reviewed in admin console

### Reverse Proxy & TLS (if using nginx path — [sections 13-14](#13-reverse-proxy-hardening-nginx-advanced))
- [ ] Reverse proxy (nginx/Caddy) in front of gateway
- [ ] Rate limiting configured (general + auth endpoints)
- [ ] Security headers set (HSTS, CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy)
- [ ] TLS termination with valid certificate
- [ ] `gateway.trustedProxies` configured

### Encrypted Storage ([section 15](#15-encrypted-storage-for-secrets-advanced))
- [ ] `~/.openclaw/` on encrypted volume (LUKS or provider-level)
- [ ] Session transcripts permissions 600 (not 644)

### Docker Hardening (for Docker deployments — [section 16](#16-docker-deployment-hardening-advanced))
- [ ] `OPENCLAW_GATEWAY_BIND=loopback` in `.env`
- [ ] `cap_drop: ALL` with minimal `cap_add`
- [ ] `no-new-privileges` security option enabled
- [ ] `read_only: true` with tmpfs for writable paths
- [ ] Resource limits (memory, PIDs) configured

### OpenClaw Config ([section 17](#17-openclaw-config-hardening-advanced))
- [ ] `browser.evaluateEnabled: false`
- [ ] Plugins disabled or allowlisted
- [ ] Sandbox mode `"all"` for agents processing untrusted input
- [ ] Gateway token >= 32 chars
- [ ] Token rotation schedule active
