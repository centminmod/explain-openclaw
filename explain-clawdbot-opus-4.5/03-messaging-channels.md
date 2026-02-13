# Messaging Channels

Clawdbot supports connecting to many messaging platforms through "channels". This document explains what channels are available and how to set them up.

## What is a Channel?

A **channel** is a connection to a messaging platform. Each channel:
- Monitors for incoming messages
- Delivers AI responses
- Handles platform-specific features (reactions, media, etc.)

---

## Built-in Channels

These channels are included in the core Clawdbot package:

### Telegram

**Best for:** Personal use, groups, bots

```json5
// ~/.clawdbot/clawdbot.json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "accounts": {
        "default": {
          "botToken": "${TELEGRAM_BOT_TOKEN}",
          "allowFrom": ["@yourusername"]
        }
      }
    }
  }
}
```

**Setup Steps:**
1. Message [@BotFather](https://t.me/BotFather) on Telegram
2. Create a new bot with `/newbot`
3. Copy the bot token
4. Set `TELEGRAM_BOT_TOKEN` environment variable

**Features:**
- Bot or user account modes
- Group chat support
- Inline keyboards
- Media handling (photos, voice, documents)

---

### WhatsApp

**Best for:** Family, personal contacts

```json5
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "accounts": {
        "default": {
          "allowFrom": ["+1234567890"]
        }
      }
    }
  }
}
```

**Setup Steps:**
1. Run `clawdbot onboard` and select WhatsApp
2. Scan the QR code with your phone's WhatsApp
3. Keep your phone connected to the internet

**Features:**
- Uses WhatsApp Web protocol (Baileys)
- Personal and group chats
- Media support
- Requires active phone session

**Note:** WhatsApp doesn't have official bot APIs for personal accounts. Clawdbot uses the web protocol.

---

### Discord

**Best for:** Communities, servers, gaming

```json5
{
  "channels": {
    "discord": {
      "enabled": true,
      "accounts": {
        "default": {
          "botToken": "${DISCORD_BOT_TOKEN}",
          "allowFrom": ["user#1234", "server:123456789"]
        }
      }
    }
  }
}
```

**Setup Steps:**
1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Create a new application
3. Go to Bot section, create bot, copy token
4. Enable Message Content Intent
5. Invite bot to your server with appropriate permissions

**Features:**
- Server and DM support
- Slash commands
- Thread support
- Rich embeds

---

### Slack

**Best for:** Work, teams

```json5
{
  "channels": {
    "slack": {
      "enabled": true,
      "accounts": {
        "default": {
          "botToken": "${SLACK_BOT_TOKEN}",
          "appToken": "${SLACK_APP_TOKEN}"
        }
      }
    }
  }
}
```

**Setup Steps:**
1. Create a Slack App at [api.slack.com](https://api.slack.com/apps)
2. Enable Socket Mode
3. Add bot scopes: `chat:write`, `channels:history`, `im:history`
4. Install to workspace
5. Copy Bot Token and App Token

**Features:**
- Workspace integration
- Channel and DM support
- Thread replies
- Slack-specific formatting

---

### Signal

**Best for:** Privacy-focused users

```json5
{
  "channels": {
    "signal": {
      "enabled": true,
      "accounts": {
        "default": {
          "signalCliUrl": "http://localhost:8080",
          "phoneNumber": "+1234567890"
        }
      }
    }
  }
}
```

**Setup Steps:**
1. Install [signal-cli](https://github.com/AsamK/signal-cli)
2. Run signal-cli in REST API mode
3. Register/link a phone number
4. Configure Clawdbot to connect

**Features:**
- End-to-end encrypted
- Individual and group chats
- Requires signal-cli daemon

---

### iMessage

**Best for:** Apple ecosystem users

```json5
{
  "channels": {
    "imessage": {
      "enabled": true
    }
  }
}
```

**Requirements:**
- macOS only
- Signed into iMessage
- Full Disk Access for Terminal/Clawdbot

**Features:**
- Native macOS integration
- Individual and group iMessages
- Uses AppleScript/automation

---

### Google Chat

**Best for:** Google Workspace users

```json5
{
  "channels": {
    "googlechat": {
      "enabled": true,
      "accounts": {
        "default": {
          "credentials": "path/to/service-account.json"
        }
      }
    }
  }
}
```

**Features:**
- Google Workspace integration
- Spaces and DMs
- Requires service account setup

---

## Extension Channels

These channels are available as plugins in `extensions/`:

| Extension | Platform | Notes |
|-----------|----------|-------|
| `bluebubbles` | iMessage | Alternative bridge via BlueBubbles server |
| `matrix` | Matrix/Element | Federated, privacy-focused |
| `msteams` | Microsoft Teams | Enterprise communication |
| `mattermost` | Mattermost | Self-hosted Slack alternative |
| `line` | LINE | Popular in Asia |
| `nextcloud-talk` | Nextcloud | Self-hosted collaboration |
| `nostr` | Nostr | Decentralized protocol |
| `twitch` | Twitch | Live streaming chat |
| `voice-call` | Voice Calls | Audio call handling |
| `zalo` / `zalouser` | Zalo | Popular in Vietnam |
| `tlon` | Tlon/Urbit | Urbit network |

### Installing Extensions

```bash
clawdbot plugins install <plugin-name>
```

Or manually:
```bash
cd extensions/<plugin-name>
npm install --omit=dev
```

---

## Channel Configuration

### Common Options

All channels support these options:

```json5
{
  "channels": {
    "<channel-name>": {
      "enabled": true,           // Enable/disable channel
      "accounts": {
        "default": {
          "allowFrom": [...],    // Allowed users/chats
          "denyFrom": [...],     // Blocked users/chats
          "autoReply": true,     // Enable AI responses
          "prefix": "!ai"        // Require prefix for messages
        }
      }
    }
  }
}
```

### Allowlist Patterns

```json5
{
  "allowFrom": [
    "@username",           // Telegram username
    "+1234567890",        // Phone number (WhatsApp, Signal)
    "user#1234",          // Discord user
    "server:123456789",   // Discord server ID
    "chat:123456789",     // Telegram/Discord chat ID
    "*"                   // Everyone (not recommended)
  ]
}
```

---

## Multi-Channel Setup

You can enable multiple channels simultaneously:

```json5
{
  "channels": {
    "telegram": {
      "enabled": true,
      "accounts": { "default": { /* ... */ } }
    },
    "whatsapp": {
      "enabled": true,
      "accounts": { "default": { /* ... */ } }
    },
    "discord": {
      "enabled": true,
      "accounts": { "default": { /* ... */ } }
    }
  }
}
```

All channels share the same AI agent (unless you configure bindings for specific routing).

---

## Checking Channel Status

```bash
# Check all channel connections
clawdbot channels status

# Probe channels (actually test connections)
clawdbot channels status --probe

# Verbose output
clawdbot channels status --all
```

---

## Troubleshooting

### "Channel not connecting"

1. Check credentials are correct
2. Verify environment variables are set
3. Run `clawdbot doctor` for diagnostics
4. Check logs: `~/.clawdbot/logs/`

### "Messages not being received"

1. Verify allowlist includes the sender
2. Check if `autoReply` is enabled
3. Ensure gateway is running: `clawdbot gateway run`

### "Bot token invalid"

1. Regenerate token from platform's developer portal
2. Make sure there are no extra spaces/quotes
3. Use environment variable substitution: `${VAR_NAME}`

---

*Continue to [04 - AI Providers](./04-ai-providers.md) →*
