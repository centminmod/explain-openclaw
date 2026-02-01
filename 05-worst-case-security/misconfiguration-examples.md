# The Misconfiguration Hall of Shame

> **In Plain English:** These are real mistakes people make when setting up OpenClaw. Each one seems harmless but can lead to serious security problems. Learn from others' mistakes so you don't repeat them!

**How to Use This Guide:**
1. Read each scenario
2. Check if your setup matches the "WRONG" example
3. If it does, follow the fix instructions immediately
4. Run `openclaw security audit --deep` to check for issues

**Time to read:** 10 minutes
**Difficulty:** Beginner-friendly with copy-paste fixes

---

## 🔴 Mistake #1: Exposing Gateway to Network

### What They Did

```json
{
  "gateway": {
    "bind": "lan",
    "auth": { "mode": "none" }
  }
}
```

### What They Thought

"I'm just testing on my local network, no big deal."

### What Actually Happened

- Gateway became accessible to everyone on the WiFi (roommates, guests, neighbors)
- No authentication required
- Anyone could send messages through their bot, run commands, access all channels

> **The Analogy:** It's like leaving your front door wide open in an apartment building. "It's just my floor" - but everyone on your floor can walk in.

### The Fix

```bash
# Set to loopback (localhost only)
openclaw config set gateway.bind loopback

# Always set an auth token
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"

# Verify
openclaw security audit
```

---

## 🔴 Mistake #2: The "Temporary" Dangerous Flag

### What They Did

```json
{
  "gateway": {
    "controlUi": {
      "dangerouslyDisableDeviceAuth": true,
      "allowInsecureAuth": true
    }
  }
}
```

### What They Thought

"I'll just disable security for testing and re-enable it later."

### What Actually Happened

- They forgot to re-enable it
- Control UI became accessible without proper authentication
- Anyone with the gateway token (or who could guess it) had full admin access

> **The Psychology:** "Temporary" security exceptions have a way of becoming permanent. If you wouldn't leave your house unlocked "just for a minute" while you get groceries, don't disable auth "just for testing."

### The Fix

```bash
# Remove the dangerous flags
openclaw config set gateway.controlUi.dangerouslyDisableDeviceAuth false
openclaw config set gateway.controlUi.allowInsecureAuth false

# Check the security audit (it flags these as CRITICAL)
openclaw security audit
```

### How the Security Audit Catches This

```typescript
// From src/security/audit.ts lines 323-343
// Both flags are flagged as severity: "critical"
```

---

## 🟠 Mistake #3: Cloud-Syncing Credentials

### What They Did

```bash
# Create symlink to Dropbox
ln -s ~/Dropbox/openclaw-backup ~/.openclaw
```

### What They Thought

"Great, now my config is backed up automatically!"

### What Actually Happened

- All credentials (API keys, bot tokens, OAuth secrets) synced to Dropbox
- These are stored in PLAIN TEXT
- Dropbox employees (or anyone who compromises their Dropbox) can read everything
- Even after deleting locally, copies may exist in Dropbox's version history

### Cloud Services That Sync Your Secrets

- ❌ Dropbox
- ❌ Google Drive
- ❌ iCloud Drive
- ❌ OneDrive
- ❌ Box
- ❌ Any folder that syncs to the cloud!

### The Fix

```bash
# If you've already synced:

# 1. Remove the symlink
rm ~/.openclaw

# 2. Move config out of synced folder
mv ~/Dropbox/openclaw-backup ~/.openclaw

# 3. Fix permissions
chmod -R 700 ~/.openclaw

# 4. Rotate ALL credentials (they may already be in cloud provider's systems)
# - Regenerate API keys at console.anthropic.com, platform.openai.com
# - Revoke and recreate bot tokens
# - Log out all WhatsApp Web sessions from phone
```

---

## 🔴 Mistake #4: The Nuclear chmod

### What They Did

```bash
# "Permission denied? Let's fix that..."
sudo chmod -R 777 ~/.openclaw/
```

### What They Thought

"Now I won't get any more permission errors!"

### What Actually Happened

- 777 means: everyone can read, write, and execute
- ANY user on the system can now read all credentials
- ANY process can modify the config
- This is the worst possible permission setting

### Understanding Permissions (Plain English)

```
chmod 777 = Everyone can read, write, execute (BAD!)
chmod 755 = Owner: all, Others: read+execute (BAD for secrets)
chmod 700 = Owner: all, Others: nothing (GOOD for directories)
chmod 600 = Owner: read+write, Others: nothing (GOOD for files)
```

### The Fix

```bash
# Fix directory permissions
chmod 700 ~/.openclaw
chmod 700 ~/.openclaw/credentials

# Fix file permissions
chmod 600 ~/.openclaw/openclaw.json
chmod 600 ~/.openclaw/credentials/*

# Let OpenClaw verify
openclaw security audit --fix
```

---

## 🟠 Mistake #5: Production Keys in Test

### What They Did

```json
{
  "models": {
    "providers": [
      { "id": "anthropic", "apiKey": "sk-ant-api03-PRODUCTION..." }
    ]
  }
}
```

### What They Thought

"I'll just use the same key everywhere, easier to manage."

### What Actually Happened

- Test environment was less secure (sharing with colleagues, looser access)
- Test environment got compromised
- Production API key was leaked
- Attacker used the key to run up $500 in API charges before it was caught

> **The Golden Rule:** Never use production credentials in test, staging, or development environments. Create separate API keys for each environment with appropriate rate limits.

### The Fix

```bash
# Create separate API keys for each environment
# In your Anthropic Console:
# - production-key: Used only in production, with billing alerts
# - test-key: Used in test/dev, with strict rate limits

# Use environment variables, not hardcoded keys
export ANTHROPIC_API_KEY="sk-ant-api03-test-..."
```

---

## 🔴 Mistake #6: The Missing Token

### What They Did

```bash
openclaw config set gateway.auth.mode token
# Forgot the next step!
# openclaw config set gateway.auth.token "..."
```

### What They Thought

"I set up token auth, I'm secure!"

### What Actually Happened

- Token auth mode was enabled
- But no token was actually set
- When token is null/undefined, auth effectively does nothing
- Anyone could connect without authentication

### The Code That Causes This

```typescript
// From src/gateway/auth.ts
const token = authConfig.token ?? env.OPENCLAW_GATEWAY_TOKEN ?? env.CLAWDBOT_GATEWAY_TOKEN;
// If all of these are undefined, token is null, and auth may be bypassed
```

### The Fix

```bash
# Always set both mode AND token
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"

# Verify it's set
openclaw config get gateway.auth.token
# Should show: a long random string (not empty!)

# Double-check with security audit
openclaw security audit
```

---

## 🟠 Mistake #7: Running as Root

### What They Did

```bash
sudo openclaw gateway run
```

### What They Thought

"Root can do anything, so it will definitely work."

### What Actually Happened

- Gateway runs with maximum system privileges
- If exploited, attacker has full root access
- No privilege separation
- All files owned by root, complicating future management

> **The Principle of Least Privilege:** Every process should run with the minimum privileges needed to do its job. OpenClaw doesn't need root.

### The Fix

```bash
# Create a dedicated user
sudo useradd -m -s /bin/bash openclaw

# Give it ownership of the config
sudo chown -R openclaw:openclaw ~/.openclaw

# Run as that user
sudo -u openclaw openclaw gateway run
```

---

## 🟡 Mistake #8: Forgot to Restart After Config Change

### What They Did

```bash
# Changed a security setting
openclaw config set gateway.auth.token "new-secure-token"

# ... continued working without restarting
```

### What They Thought

"The config is saved, so it's applied."

### What Actually Happened

- Config file was updated on disk
- But the running gateway still had the OLD config in memory
- Old (or no) auth token still in effect
- Security change not actually applied

### The Fix

```bash
# After any security-related config change, restart the gateway
openclaw gateway restart

# Verify the new config is active
openclaw status --all
```

---

## 🟡 Mistake #9: Using HTTP Instead of HTTPS

### What They Did

Accessed the Control UI at `http://your-vps:18789` (no TLS)

### What They Thought

"It works, so it's fine."

### What Actually Happened

- All traffic between browser and gateway is unencrypted
- Anyone on the network can see the auth token
- Anyone can intercept and modify commands
- Credentials transmitted in cleartext

### The Fix

```bash
# Option 1: SSH tunnel (recommended)
ssh -N -L 18789:127.0.0.1:18789 user@your-vps
# Then access: http://127.0.0.1:18789

# Option 2: Tailscale (handles TLS automatically)
openclaw config set gateway.bind tailnet

# Option 3: Reverse proxy with TLS (nginx, Caddy)
# Configure your preferred reverse proxy with Let's Encrypt
```

---

## 🟡 Mistake #10: Sharing the Gateway Token

### What They Did

Posted their gateway URL with token in a Discord message:
```
Hey can you help debug? My dashboard is at http://myserver:18789/?token=abc123xyz
```

### What They Thought

"It's just a temporary link for debugging."

### What Actually Happened

- Token is now in Discord's servers (and backups) forever
- Anyone who saw the message can access the gateway
- Even after changing the token, the old one might be cached somewhere

### The Fix

```bash
# 1. Rotate the token immediately
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
openclaw gateway restart

# 2. Never share tokens in:
# - Chat messages
# - GitHub issues
# - Emails
# - Screenshots

# 3. For debugging, use screen sharing or temporary SSH access
```

---

## Quick Security Audit Checklist

Run this checklist to catch common misconfigurations:

```bash
# 1. Check gateway binding
openclaw config get gateway.bind
# Should be: loopback (or tailnet if using Tailscale)

# 2. Check auth mode
openclaw config get gateway.auth.mode
# Should be: token (or password)

# 3. Check auth token is set
openclaw config get gateway.auth.token
# Should be: a long random string (not empty or undefined!)

# 4. Check for dangerous flags
openclaw config get gateway.controlUi
# dangerouslyDisableDeviceAuth should be: false or unset
# allowInsecureAuth should be: false or unset

# 5. Check file permissions
ls -la ~/.openclaw/
# Should show: drwx------ (700 for directories)

ls -la ~/.openclaw/credentials/
# Should show: -rw------- (600 for files)

# 6. Check for cloud sync (macOS)
ls -la ~/.openclaw | grep -E "^l"
# Should show nothing (no symlinks to cloud folders)

# 7. Full security audit
openclaw security audit --deep
```

---

## Summary: The 10 Commandments

1. **Thou shalt bind to loopback** unless you have a specific reason not to
2. **Thou shalt always set an auth token** and never leave it empty
3. **Thou shalt never cloud-sync credentials**
4. **Thou shalt use chmod 600/700**, not 777
5. **Thou shalt separate production and test keys**
6. **Thou shalt restart after config changes**
7. **Thou shalt not run as root**
8. **Thou shalt use TLS** (SSH tunnel or Tailscale)
9. **Thou shalt never share tokens** in messages or screenshots
10. **Thou shalt run security audit** regularly

```bash
# The one command to rule them all
openclaw security audit --deep
```
