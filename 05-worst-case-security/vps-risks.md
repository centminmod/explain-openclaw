# VPS Security Risks: What Could Go Wrong

> **In Plain English:** Running OpenClaw on a VPS (Virtual Private Server) is like renting an apartment in a shared building. You have your own space, but you share walls with neighbors, the building has a front door that faces the street, and the landlord has master keys.

**Time to read:** 15 minutes
**Difficulty:** Beginner-friendly with technical context for each risk

---

## Key Difference from Mac Mini

| Aspect | Mac Mini | VPS |
|--------|----------|-----|
| **Who can reach it** | Only people on your WiFi | Anyone on the internet (if firewall misconfigured) |
| **Who else uses the hardware** | Just you | Other VPS customers on same physical server |
| **Physical security** | You control it | Datacenter controls it |
| **Recovery if compromised** | Wipe and restore | Often: destroy and rebuild entirely |
| **Default exposure** | Behind router NAT | Public IP, directly addressable |

---

## A. Network Exposure (The Big Risk)

### 🔴 The "Front Door on the Street" Problem

> **The Analogy:** Your Mac Mini's front door faces your living room (private). Your VPS's front door faces a busy street (the entire internet). If you forget to lock it, ANYONE in the world can walk in.

**This Is The #1 VPS Risk.**

**What Actually Happens (Step by Step):**
1. You spin up a VPS with a public IP address (e.g., 203.0.113.55)
2. You install OpenClaw and start the gateway
3. You configure `gateway.bind = "lan"` (mistake!) or forget to set it
4. Gateway binds to 0.0.0.0:18789 (all interfaces, including public)
5. You forget to set `gateway.auth.token` (another mistake!)
6. Result: `http://203.0.113.55:18789` is accessible to ANYONE
7. Attackers scanning the internet find your gateway within hours
8. They can send messages, run commands, access all your channels

> **Why This Is Critical:** Unlike your home network (which has a router/firewall), a VPS often has a PUBLIC IP address directly accessible from the internet. Every open port is a door that anyone can try to open.

**How to Check If You're Exposed:**

From your local machine (not the VPS):
```bash
# Try to connect to your VPS gateway port
curl http://YOUR_VPS_IP:18789/health

# If this returns anything, you're exposed!
# It should timeout or refuse connection
```

**How to Fix It:**
```bash
# On your VPS, configure firewall to block external access
sudo ufw deny 18789

# OR configure gateway for loopback only
openclaw config set gateway.bind loopback

# AND always set an auth token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
```

### 🔴 The 1-Click Image Risk

> **The Analogy:** A pre-furnished apartment is convenient, but did the previous tenant leave a spare key under the mat?

**What Actually Happens:**
1. You deploy using a "1-Click Deploy" image (DigitalOcean, etc.)
2. The image may have default configurations that are too permissive
3. Default firewall rules may allow more than you expect
4. Default credentials may exist that you forgot to change

**Risk Factors in Pre-Built Images:**
| Factor | Risk | What to Check |
|--------|------|---------------|
| Default SSH keys | Someone else has access | `cat ~/.ssh/authorized_keys` |
| Default passwords | Brute-forceable | Change immediately |
| Permissive firewall | Ports open | `sudo ufw status` |
| Pre-installed services | Extra attack surface | `systemctl list-units` |

**The Fix (Post 1-Click Deploy):**
```bash
# 1. Change default passwords immediately
passwd

# 2. Remove any pre-configured SSH keys you don't recognize
nano ~/.ssh/authorized_keys

# 3. Review and tighten firewall
sudo ufw status
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw enable

# 4. Run security audit
openclaw security audit --deep
```

> **Note:** The [DigitalOcean 1-Click Deploy](../03-deploy/isolated-vps.md#11-digitalocean-1-click-deploy) for OpenClaw automatically applies security hardening, making it safer than generic images.

### 🟠 The Firewall Gap Window

> **The Analogy:** Between unlocking your front door and setting the alarm, there's a window where you're vulnerable.

**What Actually Happens:**
1. You start the gateway before configuring the firewall
2. For those few seconds/minutes, the port is open to the world
3. Automated scanners may find and probe it
4. Even if you close the port later, the attacker may already have access

**The Fix:**
```bash
# ALWAYS configure firewall BEFORE starting any services
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw enable

# THEN start the gateway
openclaw gateway run
```

---

## B. Multi-Tenant Risks

### 🟡 The "Shared Building" Problem

> **The Analogy:** Your VPS runs on a physical server that hosts many other customers' VPS instances. It's like living in an apartment building - you have your own space, but you share the physical infrastructure.

**What This Means:**
- Your VPS is a "virtual machine" running on a physical server
- Other customers have VPS instances on the same physical hardware
- In rare cases, vulnerabilities allow "neighbors" to peek into your data
- This is called a "side-channel attack" (e.g., Spectre, Meltdown)

**Practical Risk Level:** 🟡 Low-Medium

Most cloud providers patch these quickly, but nation-state attackers or sophisticated hackers could exploit these vulnerabilities.

**What You Can Do:**
- Choose reputable cloud providers (AWS, DigitalOcean, Hetzner)
- Keep your VPS kernel updated: `sudo apt update && sudo apt upgrade`
- Consider dedicated instances for highly sensitive workloads

### 🟡 Environment Variable Exposure

> **The Analogy:** If you leave your passwords on a sticky note on your desk, anyone who sits at your desk can see them.

**What Actually Happens:**
1. You store API keys in environment variables (standard practice)
2. Other processes running as the same user can read your environment
3. If attacker gets ANY code execution as your user, they can read all env vars

**Technical Detail:**
```bash
# Any process as same user can read your environment
cat /proc/$(pgrep -f "openclaw gateway")/environ | tr '\0' '\n'
```

**The Fix:**
- Use a dedicated user for OpenClaw
- Never run untrusted code on your VPS
- Consider using a secrets manager for highly sensitive deployments

---

## C. Credential Storage Vulnerabilities

### 🟠 The "Plaintext Secrets" Problem

> **The Analogy:** Imagine writing all your passwords on sticky notes and keeping them in an unlocked drawer. Anyone with access to the drawer can read them. OpenClaw stores credentials the same way - as plain text files.

**Where Your Secrets Live:**
```
~/.openclaw/
├── openclaw.json          ← May contain API keys
├── credentials/
│   ├── telegram-*.json    ← Bot tokens
│   ├── discord-*.json     ← Bot tokens
│   └── whatsapp/          ← Session credentials
└── agents/*/agent/
    └── auth-profiles.json ← ALL your API keys
```

**There Is NO Encryption:**

These files are protected ONLY by Unix file permissions (0o600/0o700). If an attacker gets `root` access, they can read everything.

**Common Permission Mistakes:**
```bash
# WRONG: This lets everyone read your secrets
chmod -R 755 ~/.openclaw/

# RIGHT: Only you can read/write
chmod 700 ~/.openclaw
chmod 700 ~/.openclaw/credentials
chmod 600 ~/.openclaw/openclaw.json
chmod 600 ~/.openclaw/credentials/*
```

**Verify Permissions:**
```bash
ls -la ~/.openclaw/
# Should show: drwx------ (700 for directories)

ls -la ~/.openclaw/credentials/
# Should show: -rw------- (600 for files)
```

### 🟠 SSH Key Compromise

> **The Analogy:** If someone gets your house key, they can come back anytime they want.

**What Actually Happens:**
1. You use SSH key authentication (good practice!)
2. Your private key is stored on your local machine
3. If your local machine is compromised, attacker gets SSH access to VPS
4. Once on VPS, they have access to all OpenClaw credentials

**The Fix:**
- Use SSH keys with passphrases
- Use `ssh-agent` with timeouts
- Consider hardware security keys (YubiKey) for SSH
- Enable 2FA for VPS console access

---

## D. Container and Isolation Risks

### 🟠 Docker Escape (If Using Containers)

> **The Analogy:** The "secure room" in your apartment has a hidden door to the hallway.

**What Actually Happens:**
1. You run OpenClaw in Docker for isolation (good idea!)
2. But you run with `--privileged` or mount sensitive paths
3. Attacker exploits a vulnerability to escape the container
4. Now they're on the host system with full access

**Dangerous Docker Flags:**
| Flag | Why It's Dangerous |
|------|-------------------|
| `--privileged` | Container has nearly host-level access |
| `-v /:/host` | Container can read/write entire host filesystem |
| `--pid=host` | Container can see all host processes |
| `--network=host` | No network isolation |

**Safe Docker Configuration:**
```bash
# Run with minimal privileges
docker run \
  --security-opt no-new-privileges \
  --cap-drop ALL \
  --read-only \
  -v ~/.openclaw:/data \
  openclaw/gateway
```

---

## E. Persistence Attacks

### 🟠 The "Hidden Backdoor" Problem

> **The Analogy:** After a break-in, the burglar leaves a copy of your key hidden outside, so they can come back whenever they want.

**What Actually Happens (If VPS Is Compromised):**
1. Attacker gets initial access (exposed gateway, SSH compromise, etc.)
2. They install "persistence" mechanisms to maintain access
3. Even if you change passwords, they can still get back in
4. These backdoors are designed to be hidden

**Common Persistence Mechanisms:**
| Mechanism | Where to Check |
|-----------|---------------|
| Cron jobs | `crontab -l` and `/etc/cron.*` |
| SSH authorized_keys | `~/.ssh/authorized_keys` |
| Systemd services | `systemctl list-units` |
| Modified binaries | `debsums -c` (if Debian/Ubuntu) |
| Login scripts | `~/.bashrc`, `~/.profile`, `/etc/profile.d/` |

**If You Suspect Compromise:**
```bash
# Check for suspicious cron jobs
crontab -l
cat /etc/cron.d/*

# Check for unknown SSH keys
cat ~/.ssh/authorized_keys

# Check for suspicious systemd services
systemctl list-units --type=service --state=running

# Check recently modified system files
find /usr/bin -mtime -7 -ls
```

> **Important:** If a VPS is compromised, the safest approach is usually to **destroy it and rebuild from scratch** rather than trying to clean it.

---

## F. 1-Click VPS Specific Risks

### 🟡 DigitalOcean Metadata API

> **The Analogy:** There's a secret phone number in your apartment that, if called, reveals your building's security codes.

**What Actually Happens:**
1. Cloud providers expose a metadata service at 169.254.169.254
2. This service returns information about your VPS
3. If attacker can make requests from your VPS, they might access this
4. Metadata may include API tokens, SSH keys, or other secrets

**The Risk:**

If prompt injection convinces the agent to make a curl request:
```bash
curl http://169.254.169.254/metadata/v1/
```

This could expose cloud provider credentials.

**The Fix:**
- Block metadata access from inside containers
- Use iptables to block 169.254.0.0/16 from non-root users
```bash
sudo iptables -A OUTPUT -m owner ! --uid-owner root -d 169.254.0.0/16 -j DROP
```

---

## G. Worst-Case Damage Assessment

### 💀 How Bad Could It Get?

| Scenario | What Happens | How to Recover |
|----------|--------------|----------------|
| 🔴 **Gateway internet-exposed + no auth** | Anyone finds it, runs commands, joins your bot to botnets | DESTROY the VPS, rotate everything |
| 🔴 **Root compromise** | Attacker has full control, may install persistent backdoors | Forensics (if needed), then destroy VPS and rebuild |
| 🟠 **API key theft** | Attacker uses your keys for their own AI usage (you pay the bill) | Rotate all keys, set up billing alerts |
| 🟠 **SSH key compromise** | Attacker can return anytime | Regenerate SSH keys, check all machines that used them |

**VPS Recovery Is Different:**

> Unlike a Mac Mini where you might try to "clean" the infection, a compromised VPS should usually be **destroyed and rebuilt from scratch**. Attackers often install hidden backdoors that are very hard to find.

### Recovery Checklist

If you suspect VPS compromise:

- [ ] **Take a snapshot for forensics** (optional, if you need evidence)
- [ ] **Destroy/delete the compromised VPS instance**
- [ ] **Rotate ALL API keys and tokens**
  - Anthropic API key
  - OpenAI API key
  - Telegram bot token
  - Discord bot token
  - Any other connected services
- [ ] **Create a NEW VPS instance**
- [ ] **Apply security configuration BEFORE starting services:**
  ```bash
  # Firewall first
  sudo ufw default deny incoming
  sudo ufw allow ssh
  sudo ufw enable

  # Then configure OpenClaw
  openclaw config set gateway.bind loopback
  openclaw config set gateway.auth.token "$(openssl rand -hex 32)"

  # Then start gateway
  openclaw gateway run
  ```
- [ ] **Run security audit:**
  ```bash
  openclaw security audit --deep
  ```

---

## H. Prevention Checklist

### Before You Deploy

- [ ] Choose a reputable cloud provider
- [ ] Use SSH key authentication (not passwords)
- [ ] Set up firewall rules BEFORE starting any services
- [ ] Use a dedicated user for OpenClaw (not root)
- [ ] Consider [DigitalOcean 1-Click Deploy](../03-deploy/isolated-vps.md#11-digitalocean-1-click-deploy) for automatic hardening

### Firewall Configuration

```bash
# Default deny all incoming
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow only SSH
sudo ufw allow ssh

# Enable firewall
sudo ufw enable

# DO NOT allow 18789 from public internet
# Access via SSH tunnel instead
```

### SSH Access Pattern

```bash
# From your local machine, create SSH tunnel
ssh -N -L 18789:127.0.0.1:18789 user@your-vps

# Then access dashboard at
# http://127.0.0.1:18789
```

### After You Deploy

- [ ] Verify gateway binds to localhost only
- [ ] Verify auth token is set
- [ ] Verify firewall blocks port 18789 externally
- [ ] Set up automatic security updates
- [ ] Configure log rotation
- [ ] Set up billing alerts for cloud resources

### Ongoing

- [ ] Keep system updated: `sudo apt update && sudo apt upgrade`
- [ ] Review SSH access logs: `sudo journalctl -u sshd`
- [ ] Run security audit monthly: `openclaw security audit`
- [ ] Monitor for unusual network activity
- [ ] Review cloud provider security advisories

---

## Code References

| Security Control | Source File | Lines |
|------------------|-------------|-------|
| Network binding modes | `src/gateway/net.ts` | 98-122 |
| File permissions (0o700) | `src/config/io.ts` | 477 |
| File permissions (0o600) | `src/config/io.ts` | 489 |
| Gateway auth | `src/gateway/auth.ts` | Authentication logic |
| Docker sandbox | `src/agents/sandbox/docker.ts` | Container isolation |
| Security audit | `src/security/audit.ts` | 323-343 |
