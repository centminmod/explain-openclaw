# Incident Response Playbook

## When to use this playbook

Use this when you suspect or confirm any of the following:
- Unauthorized access to the Gateway (unknown sessions, unexpected commands)
- Leaked API keys or bot tokens (exposed in logs, committed to git, seen in third-party services)
- A compromised plugin or skill (unexpected behavior, data exfiltration)
- Successful prompt injection that caused real-world harm (sent messages, modified files, exfiltrated data)

**Rule of thumb:** If you're asking "should I use this playbook?", the answer is yes.

---

## Phase 1: Contain (stop the bleeding)

**Goal:** Prevent further damage. Do this first, investigate later.

### Stop the Gateway

**Mac (menubar app):**
```bash
# Kill the gateway process
pkill -9 -f openclaw-gateway || true
# Or quit the OpenClaw app from the menu bar
```

**VPS:**
```bash
pkill -9 -f openclaw-gateway || true
# Or if running via systemd:
sudo systemctl stop openclaw-gateway
```

**Moltworker (Cloudflare):**
```bash
# Disable the worker
npx wrangler deployments list --name openclaw-moltworker
npx wrangler rollback --name openclaw-moltworker
```

### Disconnect the network (if physical access)

- Unplug Ethernet / disable Wi-Fi
- This prevents ongoing exfiltration while you assess

### Revoke bot tokens (per channel)

Do this immediately if tokens may be compromised:

| Channel | How to revoke |
|---------|--------------|
| **Telegram** | Message @BotFather -> `/revoke` -> select bot -> generate new token |
| **Discord** | Discord Developer Portal -> Application -> Bot -> Reset Token |
| **Slack** | Slack API -> Your Apps -> OAuth & Permissions -> Rotate tokens |
| **Signal** | Re-register the Signal number (generates new credentials) |
| **WhatsApp** | Meta Business Suite -> API Setup -> Rotate API key |

---

## Phase 2: Assess (understand what happened)

**Goal:** Determine scope of the incident before rotating credentials.

### Check Gateway logs

```bash
# Recent gateway log (default location)
tail -n 500 /tmp/openclaw-gateway.log

# macOS unified log (if using the Mac app)
./scripts/clawlog.sh --tail 500

# Look for: unusual tool calls, unknown session IDs, auth failures
grep -i "auth\|unauthorized\|denied\|error\|exec\|shell" /tmp/openclaw-gateway.log
```

### Check pairing store

Look for unknown paired devices:

```bash
openclaw pairing list telegram
openclaw pairing list discord
openclaw pairing list slack
# ... for each active channel
```

Any pairing you don't recognize? An attacker may have paired a device.

### Check credential file modification times

```bash
ls -la ~/.openclaw/credentials/
# Look for recently modified files you didn't change
stat ~/.openclaw/credentials/*
```

### Check session logs

```bash
ls -lt ~/.openclaw/agents/*/sessions/ | head -20
# Look for sessions you didn't create
```

---

## Phase 3: Rotate (change all compromised credentials)

**Goal:** Ensure the attacker can't use anything they've obtained.

| Credential | Where stored | How to rotate |
|-----------|-------------|---------------|
| **Gateway auth token** | `openclaw config` | `openclaw config set gateway.auth.token "$(openssl rand -hex 32)"` |
| **AI provider API keys** (Anthropic, OpenAI, etc.) | `~/.openclaw/credentials/` or env vars | Regenerate in provider dashboard; update config |
| **Bot tokens** (Telegram, Discord, Slack) | `~/.openclaw/credentials/` | See revocation table in Phase 1; update config with new tokens |
| **SSH keys** (if Gateway host SSH was exposed) | `~/.ssh/` | Generate new keypair; update `authorized_keys` on all hosts |
| **Tailscale auth key** (if used) | Tailscale admin console | Revoke device; generate new auth key |
| **Plugin/skill API keys** | Plugin config or env vars | Rotate any keys accessible to compromised plugins |

After rotating, update the Gateway config:
```bash
openclaw config set <key> "<new-value>"
```

---

## Phase 4: Recover (restore secure operation)

### Run the security audit with auto-fix

```bash
openclaw security audit --fix
```

This tightens common misconfigurations: file permissions, group policies, redaction settings.

### Review and tighten configuration

```bash
# Verify binding
openclaw config get gateway.bind
# Should be "loopback" unless you have a specific reason

# Verify auth is enabled
openclaw config get gateway.auth.mode
# Should be "token" or "password"

# Check DM policy
openclaw config get dm.policy
# Should be "pairing" or "allowlist", not "open"

# Check mDNS
openclaw config get discovery.mdns
# Consider "off" if on a shared network
```

### Revoke unknown pairings

```bash
# Remove any pairings you don't recognize
openclaw pairing revoke telegram <CODE>
openclaw pairing revoke discord <CODE>
```

### Monitor for 24-48 hours

After restoring the Gateway:
- Watch logs for unusual activity
- Check for unexpected tool invocations
- Verify no new unknown pairings appear
- Monitor API provider dashboards for unexpected usage

---

## Phase 5: Learn (prevent recurrence)

1. **Document the incident:** What happened, when, how it was detected, what was the impact
2. **Update your threat model:** Were there assumptions that turned out to be wrong?
3. **Add monitoring:** Can you detect this class of incident earlier next time?
4. **Review tool access:** Did the compromised agent have more access than it needed?
5. **Consider sandbox mode:** Would `sandbox.mode: "all"` have limited the blast radius?

---

## Code references

| Component | Source file | What it does |
|-----------|-----------|-------------|
| Security audit engine | `src/security/audit.ts` | Scans config for misconfigurations |
| Security auto-fix | `src/security/fix.ts` | Applies safe default fixes |
| Pairing store | `src/pairing/pairing-store.ts` | Manages device pairing (list/approve/revoke) |
| Auth middleware | `src/gateway/auth.ts` | Token/password validation, local access detection |
| Config paths | `src/config/paths.ts` | Resolves `~/.openclaw/` directory locations |
| Credential storage | `~/.openclaw/credentials/` | Bot tokens, API keys, OAuth tokens |
| Session logs | `~/.openclaw/agents/<agentId>/sessions/` | Per-agent conversation history |

---

See also:
- [Threat model](../04-privacy-safety/threat-model.md)
- [Hardening checklist](../04-privacy-safety/hardening-checklist.md)
- [Cross-cutting vulnerabilities](./cross-cutting.md)
- [Misconfiguration examples](./misconfiguration-examples.md)
