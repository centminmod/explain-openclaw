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
    - [Tailscale Setup](#12-tailscale-setup-recommended)
    - [Reverse Proxy Hardening](#13-reverse-proxy-hardening-nginx)
    - [TLS Termination](#14-tls-termination)
    - [Encrypted Storage for Secrets](#15-encrypted-storage-for-secrets)
    - [Docker Deployment Hardening](#16-docker-deployment-hardening)
    - [OpenClaw Config Hardening](#17-openclaw-config-hardening)
    - [Gateway Token Rotation](#18-gateway-token-rotation)
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

## 12) Tailscale Setup (recommended)

> If you use Tailscale, you can skip sections 13 (Reverse Proxy) and 14 (TLS) entirely — Tailscale handles encrypted transport and identity-based auth automatically.

For most single-user VPS deployments, Tailscale is simpler and more secure than the nginx reverse proxy + certbot path. It provides built-in TLS, identity-based authentication, and zero port exposure.

### Install Tailscale on VPS

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

### Configure OpenClaw for Tailscale Serve

Add to your `openclaw.json`:

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

### Access

Open `https://<your-machine>.<tailnet>.ts.net/` in your browser — auto-HTTPS, no port forwarding needed.

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

## 13) Reverse Proxy Hardening (nginx)

> Skip this section if you chose Tailscale Serve ([section 12](#12-tailscale-setup-recommended)) — Tailscale handles TLS and doesn't need a reverse proxy.

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

**Note:** Run `sudo nginx -t && sudo systemctl reload nginx` only **after** obtaining your TLS certificate in [section 14](#14-tls-termination). The config above references certificate paths that won't exist until certbot creates them.

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

## 14) TLS Termination

> Skip this section if you chose Tailscale Serve ([section 12](#12-tailscale-setup-recommended)) — TLS is automatic.

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

## 15) Encrypted Storage for Secrets

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

## 16) Docker Deployment Hardening

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

## 17) OpenClaw Config Hardening

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

## 18) Gateway Token Rotation

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

### Tailscale (if using Tailscale path — [section 12](#12-tailscale-setup-recommended))
- [ ] Tailscale installed and authenticated
- [ ] `tailscale.mode: "serve"` configured (tailnet-only)
- [ ] `gateway.auth.allowTailscale: true` set
- [ ] Funnel NOT enabled unless explicitly needed (forces password auth)

### Reverse Proxy & TLS (if using nginx path — [sections 13-14](#13-reverse-proxy-hardening-nginx))
- [ ] Reverse proxy (nginx/Caddy) in front of gateway
- [ ] Rate limiting configured (general + auth endpoints)
- [ ] Security headers set (HSTS, CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy)
- [ ] TLS termination with valid certificate
- [ ] `gateway.trustedProxies` configured

### Encrypted Storage ([section 15](#15-encrypted-storage-for-secrets))
- [ ] `~/.openclaw/` on encrypted volume (LUKS or provider-level)
- [ ] Session transcripts permissions 600 (not 644)

### Docker Hardening (for Docker deployments — [section 16](#16-docker-deployment-hardening))
- [ ] `OPENCLAW_GATEWAY_BIND=loopback` in `.env`
- [ ] `cap_drop: ALL` with minimal `cap_add`
- [ ] `no-new-privileges` security option enabled
- [ ] `read_only: true` with tmpfs for writable paths
- [ ] Resource limits (memory, PIDs) configured

### OpenClaw Config ([section 17](#17-openclaw-config-hardening))
- [ ] `browser.evaluateEnabled: false`
- [ ] Plugins disabled or allowlisted
- [ ] Sandbox mode `"all"` for agents processing untrusted input
- [ ] Gateway token >= 32 chars
- [ ] Token rotation schedule active
