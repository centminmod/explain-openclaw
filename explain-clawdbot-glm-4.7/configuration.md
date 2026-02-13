# Configuring Clawdbot

This guide explains how to configure Clawdbot to work the way you want.

## Configuration file location

Clawdbot reads its configuration from:

```
~/.clawdbot/clawdbot.json
```

This file uses **JSON5** format, which means:
- You can use comments (`// this is a comment`)
- You can have trailing commas
- You can omit quotes around keys

## Quick config commands

### View current config

```bash
# View entire config
clawdbot config get

# View specific section
clawdbot config get gateway

# View specific key
clawdbot config get agent.model
```

### Set config values

```bash
# Set the AI model
clawdbot config set agent.model anthropic/claude-opus-4-5

# Set the workspace
clawdbot config set agents.defaults.workspace ~/clawd

# Set WhatsApp allowlist
clawdbot config set channels.whatsapp.allowFrom '["+1234567890"]'
```

### Edit config directly

```bash
# Open in your default editor
clawdbot configure

# Or edit the file directly
nano ~/.clawdbot/clawdbot.json
```

## Main configuration sections

### 1. Agent settings

Control how the AI behaves:

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-5",  // Which AI model to use
    temperature: 0.7,                     // Creativity (0-1)
    maxTokens: 8192                       // Maximum response length
  },
  agents: {
    defaults: {
      workspace: "~/clawd",               // Where agent files live
      systemPrompt: "You are a helpful assistant."
    }
  }
}
```

### 2. Gateway settings

Control the Gateway server:

```json5
{
  gateway: {
    port: 18789,                          // WebSocket port
    bind: "loopback",                     // "loopback" or "all"
    mode: "local",                        // "local" or "remote"
    auth: {
      mode: "none"                        // "none", "password", or "token"
    }
  }
}
```

### 3. Channel settings

Configure each messaging channel:

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+1234567890"],         // Who can DM you
      groups: { "*": { requireMention: true } }  // Group behavior
    },
    telegram: {
      botToken: "123456:ABCDEF",          // Or set TELEGRAM_BOT_TOKEN env var
      allowFrom: ["@username"],           // Who can DM you
      groups: {
        "*": { requireMention: true }     // Group behavior
      }
    },
    discord: {
      token: "your-bot-token",            // Or set DISCORD_BOT_TOKEN env var
      dm: {
        policy: "pairing"                 // "pairing" or "open"
      }
    }
  }
}
```

### 4. Security settings

Control security and permissions:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",                 // "off", "non-main", or "all"
        scope: "session"                  // "global" or "session"
      }
    }
  },
  gateway: {
    auth: {
      mode: "password",                   // Require password for remote
      password: "your-secure-password"
    }
  }
}
```

## Environment variables

Some settings are best set via environment variables:

| Variable | Purpose | Example |
|----------|---------|---------|
| `ANTHROPIC_API_KEY` | Claude API key | `sk-ant-...` |
| `OPENAI_API_KEY` | OpenAI API key | `sk-...` |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token | `123456:ABCDEF` |
| `DISCORD_BOT_TOKEN` | Discord bot token | `your-token` |
| `SLACK_BOT_TOKEN` | Slack bot token | `xoxb-...` |
| `SLACK_APP_TOKEN` | Slack app token | `xapp-...` |
| `GATEWAY_TOKEN` | Gateway auth token | Optional |

Set them in your shell profile (`~/.zshrc`, `~/.bashrc`):

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export TELEGRAM_BOT_TOKEN="123456:ABCDEF"
```

## Minimal working config

For WhatsApp only with Claude:

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-5"
  },
  channels: {
    whatsapp: {
      allowFrom: ["+1234567890"]
    }
  }
}
```

## Common configuration scenarios

### Scenario 1: Privacy-first setup

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-5"
  },
  agents: {
    defaults: {
      sandbox: {
        mode: "all",
        scope: "session"
      }
    }
  },
  channels: {
    telegram: {
      botToken: "your-token",
      dm: { policy: "pairing" }
    }
  }
}
```

### Scenario 2: Group-friendly setup

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+1234567890"],
      groups: {
        "*": { requireMention: true }    // Only respond when @mentioned
      }
    },
    discord: {
      token: "your-token",
      guilds: {
        "your-guild-id": {
          requireMention: true
        }
      }
    }
  }
}
```

### Scenario 3: Multiple AI providers with failover

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-5"
  },
  agents: {
    defaults: {
      models: [
        { provider: "anthropic", model: "claude-opus-4-5" },
        { provider: "openai", model: "gpt-4o" }  // Fallback
      ]
    }
  }
}
```

## Config validation

Clawdbot validates your config on startup. If there's an error:

```bash
# Check for issues
clawdbot doctor

# Auto-fix common issues
clawdbot doctor --fix
```

Common validation errors:
- Unknown keys
- Wrong types (string instead of number, etc.)
- Invalid values (port out of range, etc.)

## Advanced: Config includes

Split your config into multiple files:

```json5
// ~/.clawdbot/clawdbot.json
{
  gateway: { port: 18789 },
  agents: { "$include": "./agents.json5" },
  channels: { "$include": "./channels.json5" }
}
```

```json5
// ~/.clawdbot/agents.json5
{
  defaults: { workspace: "~/clawd" },
  list: [
    { id: "main", workspace: "~/clawd" }
  ]
}
```

## Next steps

After configuring:

- Learn about [privacy/security](./privacy-security.md)
- Set up on [Mac Mini](./usage-mac-mini.md) or [VPS](./usage-vps.md)
- See the [reference](./reference.md) for all commands

For complete configuration reference, see the [official docs](https://docs.clawd.bot/gateway/configuration).
