# Commands Reference

This document provides a quick reference for Clawdbot CLI commands.

## Command Structure

```bash
clawdbot <command> [subcommand] [options]
```

Get help for any command:
```bash
clawdbot <command> --help
```

---

## Essential Commands

### Setup & Maintenance

| Command | Description |
|---------|-------------|
| `clawdbot onboard` | Interactive setup wizard |
| `clawdbot onboard --install-daemon` | Setup + install daemon |
| `clawdbot doctor` | System health check |
| `clawdbot update` | Update to latest version |
| `clawdbot login` | Authenticate AI providers |
| `clawdbot --version` | Show version |

### Gateway Control

| Command | Description |
|---------|-------------|
| `clawdbot gateway run` | Start gateway (foreground) |
| `clawdbot gateway run --port 9000` | Custom port |
| `clawdbot gateway run --bind lan` | Bind to all interfaces |
| `clawdbot daemon start` | Start background daemon |
| `clawdbot daemon stop` | Stop daemon |
| `clawdbot daemon restart` | Restart daemon |
| `clawdbot daemon status` | Check daemon status |

### Configuration

| Command | Description |
|---------|-------------|
| `clawdbot config get` | View all config |
| `clawdbot config get agent.model` | View specific key |
| `clawdbot config get --json` | Output as JSON |
| `clawdbot config set <key> <value>` | Set config value |
| `clawdbot config reset` | Reset to defaults |

### Channel Management

| Command | Description |
|---------|-------------|
| `clawdbot channels status` | Check channel status |
| `clawdbot channels status --probe` | Test connections |
| `clawdbot channels status --all` | Verbose output |

### Pairing & Access

| Command | Description |
|---------|-------------|
| `clawdbot pairing` | List pairing codes |
| `clawdbot pairing create` | Generate new code |
| `clawdbot pairing revoke <code>` | Revoke a code |

---

## Messaging Commands

### Send Messages

```bash
# Send text message
clawdbot message send --to "telegram:@username" --text "Hello!"

# Send to WhatsApp
clawdbot message send --to "whatsapp:+1234567890" --text "Hi there"

# Send with attachment
clawdbot message send --to "telegram:@username" --file /path/to/image.jpg
```

### Run Agent

```bash
# Single message
clawdbot agent --message "What's the weather today?"

# With specific thinking level
clawdbot agent --message "Complex analysis" --thinking high

# Interactive mode
clawdbot agent
```

---

## Advanced Commands

### AI Models

| Command | Description |
|---------|-------------|
| `clawdbot models` | List available models |
| `clawdbot models list` | List with details |
| `clawdbot models pull <model>` | Download Ollama model |

### Plugins

| Command | Description |
|---------|-------------|
| `clawdbot plugins` | List installed plugins |
| `clawdbot plugins install <name>` | Install plugin |
| `clawdbot plugins uninstall <name>` | Remove plugin |
| `clawdbot plugins enable <name>` | Enable plugin |
| `clawdbot plugins disable <name>` | Disable plugin |

### Memory & Sessions

| Command | Description |
|---------|-------------|
| `clawdbot memory` | Memory system status |
| `clawdbot memory search <query>` | Search memories |
| `clawdbot memory clear` | Clear memory |

### Security

| Command | Description |
|---------|-------------|
| `clawdbot security` | Security overview |
| `clawdbot security audit` | Run security audit |

### Sandbox

| Command | Description |
|---------|-------------|
| `clawdbot sandbox` | Sandbox configuration |
| `clawdbot sandbox status` | Check sandbox status |

### Scheduled Tasks

| Command | Description |
|---------|-------------|
| `clawdbot cron` | List cron jobs |
| `clawdbot cron add` | Add scheduled task |
| `clawdbot cron remove <id>` | Remove task |
| `clawdbot cron enable <id>` | Enable task |
| `clawdbot cron disable <id>` | Disable task |

### Browser Automation

| Command | Description |
|---------|-------------|
| `clawdbot browser` | Browser control |
| `clawdbot browser status` | Check browser status |

### Device Management

| Command | Description |
|---------|-------------|
| `clawdbot nodes` | List paired devices |
| `clawdbot nodes list` | List with details |
| `clawdbot nodes remove <id>` | Unpair device |

### Hooks

| Command | Description |
|---------|-------------|
| `clawdbot hooks` | List event hooks |
| `clawdbot hooks add` | Add hook |
| `clawdbot hooks remove <id>` | Remove hook |

---

## Common Options

### Global Options

| Option | Description |
|--------|-------------|
| `--help, -h` | Show help |
| `--version, -v` | Show version |
| `--config <path>` | Use alternate config |
| `--debug` | Enable debug output |
| `--quiet, -q` | Minimal output |

### Gateway Options

| Option | Description |
|--------|-------------|
| `--port <number>` | Port to listen on |
| `--bind <type>` | Bind address (loopback/lan) |
| `--force` | Force start even if running |

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `CLAWDBOT_STATE_DIR` | State directory (default: `~/.clawdbot`) |
| `CLAWDBOT_CONFIG_PATH` | Config file path |
| `CLAWDBOT_GATEWAY_TOKEN` | Gateway auth token |
| `CLAWDBOT_GATEWAY_PASSWORD` | Gateway password |
| `CLAWDBOT_GATEWAY_PORT` | Override port |
| `CLAWDBOT_NIX_MODE` | Read-only config mode |
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `OPENAI_API_KEY` | OpenAI API key |
| `GEMINI_API_KEY` | Google API key |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token |
| `DISCORD_BOT_TOKEN` | Discord bot token |
| `SLACK_BOT_TOKEN` | Slack bot token |
| `SLACK_APP_TOKEN` | Slack app token |

---

## Quick Reference Card

```bash
# ===== SETUP =====
clawdbot onboard --install-daemon  # First-time setup
clawdbot doctor                    # Health check
clawdbot login                     # Auth providers

# ===== RUNNING =====
clawdbot daemon start              # Start daemon
clawdbot daemon stop               # Stop daemon
clawdbot daemon status             # Check status
clawdbot gateway run               # Manual start

# ===== CONFIG =====
clawdbot config get                # View config
clawdbot config set <key> <value>  # Set value

# ===== CHANNELS =====
clawdbot channels status --probe   # Test channels

# ===== MESSAGING =====
clawdbot message send --to "telegram:@user" --text "Hi"
clawdbot agent --message "Question?"

# ===== MAINTENANCE =====
clawdbot update                    # Update
clawdbot security audit            # Security check
```

---

## Getting Help

```bash
# General help
clawdbot --help

# Command-specific help
clawdbot gateway --help
clawdbot config --help
clawdbot channels --help

# Online documentation
# https://docs.clawd.bot
```

---

## See Also

- [Installation Guide](./05-installation-guide.md)
- [Configuration](./06-configuration.md)
- [Security & Privacy](./07-security-privacy.md)
- [Official Documentation](https://docs.clawd.bot)

---

*← Back to [README](./README.md)*
