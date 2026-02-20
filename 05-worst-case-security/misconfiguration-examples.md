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

## Table of Contents

1. [Mistake #1: Exposing Gateway to Network](#-mistake-1-exposing-gateway-to-network)
2. [Mistake #2: The "Temporary" Dangerous Flag](#-mistake-2-the-temporary-dangerous-flag)
3. [Mistake #3: Cloud-Syncing Credentials](#-mistake-3-cloud-syncing-credentials)
4. [Mistake #4: The Nuclear chmod](#-mistake-4-the-nuclear-chmod)
5. [Mistake #5: Production Keys in Test](#-mistake-5-production-keys-in-test)
6. [Mistake #6: The Missing Token](#-mistake-6-the-missing-token)
7. [Mistake #7: Running as Root](#-mistake-7-running-as-root)
8. [Mistake #8: Forgot to Restart After Config Change](#-mistake-8-forgot-to-restart-after-config-change)
9. [Mistake #9: Using HTTP Instead of HTTPS](#-mistake-9-using-http-instead-of-https)
10. [Mistake #10: Sharing the Gateway Token](#-mistake-10-sharing-the-gateway-token)
11. [Mistake #11: Asking AI to "Optimize" Your Config](#-mistake-11-asking-ai-to-optimize-your-config)
12. [Mistake #12: AI Set Schema-Valid But Dangerous Values](#-mistake-12-ai-set-schema-valid-but-dangerous-values)
13. [Quick Security Audit Checklist](#quick-security-audit-checklist)
14. [Summary: The 12 Commandments](#summary-the-12-commandments)

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
// From src/security/audit.ts lines 343-363
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

## 🔴 Mistake #11: Asking AI to "Optimize" Your Config

### What They Did

Told their AI assistant: "Optimize my OpenClaw config for performance."

### What They Thought

"The AI knows the config schema, it'll make smart changes."

### What Actually Happened

The AI called the gateway tool's `config.patch` action and:
- Set `tools.exec.security: "full"` (for "faster command execution without approval overhead")
- Disabled TLS verification (for "reduced connection latency")
- Widened `gateway.bind` from `loopback` to `lan` (for "better connectivity")

Each change had a plausible justification in the AI's response. None were what the user wanted. All weakened security. The gateway tool's `config.patch` has **zero permission checks** — no `configWrites` gate, no `commands.config` gate — so these changes went through immediately.

> **The Root Cause:** The AI doesn't understand the security implications of config changes. It optimizes for the goal you stated ("performance") and confidently disables safety mechanisms that it perceives as overhead.

Source: `src/agents/tools/gateway-tool.ts:175-225` — zero permission gating on `config.patch`

### The Fix

```bash
# 1. Remove the gateway tool so AI can't modify config at all
openclaw config set tools.profile coding
# Or: openclaw config set tools.deny '["gateway"]'

# 2. Review and revert the changes
diff ~/.openclaw/openclaw.json.bak ~/.openclaw/openclaw.json

# 3. Restore from backup if needed
cp ~/.openclaw/openclaw.json.bak ~/.openclaw/openclaw.json
openclaw gateway restart

# 4. Always review config changes manually — never let AI modify config unattended
openclaw security audit --deep
```

See: [AI Self-Misconfiguration Guide](./ai-self-misconfiguration.md)

---

## 🟠 Mistake #12: AI Set Schema-Valid But Dangerous Values

### What They Did

Asked their AI: "Make OpenClaw accessible from my phone."

### What They Thought

"The AI will figure out the right way to set up remote access."

### What Actually Happened

The AI set:
```json
{
  "gateway": {
    "bind": "lan",
    "tailscale": { "mode": "funnel" }
  }
}
```

Both values are **perfectly schema-valid** — OpenClaw's Zod schema with `.strict()` mode rejects unknown keys and type errors, but `"lan"` is a legitimate value for `gateway.bind` and `"funnel"` is a legitimate value for `tailscale.mode`. The AI chose the most "accessible" options without understanding that:
- `bind: "lan"` exposes the gateway to everyone on the local network
- `tailscale.mode: "funnel"` exposes the gateway to the **entire internet** through Tailscale's public funnel

The user's phone was now able to connect — along with everyone else.

> **The Core Issue:** Schema validation catches **structural** errors (wrong types, unknown keys) but not **semantic** security errors. The AI can set dangerous values for every known key, and validation will happily accept them.

Source: `src/config/validation.ts:90-133`, `src/config/zod-schema.ts:666`

### The Fix

```bash
# 1. Set safe values
openclaw config set gateway.bind loopback
openclaw config set gateway.tailscale.mode serve

# 2. Use SSH tunnel or Tailscale Serve for phone access instead
# SSH tunnel:
ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
# Tailscale Serve (HTTPS within your tailnet only):
openclaw config set gateway.bind tailnet

# 3. Always run security audit after AI config changes
openclaw security audit --deep
```

See: [AI Self-Misconfiguration Guide](./ai-self-misconfiguration.md) — Part 4: Schema-Valid but Unsafe Values

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

## Summary: The 12 Commandments

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
11. **Thou shalt not let AI modify thy config** — remove the `gateway` tool
12. **Thou shalt not trust schema validation** for security — audit after AI changes

```bash
# The one command to rule them all
openclaw security audit --deep
```

> **See also:** Many misconfigurations become critical when combined with prompt injection. See [Prompt Injection Attacks](./prompt-injection-attacks.md) for 30 real attack examples that exploit the configurations described above. For AI-specific config modification risks, see [AI Self-Misconfiguration](./ai-self-misconfiguration.md).
