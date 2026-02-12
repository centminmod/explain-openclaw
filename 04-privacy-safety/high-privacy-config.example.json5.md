# High Privacy Config Example

## Table of contents (Explain OpenClaw)

- [Home (README)](../README.md)
- Plain English
  - [What is OpenClaw?](../01-plain-english/what-is-clawdbot.md)
  - [Glossary](../01-plain-english/glossary.md)
  - [CLI commands](../01-plain-english/cli-commands.md)
- Technical
  - [Architecture](../02-technical/architecture.md)
  - [Repo map](../02-technical/repo-map.md)
- Privacy + safety
  - [Threat model](./threat-model.md)
  - [Hardening checklist](./hardening-checklist.md)
  - [High privacy config example](./high-privacy-config.example.json5.md)
- Deployment
  - [Standalone Mac mini](../03-deploy/standalone-mac-mini.md)
  - [Isolated VPS](../03-deploy/isolated-vps.md)
  - [Cloudflare Moltworker](../03-deploy/cloudflare-moltworker.md)
- Optimizations
  - [Cost + token optimization](../06-optimizations/cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

> **Purpose:** This is a reference configuration for a hardened, high-privacy OpenClaw deployment. Save this as `~/.openclaw/openclaw.json` after reviewing and customizing for your environment.

This config assumes:
- Gateway bound to loopback only
- Access via SSH tunnel or Tailscale
- Your chosen model provider configured via `openclaw onboard`
- Minimal tool surface
- Pairing required for DM access

> **Note:** Configure your model provider credentials using `openclaw onboard --install-daemon`. The onboarding wizard handles API key storage securely. Do not put API keys directly in this config file.

---

## Configuration File

Every option below has been verified against OpenClaw's config type definitions in `src/config/types.*.ts`.

```json5
{
  "$schema": "https://openclaw.ai/schemas/2024-11/config.json",
  "version": 1,

  // --- Gateway Security ---
  "gateway": {
    "bind": "loopback",          // Listen on 127.0.0.1 only
    "port": 18789,               // Default port
    "auth": {
      "mode": "token",
      "token": "GENERATE_WITH_OPENSSL_COMMAND_BELOW"
    },
    "trustedProxies": [],        // Empty = no reverse proxy
    "controlUi": {
      "dangerouslyDisableDeviceAuth": false  // Keep device auth enabled
    }
  },

  // --- Channel Configuration ---
  // Configure per-channel via openclaw onboard or openclaw config set
  // Examples shown as comments:

  // Telegram - pairing mode (recommended)
  // "channels": [{
  //   "type": "telegram",
  //   "enabled": true,
  //   "dmPolicy": "pairing",
  //   "groupPolicy": {
  //     "requireMention": true,
  //     "allowedGroups": ["your-approved-group-id"]
  //   },
  //   "configWrites": false
  // }]

  // WhatsApp - pairing mode
  // "channels": [{
  //   "type": "whatsapp",
  //   "enabled": true,
  //   "dmPolicy": "pairing"
  // }]

  // Discord - allowlist mode
  // "channels": [{
  //   "type": "discord",
  //   "enabled": true,
  //   "dmPolicy": "allowlist",
  //   "allowFrom": ["your-user-id"],
  //   "configWrites": false
  // }]

  // --- Agent Defaults: Tool Security ---
  "agents": {
    "defaults": {
      "tools": {
        "profile": "minimal"     // Read-only status, no tool access
      },
      "sandbox": {
        "mode": "all",           // Sandbox all non-main sessions
        "workspaceAccess": "none" // No workspace filesystem access
      }
    }
  },

  // --- Logging with Redaction ---
  "logging": {
    "redactSensitive": "tools"   // Redact tool input/output from logs
  },

  // --- Browser Safety ---
  "browser": {
    "evaluateEnabled": false     // Disable browser JS evaluation
  },

  // --- Discovery ---
  "discovery": {
    "mdns": {
      "mode": "off"              // Don't advertise on local network
    }
  },

  // --- Plugin Safety ---
  "plugins": {
    "enabled": false             // No plugins until explicitly needed
  },

  // --- Command Safety ---
  "commands": {
    "config": false              // Disable /config set chat command
  }
}
```

---

## Key Security Settings Explained

### Gateway Binding (`gateway.bind`)

| Value | Meaning | Risk Level |
|-------|----------|------------|
| `"loopback"` | Listen on 127.0.0.1 only | Lowest - no external access |
| `"lan"` | Listen on all LAN interfaces | Medium - requires auth |
| `"tailnet"` | Bind to Tailscale IP only | Low - tailnet-only |

Source: `src/config/types.gateway.ts`

### DM Policy (`dmPolicy`)

| Value | Meaning |
|-------|----------|
| `"pairing"` | Unknown senders get a pairing code; ignored until approved |
| `"allowlist"` | Only listed IDs can DM; others blocked |
| `"open"` | Anyone can DM (NOT recommended for bots with tools) |

### Group Policy (`groupPolicy`)

- `requireMention: true` - Bot only responds when mentioned
- `allowedGroups: ["id1", "id2"]` - Only respond in these groups
- `requireCommandPrefix: "!"` - Only respond to commands starting with !

### Tool Profiles (`tools.profile`)

| Profile | Description |
|---------|-------------|
| `"minimal"` | Read-only status - no tool access |
| `"coding"` | Filesystem, runtime, sessions, memory, images |
| `"messaging"` | Messaging group, session management |
| `null` (empty) | All tools available |

Source: `src/agents/tool-policy.ts:63-80`

---

## Setup Instructions

### 1) Generate a secure gateway token

```bash
# Generate a random 64-character hex token
export GATEWAY_AUTH_TOKEN="$(openssl rand -hex 32)"

# Verify it's set
echo $GATEWAY_AUTH_TOKEN
```

**Replace `GENERATE_WITH_OPENSSL_COMMAND_BELOW` with your actual token.**

### 2) Configure your model provider

Use the onboarding wizard to set up your chosen provider securely:

```bash
openclaw onboard --install-daemon
```

The wizard handles:
- Provider selection (Anthropic, OpenAI, Zhipu AI, etc.)
- API key storage (environment variable or keychain)
- Default model selection
- Background service installation

### 3) Apply configuration

```bash
# Copy this config to your OpenClaw directory
cp high-privacy-config.example.json5 ~/.openclaw/openclaw.json

# Edit to add your specific values
nano ~/.openclaw/openclaw.json
```

### 4) Restart Gateway

```bash
openclaw gateway restart
```

### 5) Verify setup

```bash
# Check Gateway status
openclaw gateway status

# Run security audit
openclaw security audit --deep

# Check health
openclaw health
```

---

## Channel-Specific Notes

### Telegram

1. Create a bot via @BotFather:
   - Send `/newbot`
   - Follow prompts to set name and description
   - Copy the bot token

2. Add your user ID to `allowFrom` array
   - Find your Telegram user ID: message `@userinfobot` on Telegram
   - Add numeric ID to config

3. For pairing, after receiving a code:
   ```bash
   openclaw pairing approve telegram <CODE>
   ```

### WhatsApp

1. Run `openclaw channels login`
2. Scan QR code with your phone
3. Approve paired devices as needed

---

## Remote Access Setup

### SSH Tunnel (Universal)

```bash
# From your local machine
ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
```

Then open: http://127.0.0.1:18789/

### Tailscale Serve (Recommended)

1. Install Tailscale on both Gateway host and your local machine
2. On Gateway host:
   ```bash
   sudo tailscale serve --bg --https=443 127.0.0.1:18789
   ```
3. Configure OpenClaw to accept Tailscale identity:
   ```json5
   "gateway": {
     "auth": {
       "allowTailscale": true
     }
   }
   ```
4. Access from your local machine: https://<machine-name>.<tailnet>.ts.net/

---

## Troubleshooting

### Gateway won't start

```bash
# Validate configuration syntax
openclaw config validate

# Check logs
openclaw logs --follow
```

### Auth issues after config changes

```bash
# Get a tokenized URL
openclaw dashboard

# Run security audit to detect misconfigurations
openclaw security audit --deep
```

---

## Security Reminders

1. **Never commit** `openclaw.json` to git - it may contain tokens
2. **Never share** screenshots containing your config or tokens
3. **Rotate tokens** regularly if you suspect exposure
4. **Run security audit** after any config changes: `openclaw security audit --fix`
5. **Back up known-good config** before making changes (OpenClaw keeps 5 rotating `.bak` files automatically)

---

## Official Docs

- Gateway Security: https://docs.openclaw.ai/gateway/security
- Configuration: https://docs.openclaw.ai/gateway/configuration
- Tailscale: https://docs.openclaw.ai/gateway/tailscale
- Pairing: https://docs.openclaw.ai/start/pairing
