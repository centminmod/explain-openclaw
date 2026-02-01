# Configuration Reference

Clawdbot uses a hierarchical configuration system. It looks for `~/.clawdbot/clawdbot.json` and overrides it with environment variables.

## Basic Structure (`clawdbot.json`)

```json
{
  "agent": {
    "model": "anthropic/claude-3-5-sonnet-20241022",
    "temperature": 0.7
  },
  "gateway": {
    "port": 18789,
    "bind": "loopback"
  },
  "channels": {
    "whatsapp": { "enabled": true },
    "telegram": { "enabled": false }
  },
  "security": {
    "dmPolicy": "pairing"
  }
}
```

## Environment Variables (`.env`)
You can override any setting using dot-notation syntax with `CLAWDBOT_`.

```bash
# Override the model
CLAWDBOT_AGENT_MODEL="openai/gpt-4-turbo"

# Set Telegram Token
CLAWDBOT_CHANNELS_TELEGRAM_BOTTOKEN="123456:ABC-DEF..."
```

## Critical Security Settings

### 1. DM Policy
Controls how the bot handles new users in DMs.
*   `"pairing"` (Default): Users must be manually approved via CLI.
*   `"open"`: Anyone can message the bot (RISKY - use with allowlists).

```json
"security": {
  "dmPolicy": "pairing"
}
```

### 2. Allow/Deny Lists
Explicitly allow specific phone numbers or usernames.

```json
"channels": {
  "whatsapp": {
    "allowFrom": ["+1234567890"],
    "denyFrom": ["+1987654321"]
  }
}
```

### 3. Sandbox Configuration
Force code execution into Docker.

```json
"agents": {
  "defaults": {
    "sandbox": {
      "mode": "all", // Options: "none", "non-main", "all"
      "image": "node:22-slim" // Docker image to use
    }
  }
}
```

## Troubleshooting
If configuration seems to be ignored:
1.  Run `clawdbot config` to see the *resolved* configuration (merged from file + env).
2.  Check for `.env` files in your current directory which might be overriding your global config.
