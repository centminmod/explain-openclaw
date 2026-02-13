# Commands + troubleshooting (quick reference)

## Table of contents (Explain OpenClaw - GLM-5.0 Edition)

- [Home (README)](../README.md)
- Plain English
  - [What is OpenClaw?](../01-plain-english/what-is-clawdbot.md)
  - [Glossary](../01-plain-english/glossary.md)
- Technical
  - [Architecture](../02-technical/architecture.md)
  - [Repo map](../02-technical/repo-map.md)
- Privacy + safety
  - [Threat model](../04-privacy-safety/threat-model.md)
  - [Hardening checklist](../04-privacy-safety/hardening-checklist.md)
  - [High privacy config example](../04-privacy-safety/high-privacy-config.example.json5.md)
- Installation
  - [Install and onboarding](../03-install/install-and-onboard.md)
  - [Build from source](../03-install/from-source.md)
- Deployment
  - [Standalone Mac mini](../05-use-cases/mac-mini-standalone.md)
  - [Isolated VPS](../05-use-cases/vps-isolated.md)

---

This is a copy/paste oriented page. For deeper explanations, use official docs linked below.

---

## Install + onboard

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
```

During onboarding, select **Z.AI** as your provider and **GLM-5.0** as your model.

Docs: https://docs.openclaw.ai/install and https://docs.openclaw.ai/start/wizard

---

## Start/stop Gateway

Foreground:

```bash
openclaw gateway
# alias:
openclaw gateway run
```

Service lifecycle:

```bash
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
openclaw gateway start
```

Docs: https://docs.openclaw.ai/gateway and https://docs.openclaw.ai/cli/gateway

---

## Health and status

```bash
openclaw status
openclaw status --all
openclaw health
```

---

## Security audit

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

`--fix` applies safe fixes and sets recommended defaults.

Docs: https://docs.openclaw.ai/gateway/security

---

## GLM-specific commands

### Provider configuration

```bash
# Set Z.AI as provider
openclaw config set agents.defaults.provider zai

# Set GLM-5.0 as default model
openclaw config set agents.defaults.model glm-5.0

# Set alternative model variants
openclaw config set agents.defaults.model glm-5.0-flash
openclaw config set agents.defaults.model glm-5.0-long

# Enable function calling (disabled by default)
openclaw config set agents.defaults.functionCalling true

# Set temperature for GLM-5.0
openclaw config set agents.defaults.temperature 0.3
```

### Z.AI API key setup

```bash
# Set API key via environment variable (recommended)
export ZAI_API_KEY="your-key-here"

# Verify configuration
openclaw config get agents.defaults
```

---

## Pairing (DM approvals)

```bash
# List pending pairing requests
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>

# For WhatsApp (after QR login)
openclaw pairing approve whatsapp <PHONE_NUMBER>
```

Docs: https://docs.openclaw.ai/start/pairing

---

## Remote access (SSH tunnel)

```bash
ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
```

Then open: http://127.0.0.1:18789/

Docs: https://docs.openclaw.ai/gateway/remote

---

## Common problems

### Control UI says unauthorized

- Run `openclaw dashboard` and open the printed tokenized URL.
- Ensure you are connecting to the correct Gateway instance/profile.
- Verify auth token is set correctly.

### Port already in use (18789)

- Stop supervised service: `openclaw gateway stop` or `launchctl stop com.openclaw.gateway`
- Foreground reclaim: `openclaw gateway --force`

### "Provider not recognized" error for Z.AI

- Ensure you're running the latest OpenClaw version
- Check: `openclaw models list` to see available providers

### Nothing responds in Telegram/WhatsApp

- Check channel status and logs: `openclaw status --all`
- Confirm pairing/allowlists aren't blocking.
- Verify Z.AI API key is valid and set correctly.

### GLM-5.0 specific issues

**High memory usage:** GLM-5.0's ~200K context window can use significant memory on small VPS instances.

```bash
# Monitor resource usage
htop
```

**Slow responses:** GLM-5.0 may take longer for complex reasoning tasks.

Consider using `glm-5.0-flash` for faster responses when speed is critical.

---

## Logs and debugging

```bash
# Stream live logs (Ctrl+C to stop)
openclaw logs --follow

# View last 100 lines
openclaw logs | tail -n 100

# Filter by component
openclaw logs --component gateway
openclaw logs --component agent
openclaw logs --component telegram
```

---

## High-signal official docs

- Getting started: https://docs.openclaw.ai/start/getting-started
- Gateway (runbook): https://docs.openclaw.ai/gateway
- Gateway security: https://docs.openclaw.ai/gateway/security
- Remote access: https://docs.openclaw.ai/gateway/remote
- Help / FAQ: https://docs.openclaw.ai/help/faq
- Troubleshooting: https://docs.openclaw.ai/gateway/troubleshooting
- Z.AI Provider: https://docs.openclaw.ai/providers/zai

---

## State on-disk reference

OpenClaw stores state under `~/.openclaw/` by default:

| Path | Contents |
|------|----------|
| `~/.openclaw/openclaw.json` | Main configuration file |
| `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` | API keys and provider settings |
| `~/.openclaw/agents/<agentId>/sessions/*.jsonl` | Chat transcripts |
| `~/.openclaw/sessions` | Session compaction/management files |

Docs: https://docs.openclaw.ai/gateway/security ("Credential storage map")
