# High Privacy Config Example (GLM-5.0 Edition)

> **Purpose:** This is a reference configuration for a hardened, high-privacy OpenClaw deployment using GLM-5.0. Save this as `~/.openclaw/openclaw.json` after reviewing and customizing for your environment.

This config assumes:
- Gateway bound to loopback only
- Access via SSH tunnel or Tailscale
- Zhipu AI (GLM-5.0) as model provider
- Minimal tool surface
- Pairing required for DM access
- Security audit findings addressed

---

## Configuration File

```json5
{
  "$schema": "https://openclaw.ai/schemas/2024-11/config.json",
  "version": 1,

  "// Gateway Security": {
    "bind": "loopback",
    "port": 18789,
    "auth": {
      "mode": "token",
      "token": "GENERATE_WITH_OPENSSL_COMMAND_BELOW"
    },
    "trustedProxies": [],
    "controlUi": {
      "dangerouslyDisableDeviceAuth": false
    }
  },

  "// Zhipu AI / GLM-5.0 Provider Configuration": {
    "agents": {
      "defaults": {
        "provider": "zhipu",
        "model": "glm-5.0",
        "temperature": 0.3,
        "maxTokens": 200000,
        "authProfile": "default",
        "// Disable function calling by default - enable explicitly if needed": "functionCalling": false,
        "// GLM-5.0 API key - store in environment variable, never in config file": "apiEndpoint": "https://open.bigmodel.cn/api/paas/v4/chat/completions"
      }
    }
  },

  "// Channel Configuration": {
    "// Telegram - example with high security settings": {
      "type": "telegram",
      "enabled": true,
      "dmPolicy": "pairing",
      "groupPolicy": {
        "requireMention": true,
        "allowedGroups": ["your-approved-group-id"]
      },
      "configWrites": false
    },
    "// WhatsApp - example with pairing": {
      "type": "whatsapp",
      "enabled": true,
      "dmPolicy": "pairing"
    },
    "// Discord - example with allowlist": {
      "type": "discord",
      "enabled": true,
      "dmPolicy": "allowlist",
      "allowFrom": ["your-user-id"],
      "configWrites": false
    }
  },

  "// Tool Security - Minimal Profile": {
    "agents": {
      "defaults": {
        "tools": {
          "profile": "minimal"
        },
        "sandbox": {
          "mode": "all",
          "workspaceAccess": "none"
        }
      }
    }
  }
  },

  "// Logging with Redaction": {
    "logging": {
      "redactSensitive": "tools"
    }
  },

  "// Security Audit Settings": {
    "// Run security audit on startup": {
      "security": {
        "auditOnStart": true
      }
    }
  },

  "// Browser Safety - Disabled by default": {
    "browser": {
      "evaluateEnabled": false
    }
  },

  "// Discovery - Disabled on VPS/remote deployments": {
    "discovery": {
      "mdns": {
        "mode": "off"
      }
    }
  },

  "// Memory Management": {
    "memory": {
      "enabled": true,
      "maxMemoryChars": 64000,
      "vector": {
        "enabled": false
      }
    }
  },

  "// Session Management": {
    "sessions": {
      "collapsing": {
        "dms": "main",
        "strategy": "recent"
      },
      "contextPruning": {
        "enabled": true,
        "targetMaxChars": 180000
      }
    }
  },

  "// Plugin Safety - Disabled by default": {
    "plugins": {
      "enabled": false
    }
  },

  "// Command Safety - Disabled by default": {
    "commands": {
      "config": false
    }
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

### 2) Get Zhipu AI API key

1. Visit: https://open.bigmodel.cn/usercenter/apikeys
2. Create a new API key
3. Export as environment variable:

```bash
# Add to ~/.zshrc or ~/.bashrc
export ZHIPU_API_KEY="your-api-key-here"
```

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

### "Provider not recognized" for Zhipu AI

Ensure you're using the latest OpenClaw version with GLM support:

```bash
openclaw --version
npm list -g openclaw@latest
```

---

## Security Reminders

1. **Never commit** `openclaw.json` to git - it contains your API key
2. **Never share** screenshots containing your config or tokens
3. **Rotate tokens** regularly if you suspect exposure
4. **Run security audit** after any config changes

---

## Official Docs

- Gateway Security: https://docs.openclaw.ai/gateway/security
- Configuration: https://docs.openclaw.ai/gateway/configuration
- Tailscale: https://docs.openclaw.ai/gateway/tailscale
