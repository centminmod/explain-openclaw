# Quick Reference

This is a cheat sheet for common Clawdbot commands, file locations, and troubleshooting.

## CLI commands

### Core commands

| Command | What it does |
|---------|--------------|
| `clawdbot --version` | Show version |
| `clawdbot --help` | Show help |
| `clawdbot doctor` | Diagnose issues |
| `clawdbot doctor --fix` | Auto-fix issues |

### Gateway commands

| Command | What it does |
|---------|--------------|
| `clawdbot gateway` | Start gateway (foreground) |
| `clawdbot gateway --port 18789` | Start on specific port |
| `clawdbot gateway --verbose` | Start with verbose logs |
| `clawdbot gateway --force` | Start even if lock file exists |
| `clawdbot service start` | Start as background service |
| `clawdbot service stop` | Stop background service |
| `clawdbot service restart` | Restart background service |
| `clawdbot service status` | Check service status |

### Agent commands

| Command | What it does |
|---------|--------------|
| `clawdbot agent --message "Hello"` | Send message to agent |
| `clawdbot agent --session key` | Use specific session |
| `clawdbot agent --thinking high` | Set thinking level |
| `clawdbot agent --json` | Output JSON format |
| `clawdbot agents` | List all agents |

### Message commands

| Command | What it does |
|---------|--------------|
| `clawdbot message send --to +1234567890 --message "Hi"` | Send WhatsApp message |
| `clawdbot message send --channel telegram --to @user --message "Hi"` | Send Telegram message |

### Config commands

| Command | What it does |
|---------|--------------|
| `clawdbot config get` | Show all config |
| `clawdbot config get gateway.port` | Get specific value |
| `clawdbot config set gateway.port 18789` | Set specific value |
| `clawdbot configure` | Open config editor |
| `clawdbot config set key value` | Set a config value |

### Channel commands

| Command | What it does |
|---------|--------------|
| `clawdbot channels status` | Show channel status |
| `clawdbot channels login` | Link channel (WhatsApp) |
| `clawdbot channels logout` | Unlink channel |
| `clawdbot channels list` | List configured channels |
| `clawdbot channels --all status` | Show all channels with probes |

### Session commands

| Command | What it does |
|---------|--------------|
| `clawdbot sessions` | List sessions |
| `clawdbot sessions --all` | List all sessions including inactive |
| `clawdbot sessions reset <key>` | Reset a session |
| `clawdbot sessions prune` | Clean up old sessions |

### Pairing commands

| Command | What it does |
|---------|--------------|
| `clawdbot pairing list` | List paired devices |
| `clawdbot pairing approve <channel> <code>` | Approve pairing request |
| `clawdbot pairing revoke <device-id>` | Revoke device pairing |

### Onboarding

| Command | What it does |
|---------|--------------|
| `clawdbot onboard` | Run setup wizard |
| `clawdbot onboard --install-daemon` | Setup and install service |

## File locations

### Configuration directory

```
~/.clawdbot/
├── clawdbot.json              # Main configuration
├── credentials/               # API keys and tokens (encrypted)
│   ├── anthropic.json
│   ├── openai.json
│   ├── telegram.json
│   └── ...
├── sessions/                  # Conversation history
│   ├── agent:main:cli:main/
│   ├── agent:main:whatsapp:dm:+1234567890/
│   └── ...
├── agents/                    # Agent-specific data
├── pairing/                   # Device pairing info
├── gateway.lock              # Gateway lock file
├── gateway.log              # Gateway logs (if logging to file)
└── broadcasts/              # Broadcast rules
```

### Workspace (agent data)

```
~/clawd/                      # Default workspace
├── skills/                   # Custom skills
│   └── <skill-name>/
│       └── SKILL.md
├── AGENTS.md                 # Agent behavior (customizable)
├── SOUL.md                   # Agent personality (customizable)
├── TOOLS.md                  # Tool descriptions (customizable)
└── USER.md                   # Your info (customizable)
```

## Environment variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `ANTHROPIC_API_KEY` | Claude API key | `sk-ant-xxx...` |
| `OPENAI_API_KEY` | OpenAI API key | `sk-xxx...` |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token | `123456:ABCDEF` |
| `DISCORD_BOT_TOKEN` | Discord bot token | `MTI...` |
| `SLACK_BOT_TOKEN` | Slack bot token | `xoxb-...` |
| `SLACK_APP_TOKEN` | Slack app token | `xapp-...` |
| `GATEWAY_TOKEN` | Gateway auth | Optional |
| `CLAWDBOT_workspace` | Override workspace path | `/custom/path` |

## Chat commands

Send these in any messaging channel:

| Command | What it does |
|---------|--------------|
| `/status` | Show session status |
| `/new` or `/reset` | Reset the session |
| `/compact` | Compress session context |
| `/think <level>` | Set thinking level (off/minimal/low/medium/high) |
| `/verbose on/off` | Toggle verbose mode |
| `/usage <mode>` | Set usage display (off/tokens/full) |
| `/restart` | Restart gateway (owner only) |
| `/activation <mode>` | Set group activation (mention/always) |

## Thinking levels

| Level | When to use |
|-------|-------------|
| `off` | Quick responses, no deep thinking |
| `minimal` | Simple questions, fast replies |
| `low` | Default for most tasks |
| `medium` | Complex tasks, some research |
| `high` | Deep research, multi-step reasoning |
| `xhigh` | Maximum reasoning (GPT-5.2/Codex only) |

## Send policies

| Policy | Behavior |
|--------|----------|
| `auto` | Agent decides when to respond |
| `never` | Never respond automatically |
| `pending` | Hold response for approval |

## Common tasks

### Add a new messaging channel

1. **WhatsApp:**
   ```bash
   clawdbot channels login
   # Scan QR code with phone
   ```

2. **Telegram:**
   - Create bot via @BotFather
   - Get token
   - Set `TELEGRAM_BOT_TOKEN` or add to config
   - Restart gateway

3. **Discord:**
   - Create application at Discord Developer Portal
   - Enable bot with appropriate scopes
   - Get token
   - Invite bot to your server

### Test your setup

```bash
# Check gateway is running
clawdbot status

# Send a test message
clawdbot agent --message "Hello, are you working?"

# Check channel status
clawdbot channels status
```

### View logs

```bash
# systemd service
sudo journalctl -u clawdbot -f

# launchd (macOS)
log show --predicate 'subsystem == "com.clawdbot.gateway"' --last 1h

# Or via clawdbot
clawdbot logs
```

### Backup everything

```bash
# Backup config and sessions
tar -czf clawdbot-backup-$(date +%Y%m%d).tar.gz ~/.clawdbot

# Backup workspace
tar -czf clawd-backup-$(date +%Y%m%d).tar.gz ~/clawd
```

## Troubleshooting

### Gateway won't start

```bash
# Check what's wrong
clawdbot doctor

# Remove lock file if stuck
rm ~/.clawdbot/gateway.lock

# Check port availability
ss -ltnp | grep 18789

# Try starting with verbose output
clawdbot gateway --verbose --force
```

### "Cannot connect to Gateway"

- Verify gateway is running: `clawdbot status`
- Check port matches: `clawdbot config get gateway.port`
- If remote, check tunnel/VPN is active
- Check firewall rules

### API errors

- Verify API key is set: `echo $ANTHROPIC_API_KEY`
- Check key has credits/billing
- Try a simple test message first
- Check provider status page

### Channel not working

```bash
# Check channel status
clawdbot channels status

# Probe channel (connectivity check)
clawdbot channels status --probe

# Re-link if needed
clawdbot channels logout
clawdbot channels login
```

### High memory usage

- Reduce session history
- Enable session pruning in config
- Use a smaller model
- Restart gateway periodically

### Permission errors (macOS)

- System Settings → Privacy → grant permissions
- Restart app after granting
- Check with `clawdbot doctor`

## Performance tips

1. **Use appropriate models** — Don't use Opus for simple tasks
2. **Compact sessions** — Run `/compact` in long conversations
3. **Enable pruning** — Set session pruning in config
4. **Cache effectively** — Let Clawdbot cache responses
5. **Use thinking levels** — Lower levels for simple tasks

## Getting help

### Built-in help

```bash
clawdbot --help
clawdbot <command> --help
```

### Diagnostic

```bash
clawdbot doctor
```

### Community

- **Discord:** https://discord.gg/clawd
- **GitHub:** https://github.com/clawdbot/clawdbot/issues
- **Docs:** https://docs.clawd.bot

## External links

| Resource | URL |
|----------|-----|
| Official docs | https://docs.clawd.bot |
| GitHub repo | https://github.com/clawdbot/clawdbot |
| Getting started | https://docs.clawd.bot/start/getting-started |
| Configuration reference | https://docs.clawd.bot/gateway/configuration |
| Security guide | https://docs.clawd.bot/gateway/security |
| Docker setup | https://docs.clawd.bot/install/docker |
| macOS guide | https://docs.clawd.bot/platforms/macos |
| Troubleshooting | https://docs.clawd.bot/channels/troubleshooting |

---

**End of reference.** For detailed guides, see the other documentation files in this directory.
