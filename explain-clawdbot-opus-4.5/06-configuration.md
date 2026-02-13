# Configuration

This document explains Clawdbot's configuration system in detail.

## Config File Location

**Primary config:** `~/.clawdbot/clawdbot.json`

The config uses [JSON5](https://json5.org/) format, which allows:
- Comments (`// comment` or `/* comment */`)
- Trailing commas
- Unquoted keys
- Single-quoted strings

---

## Directory Structure

```
~/.clawdbot/
├── clawdbot.json           # Main configuration
├── clawdbot.json.bak.1     # Automatic backups (up to 5)
├── clawdbot.json.bak.2
├── credentials/            # AI provider credentials
│   ├── anthropic.json
│   └── openai.json
├── agents/                 # Per-agent data
│   └── default/
│       └── sessions/       # Conversation logs (*.jsonl)
├── sessions/               # Platform session data (WhatsApp, etc.)
└── logs/                   # Log files
```

---

## Configuration Sections

### Agent Settings

```json5
{
  "agent": {
    // AI model to use
    "model": "claude-opus-4-5",

    // AI provider
    "provider": "anthropic",

    // Maximum context tokens
    "maxTokens": 200000,

    // Thinking/reasoning level: "low", "medium", "high"
    "thinking": "medium",

    // System prompt additions
    "systemPrompt": "You are a helpful assistant.",

    // Default agent ID
    "id": "default"
  }
}
```

### Gateway Settings

```json5
{
  "gateway": {
    // Port to listen on
    "port": 18789,

    // Bind address: "loopback" (127.0.0.1) or "lan" (0.0.0.0)
    "bind": "loopback",

    // Gateway mode: "local" or "remote"
    "mode": "local",

    // Authentication
    "auth": {
      "mode": "token",           // "none", "token", "password", "tailscale"
      "token": "${CLAWDBOT_GATEWAY_TOKEN}"
    },

    // TLS settings
    "tls": {
      "enabled": false,
      "cert": "/path/to/cert.pem",
      "key": "/path/to/key.pem"
    }
  }
}
```

### Channel Configuration

```json5
{
  "channels": {
    "telegram": {
      "enabled": true,
      "accounts": {
        "default": {
          // Bot token (use env var)
          "botToken": "${TELEGRAM_BOT_TOKEN}",

          // Allowed users/chats
          "allowFrom": [
            "@username",
            "chat:123456789"
          ],

          // Blocked users/chats
          "denyFrom": [],

          // Enable auto-reply
          "autoReply": true,

          // Require prefix for messages
          "prefix": null,  // or "!ai"

          // Max message length
          "maxLength": 4096
        }
      }
    },

    "whatsapp": {
      "enabled": true,
      "accounts": {
        "default": {
          "allowFrom": ["+1234567890"]
        }
      }
    },

    "discord": {
      "enabled": false,
      "accounts": {
        "default": {
          "botToken": "${DISCORD_BOT_TOKEN}",
          "allowFrom": ["server:123456789"]
        }
      }
    }
  }
}
```

### Bindings (Message Routing)

```json5
{
  "bindings": [
    // Route work chat to work agent
    {
      "match": {
        "channel": "telegram",
        "chat": "chat:work_group_id"
      },
      "agent": "work-assistant"
    },

    // Route personal chats to personal agent
    {
      "match": {
        "channel": "whatsapp"
      },
      "agent": {
        "id": "personal",
        "model": "claude-haiku-3-5"  // Cheaper model
      }
    }
  ]
}
```

### Media Settings

```json5
{
  "media": {
    // Audio transcription
    "audio": {
      "provider": "groq",
      "model": "whisper-large-v3"
    },

    // Image understanding
    "image": {
      "provider": "anthropic",
      "model": "claude-opus-4-5"
    },

    // Video understanding
    "video": {
      "provider": "google",
      "model": "gemini-1.5-flash"
    }
  }
}
```

### Session Settings

```json5
{
  "session": {
    // Session timeout (ms)
    "timeout": 3600000,  // 1 hour

    // Max conversation history
    "maxHistory": 100,

    // Compress old messages
    "compression": true
  }
}
```

### Logging Settings

```json5
{
  "logging": {
    // Log level: "debug", "info", "warn", "error"
    "level": "info",

    // Log to file
    "file": "~/.clawdbot/logs/clawdbot.log",

    // Max log file size
    "maxSize": "10m",

    // Log rotation count
    "maxFiles": 5
  }
}
```

---

## Environment Variable Substitution

Use `${VAR_NAME}` syntax to reference environment variables:

```json5
{
  "channels": {
    "telegram": {
      "accounts": {
        "default": {
          "botToken": "${TELEGRAM_BOT_TOKEN}"
        }
      }
    }
  }
}
```

This allows you to keep secrets out of the config file.

---

## Config Overrides

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `CLAWDBOT_STATE_DIR` | Override state directory (default: `~/.clawdbot`) |
| `CLAWDBOT_CONFIG_PATH` | Override config file path |
| `CLAWDBOT_NIX_MODE=1` | Read-only mode (for NixOS) |
| `CLAWDBOT_GATEWAY_PORT` | Override gateway port |

### CLI Flags

```bash
# Override port
clawdbot gateway run --port 9000

# Override bind address
clawdbot gateway run --bind lan

# Override config file
clawdbot --config /path/to/config.json gateway run
```

---

## Config Commands

### View Configuration

```bash
# View all config
clawdbot config get

# View specific key
clawdbot config get agent.model

# View as JSON
clawdbot config get --json
```

### Set Configuration

```bash
# Set a value
clawdbot config set agent.model gpt-4o

# Set nested value
clawdbot config set channels.telegram.enabled true

# Set boolean
clawdbot config set gateway.tls.enabled false
```

### Reset Configuration

```bash
# Reset to defaults
clawdbot config reset
```

---

## Config Includes

Split large configs into multiple files:

```json5
// ~/.clawdbot/clawdbot.json
{
  "$include": [
    "./channels.json",
    "./agents.json"
  ],

  "gateway": {
    "port": 18789
  }
}
```

```json5
// ~/.clawdbot/channels.json
{
  "channels": {
    "telegram": { /* ... */ }
  }
}
```

---

## Validation

Clawdbot validates your config on startup. If there are errors:

```bash
clawdbot doctor
```

Common issues:
- Invalid JSON5 syntax
- Missing required fields
- Invalid enum values
- Type mismatches

---

## Config Backup

Clawdbot automatically creates backups when config changes:

```
~/.clawdbot/clawdbot.json.bak.1  # Most recent
~/.clawdbot/clawdbot.json.bak.2
~/.clawdbot/clawdbot.json.bak.3
~/.clawdbot/clawdbot.json.bak.4
~/.clawdbot/clawdbot.json.bak.5  # Oldest
```

To restore:
```bash
cp ~/.clawdbot/clawdbot.json.bak.1 ~/.clawdbot/clawdbot.json
```

---

## Hot Reload

The gateway watches for config changes and reloads automatically. No restart needed for most changes.

Changes requiring restart:
- Port change
- TLS settings
- Bind address

---

## Complete Example

```json5
// ~/.clawdbot/clawdbot.json
{
  // AI Configuration
  "agent": {
    "model": "claude-opus-4-5",
    "provider": "anthropic",
    "thinking": "medium",
    "systemPrompt": "You are a helpful, harmless assistant."
  },

  // Gateway Configuration
  "gateway": {
    "port": 18789,
    "bind": "loopback",
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "${CLAWDBOT_GATEWAY_TOKEN}"
    }
  },

  // Channels
  "channels": {
    "telegram": {
      "enabled": true,
      "accounts": {
        "default": {
          "botToken": "${TELEGRAM_BOT_TOKEN}",
          "allowFrom": ["@myusername"],
          "autoReply": true
        }
      }
    },
    "whatsapp": {
      "enabled": true,
      "accounts": {
        "default": {
          "allowFrom": ["+1234567890"]
        }
      }
    }
  },

  // Media Processing
  "media": {
    "audio": {
      "provider": "groq",
      "model": "whisper-large-v3"
    }
  },

  // Logging
  "logging": {
    "level": "info"
  }
}
```

---

*Continue to [07 - Security & Privacy](./07-security-privacy.md) →*
