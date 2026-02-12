# Commands + troubleshooting (quick reference)

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
  - [Threat model](../04-privacy-safety/threat-model.md)
  - [Hardening checklist](../04-privacy-safety/hardening-checklist.md)
- Deployment
  - [Standalone Mac mini](../03-deploy/standalone-mac-mini.md)
  - [Isolated VPS](../03-deploy/isolated-vps.md)
  - [Cloudflare Moltworker](../03-deploy/cloudflare-moltworker.md)
- Optimizations
  - [Cost + token optimization](../06-optimizations/cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](./commands-and-troubleshooting.md)

---

This is a copy/paste oriented page. For deeper explanations, use the official docs linked below.

---

## Install + onboard

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
```

Docs: https://docs.openclaw.ai/install and https://docs.openclaw.ai/start/wizard

---

## Non-interactive custom provider onboarding

```bash
export CUSTOM_API_KEY="your-api-key-here"
openclaw onboard --non-interactive --install-daemon \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "my-model" \
  --custom-compatibility openai
```

Flags: `--custom-base-url` (required), `--custom-model-id` (required), `--custom-api-key` (optional, env var preferred), `--custom-provider-id` (optional), `--custom-compatibility openai|anthropic` (optional).

Auto-inference: providing any `--custom-*` flag auto-sets `--auth-choice custom-api-key`.

Docs: https://docs.openclaw.ai/start/wizard-cli-automation and https://docs.openclaw.ai/cli/onboard

---

## Start/stop the Gateway

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

Docs: https://docs.openclaw.ai/gateway/security

---

## Pairing (DM approvals)

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Docs: https://docs.openclaw.ai/start/pairing

---

## Remote access (SSH tunnel)

```bash
ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
```

Docs: https://docs.openclaw.ai/gateway/remote

---

## Common problems

### Control UI says unauthorized
- Run `openclaw dashboard` and open the printed tokenized URL.
- Ensure you are connecting to the correct Gateway instance/profile.

Docs: https://docs.openclaw.ai/help/faq

### Port already in use (18789)
- Stop the supervised service (`openclaw gateway stop`) or choose another port.
- Foreground reclaim: `openclaw gateway --force`.

Docs: https://docs.openclaw.ai/help/faq

### Nothing responds in Telegram/WhatsApp
- Check channel status and logs.
- Confirm pairing/allowlists aren’t blocking.
- Confirm model auth is present on the **gateway host**.

Docs: https://docs.openclaw.ai/help/faq and https://docs.openclaw.ai/channels/troubleshooting

---

## High-signal official docs

- Getting started: https://docs.openclaw.ai/start/getting-started
- Gateway runbook: https://docs.openclaw.ai/gateway
- Security: https://docs.openclaw.ai/gateway/security
- Remote: https://docs.openclaw.ai/gateway/remote
- Help/FAQ: https://docs.openclaw.ai/help/faq
- Troubleshooting: https://docs.openclaw.ai/gateway/troubleshooting
