# Privacy and Security in Clawdbot

This guide explains how Clawdbot protects your privacy and how to configure it for maximum security.

## Privacy fundamentals

### What stays local

By default, everything stays on your machine:

```
~/.clawdbot/
├── sessions/          # Your conversation history
├── credentials/       # API keys and tokens
├── agents/            # Agent state and memory
├── pairing/           # Device pairing info
└── clawdbot.json      # Your configuration
```

- No data is sent to Clawdbot developers
- No telemetry or analytics
- No tracking or monitoring
- No account creation required

### What goes to AI providers

Only your messages are sent to the AI provider you choose:

- Anthropic (Claude) — see [anthropic.com/legal/privacy](https://www.anthropic.com/legal/privacy)
- OpenAI (GPT-4) — see [openai.com/policies/privacy-policy](https://openai.com/policies/privacy-policy)

You choose which provider to use, and you can switch anytime.

## Security features built-in

### 1. Strict config validation

Clawdbot refuses to start if the config is invalid:

- Unknown keys are rejected
- Wrong types cause errors
- Dangerous values are blocked

Run `clawdbot doctor` to check your config.

### 2. Channel allowlists

Control who can message your bot:

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+1234567890", "+0987654321"]  // Only these numbers
    },
    telegram: {
      allowFrom: ["@username", "@anotheruser"]
    }
  }
}
```

If `allowFrom` is not set, anyone can message your bot (not recommended).

### 3. DM pairing policy

Unknown senders must be approved:

```json5
{
  channels: {
    discord: {
      dm: { policy: "pairing" }  // Default: requires approval
    }
  }
}
```

When someone new messages your bot, they get a pairing code. Approve with:

```bash
clawdbot pairing approve discord <code>
```

### 4. Group message controls

Groups are mention-gated by default:

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true }  // Only respond when @mentioned
      }
    }
  }
}
```

### 5. Sandboxing

Run untrusted sessions in Docker:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",    // Sandbox non-main sessions
        scope: "session"     // Isolated per session
      }
    }
  }
}
```

This restricts what the AI can do in group chats.

### 6. Gateway authentication

Require a password for remote connections:

```json5
{
  gateway: {
    auth: {
      mode: "password",
      password: "your-secure-password"
    }
  }
}
```

Or use token-based auth with `CLAWDBOT_GATEWAY_TOKEN`.

## Security audit command

Check your security settings:

```bash
clawdbot doctor
```

This checks for:
- Open DM policies
- Missing allowlists
- Weak authentication
- Unsafe sandbox settings
- Outdated versions

## Best practices for high privacy

### 1. Use a dedicated machine

Run Clawdbot on a device that doesn't do anything else:

- A spare Mac Mini
- A small VPS
- A Raspberry Pi (with some limitations)

### 2. Enable all security features

```json5
{
  gateway: {
    bind: "loopback",              // Don't expose to network
    auth: {
      mode: "password",
      password: "strong-password-here"
    }
  },
  agents: {
    defaults: {
      sandbox: { mode: "all" }     // Maximum sandboxing
    }
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550100"], // Strict allowlist
      dm: { policy: "pairing" }
    }
  }
}
```

### 3. Use secure remote access

Instead of opening ports, use:

- **Tailscale** — Private network between devices
- **SSH tunnel** — Encrypted tunnel to your server
- **VPN** — Your existing VPN connection

### 4. Rotate credentials regularly

```bash
# Generate new API keys from your provider
# Update in clawdbot.json or environment variables
# Restart gateway
clawdbot service restart
```

### 5. Monitor logs

```bash
# View gateway logs
clawdbot logs

# Or check the log file
tail -f ~/.clawdbot/gateway.log
```

## Understanding the risks

### AI provider privacy

Your messages are sent to the AI provider:

- Review their privacy policies
- Consider using Anthropic (stronger privacy commitments)
- For sensitive data, use local models (Ollama)

### Channel app security

Each channel app has different security:

- **WhatsApp** — End-to-end encrypted, but Meta sees metadata
- **Telegram** — Optional E2E for secret chats
- **Signal** — E2E encrypted by default
- **iMessage** — E2E encrypted with Apple

Choose based on your threat model.

### Tool access

The AI can execute commands on your machine:

- Keep sandboxing enabled for non-main sessions
- Review what commands the AI wants to run (verbose mode)
- Use the approval system for sensitive operations

## Local-only mode

For maximum privacy, use local AI models:

```json5
{
  agent: {
    model: "ollama/llama3"  // Or any Ollama model
  }
}
```

This requires:
- Ollama installed locally
- Sufficient RAM/CPU
- Accepting lower model quality

## Credential storage

Credentials are stored in `~/.clawdbot/credentials/`:

- Encrypted at rest on macOS Keychain and Linux secret storage
- Only readable by your user
- Backed up when you backup `~/.clawdbot`

### Setting credentials

```bash
# For Anthropic Claude
clawdbot login --provider anthropic

# For OpenAI
clawdbot login --provider openai
```

Or set via environment variables (recommended for servers).

## Common security mistakes

### Mistake 1: No allowlist

```json5
// BAD: Anyone can message your bot
{
  channels: {
    whatsapp: {}
  }
}

// GOOD: Only approved numbers
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550100"]
    }
  }
}
```

### Mistake 2: Open binding

```json5
// BAD: Exposed to the internet
{
  gateway: {
    bind: "all"
  }
}

// GOOD: Local only
{
  gateway: {
    bind: "loopback"
  }
}
```

### Mistake 3: No authentication

```json5
// BAD: Anyone who can connect has full access
{
  gateway: {
    auth: { mode: "none" }
  }
}

// GOOD: Password required
{
  gateway: {
    auth: {
      mode: "password",
      password: "secure-password"
    }
  }
}
```

### Mistake 4: No sandboxing

```json5
// BAD: Groups can run dangerous commands
{
  agents: {
    defaults: {
      sandbox: { mode: "off" }
    }
  }
}

// GOOD: Groups are sandboxed
{
  agents: {
    defaults: {
      sandbox: { mode: "non-main" }
    }
  }
}
```

## Next steps

- [Install Clawdbot](./installation.md) — Get started securely
- [Configure it](./configuration.md) — Set up security features
- Choose your setup: [Mac Mini](./usage-mac-mini.md) or [VPS](./usage-vps.md)

For detailed security guidance, see the [official security docs](https://docs.clawd.bot/gateway/security).
