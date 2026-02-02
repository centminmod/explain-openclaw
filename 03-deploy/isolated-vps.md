# Deployment runbook: Isolated VPS (remote + locked down)

> **Note:** This guide is for OpenClaw (formerly Moltbot/Clawdbot).

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
    - [DigitalOcean 1-Click Deploy](#11-digitalocean-1-click-deploy)
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

Related official docs:
- https://docs.openclaw.ai/gateway/remote
- https://docs.openclaw.ai/gateway/security
- https://docs.openclaw.ai/platforms/linux
- https://docs.openclaw.ai/help/faq

---

## Recommended posture (summary)

- Keep Gateway **loopback-only** (`gateway.bind: "loopback"`).
- Access via **SSH tunnel** or **tailnet** (Tailscale).
- Use token/password auth.
- Run as a dedicated non-root user.
- Lock down file permissions.

---

## 1) Provision the VPS (baseline hardening)

Based on [VibeProof Security Guide](https://vibeproof.dev/blog/moltbot-security-setup-guide) (uses legacy "Moltbot" name).

### Choose a Provider
- AWS EC2: t3.small, Ubuntu 24.04 LTS
- DigitalOcean: Basic $6/month Droplet, Ubuntu 24.04
- Linode: Nanode 1GB, Ubuntu 24.04
- Hetzner: CX11, Ubuntu 24.04

### Initial Setup

```bash
# Connect to your VPS
ssh -i your-key.pem ubuntu@YOUR_SERVER_IP

# Update system
sudo apt update && sudo apt upgrade -y

# Create dedicated user
sudo adduser moltbot
sudo usermod -aG sudo moltbot
```

### Firewall Configuration (Critical)

Only allow SSH from your IP. **Never allow `0.0.0.0/0` (anywhere) access.**

```bash
# Enable UFW
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw enable

# Verify
sudo ufw status
```

**Cloud firewall**: Also configure your provider's firewall (Security Groups on AWS, etc.) to only allow SSH from your IP. Do **not** open 18789 to the public internet.

---

## 2) Install OpenClaw

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

## 3) Keep the Gateway loopback-only

This is the safest remote pattern:
- Gateway listens only on `127.0.0.1:18789`.
- You forward it when you need access.

---

## 4) Access it remotely (recommended)

### Option A: SSH tunnel (universal)

From your laptop:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
```

Now your local browser can open:
- http://127.0.0.1:18789/

…and your local CLI can talk to the Gateway at the forwarded URL.

Docs: https://docs.openclaw.ai/gateway/remote

### Option B: Tailnet (Tailscale)

If you use Tailscale:
- you can either bind directly to tailnet IP (`gateway.bind: "tailnet"`), or
- keep loopback-only and publish the dashboard via Serve (HTTPS)

Docs: https://docs.openclaw.ai/gateway/tailscale

---

## 5) Verify and lock down

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

## 6) Logging + troubleshooting on a server

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

## 7) Operational advice for VPS safety

- Keep plugins to a minimum.
- Use separate messaging accounts for the bot.
- Treat browser control endpoints as admin APIs.
- Rotate tokens and API keys if you suspect exposure.
- Keep `~/.openclaw/` permissions tight (`chmod 700 ~/.openclaw`).

Protect shell history from credential leakage:

```bash
# Add to ~/.profile or ~/.bashrc
export HISTCONTROL=ignoreboth
export HISTFILESIZE=0
```

Docs: https://docs.openclaw.ai/gateway/security

---

## 8) Automatic Security Updates

Keep your system patched automatically:

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

This ensures critical security patches are applied without manual intervention.

---

## 9) SSH Hardening with Fail2ban

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

## 10) Systemd Resource Limits

Add resource limits to your OpenClaw service file (`/etc/systemd/system/moltbot.service`):

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

## 11) DigitalOcean 1-Click Deploy

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
- Optionally add SSH tunnel or Tailscale for remote access (see [section 4](#4-access-it-remotely-recommended))
- Review the [Security Checklist](#security-checklist-vps) below

### Resources

- **OpenClaw Documentation:** https://docs.openclaw.ai/
- **Gateway Configuration:** https://docs.openclaw.ai/gateway/configuration
- **Channel Setup:** https://docs.openclaw.ai/channels
- **Security Guide:** https://docs.openclaw.ai/gateway/security
- **Discord Community:** https://discord.gg/molt
- **GitHub:** https://github.com/openclaw/openclaw

---

## Security Checklist (VPS)

Based on [VibeProof Security Guide](https://vibeproof.dev/blog/moltbot-security-setup-guide) (uses legacy "Moltbot" name).

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

### System Maintenance
- [ ] Automatic security updates enabled
- [ ] Node.js 22.12.0+ installed
- [ ] Systemd resource limits configured

### Observability
- [ ] Session logging enabled
- [ ] Log rotation active (`/var/log/moltbot/`)
- [ ] Weekly review habit
