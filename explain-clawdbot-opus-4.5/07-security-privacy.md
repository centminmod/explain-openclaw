# Security & Privacy

This document covers Clawdbot's security features and how to configure them for maximum privacy.

## Privacy Model

Clawdbot is designed with privacy in mind:

| Aspect | Default Behavior |
|--------|------------------|
| Data storage | Local only (`~/.clawdbot/`) |
| AI calls | Sent to your chosen provider |
| Message routing | No cloud relay |
| Credentials | Encrypted at rest |
| Access control | Allowlist-based |

---

## Access Control

### Allowlists

The most important security feature - **only users on the allowlist can interact**:

```json5
{
  "channels": {
    "telegram": {
      "accounts": {
        "default": {
          "allowFrom": [
            "@trusted_user",
            "chat:123456789"
          ]
        }
      }
    }
  }
}
```

**Allowlist patterns:**

| Pattern | Matches |
|---------|---------|
| `@username` | Telegram username |
| `+1234567890` | Phone number |
| `chat:123456` | Specific chat ID |
| `server:123456` | Discord server |
| `user#1234` | Discord user |
| `*` | Everyone (dangerous!) |

**Never use `"*"` in production.**

### Denylists

Block specific users/chats:

```json5
{
  "denyFrom": [
    "@blocked_user",
    "chat:spam_group"
  ]
}
```

Denylist takes precedence over allowlist.

---

## Pairing Codes

For secure device pairing:

```bash
# Generate a pairing code
clawdbot pairing create

# Output: ABCD-1234 (valid for 1 hour)
```

Users enter this code to pair their device. After pairing, they're added to the allowlist.

**Security:**
- 8-character codes
- 1-hour TTL
- Single-use
- Rate-limited generation

---

## Gateway Authentication

### No Authentication (Not Recommended)

```json5
{
  "gateway": {
    "auth": {
      "mode": "none"
    }
  }
}
```

Only use for testing on localhost.

### Token Authentication

```json5
{
  "gateway": {
    "auth": {
      "mode": "token",
      "token": "${CLAWDBOT_GATEWAY_TOKEN}"
    }
  }
}
```

Generate a strong token:
```bash
openssl rand -hex 32
```

Clients must send: `Authorization: Bearer <token>`

### Password Authentication

```json5
{
  "gateway": {
    "auth": {
      "mode": "password"
    }
  }
}
```

Set via environment:
```bash
export CLAWDBOT_GATEWAY_PASSWORD="your-strong-password"
```

### Tailscale Authentication (Recommended for Remote Access)

```json5
{
  "gateway": {
    "auth": {
      "mode": "tailscale"
    }
  }
}
```

Uses Tailscale identity - no shared secrets needed. Only machines on your Tailscale network can connect.

---

## File Security

### Permissions

Clawdbot sets restrictive permissions:

| File | Mode | Meaning |
|------|------|---------|
| `clawdbot.json` | `0o600` | Owner read/write only |
| `credentials/` | `0o700` | Owner access only |
| `sessions/` | `0o700` | Owner access only |

### Audit

Run a security audit:

```bash
clawdbot security audit
```

This checks:
- File permissions
- Config validation
- Exposed secrets
- Network exposure

---

## Prompt Injection Protection

Clawdbot includes defenses against prompt injection attacks:

### Content Wrapping

User messages are wrapped to prevent them from being interpreted as system instructions:

```
<user_message>
[User's actual message here]
</user_message>
```

### Detection

Suspicious patterns are flagged and logged.

### Sandboxing

Tool execution is sandboxed to limit potential damage.

---

## Data Storage

### What's Stored Locally

| Data | Location | Purpose |
|------|----------|---------|
| Config | `~/.clawdbot/clawdbot.json` | Settings |
| Credentials | `~/.clawdbot/credentials/` | API keys |
| Sessions | `~/.clawdbot/agents/*/sessions/` | Chat history |
| Platform data | `~/.clawdbot/sessions/` | WhatsApp/Signal sessions |

### What's Sent to AI Providers

- Your messages (after allowlist check)
- Attached media (images, audio, video)
- Conversation context

**Nothing is sent if you use Ollama (local).**

### Clearing Data

```bash
# Clear all sessions
rm -rf ~/.clawdbot/agents/*/sessions/*

# Clear specific channel data
rm -rf ~/.clawdbot/sessions/whatsapp/

# Full reset
rm -rf ~/.clawdbot
```

---

## Network Security

### Loopback Binding

Default: only listen on localhost:

```json5
{
  "gateway": {
    "bind": "loopback"  // 127.0.0.1 only
  }
}
```

### LAN Binding

To allow LAN access:

```json5
{
  "gateway": {
    "bind": "lan"  // 0.0.0.0
  }
}
```

**Always use authentication with LAN binding.**

### TLS/HTTPS

Enable TLS for encrypted connections:

```json5
{
  "gateway": {
    "tls": {
      "enabled": true,
      "cert": "/path/to/cert.pem",
      "key": "/path/to/key.pem"
    }
  }
}
```

---

## Secure Configuration Checklist

### Minimum Security

- [ ] Allowlist configured (no `*`)
- [ ] Gateway auth enabled
- [ ] File permissions correct (`0o600`)

### Recommended Security

- [ ] Token or Tailscale auth
- [ ] Loopback binding
- [ ] Environment variables for secrets
- [ ] Regular security audits

### Maximum Security

- [ ] Ollama for local AI (no cloud)
- [ ] Tailscale-only access
- [ ] TLS enabled
- [ ] Firewall rules
- [ ] Isolated user account
- [ ] Regular data purging

---

## Privacy-First Configuration

For maximum privacy, use local AI only:

```json5
{
  "agent": {
    "model": "ollama:llama3.2",
    "provider": "ollama"
  },

  "gateway": {
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "${CLAWDBOT_GATEWAY_TOKEN}"
    }
  },

  "channels": {
    "telegram": {
      "accounts": {
        "default": {
          "allowFrom": ["@yourusername"]  // Only you
        }
      }
    }
  },

  "logging": {
    "level": "warn"  // Minimal logging
  }
}
```

With this config:
- No data leaves your machine (Ollama runs locally)
- Only you can interact (allowlist)
- Gateway only accessible locally (loopback)
- Minimal logging

---

## Security Audit Command

```bash
clawdbot security audit
```

Output:
```
Security Audit Report
=====================

[✓] Config file permissions: 0600
[✓] Credentials directory: 0700
[✓] Gateway auth: token
[✓] Allowlists configured
[!] Warning: LAN binding enabled
[✓] No hardcoded secrets in config

Overall: GOOD (1 warning)
```

---

## Incident Response

### If Credentials Exposed

1. Immediately rotate affected API keys
2. Check provider dashboards for unusual usage
3. Update config with new keys
4. Review access logs

### If Unauthorized Access

1. Stop gateway: `clawdbot daemon stop`
2. Review session logs
3. Update allowlists
4. Rotate gateway token
5. Check firewall rules

---

*Continue to [08 - Mac Mini Deployment](./08-mac-mini-deployment.md) →*
