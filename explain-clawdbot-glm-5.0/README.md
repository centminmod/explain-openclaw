# OpenClaw with GLM-5.0 - Beginner + Technical Guide

> **GLM-5.0 Edition:** This documentation explains OpenClaw specifically for users deploying with Z.AI's GLM-5.0 models. GLM-5.0 brings significant improvements in reasoning, code generation, and tool-use capabilities.
>
> **Opus 4.6 correction:** This doc set originally referenced "Zhipu AI" and `open.bigmodel.cn` throughout. The correct provider is **Z.AI** (`api.z.ai`) with `ZAI_API_KEY`. Zhipu AI (智谱AI) is the parent research company; Z.AI is the international API platform used by OpenClaw, with infrastructure in Singapore. All references corrected below.

**Public repo:** This documentation is also available at [github.com/centminmod/explain-openclaw](https://github.com/centminmod/explain-openclaw)

---

## Table of contents

- [What is OpenClaw? (plain English)](./01-plain-english/what-is-clawdbot.md)
- [Glossary](./01-plain-english/glossary.md)
- [Technical overview](./02-technical/architecture.md)
- [Repo map](./02-technical/repo-map.md)
- Privacy + safety
  - [Threat model](./04-privacy-safety/threat-model.md)
  - [Hardening checklist](./04-privacy-safety/hardening-checklist.md)
  - [High privacy config example](./04-privacy-safety/high-privacy-config.example.json5.md)
- Installation
  - [Install and onboarding](./03-install/install-and-onboard.md)
  - [Build from source](./03-install/from-source.md)
- Deployment
  - [Standalone Mac mini](./05-use-cases/mac-mini-standalone.md)
  - [Isolated VPS](./05-use-cases/vps-isolated.md)

---

## What is GLM-5.0?

**GLM-5.0** (General Language Model 5.0) is Z.AI's latest flagship model series. Compared to GLM-4.7, GLM-5.0 offers:

| Capability | GLM-4.7 | GLM-5.0 | Improvement |
|-----------|---------|---------|-------------|
| Context window | ~128K tokens | ~200K tokens | ~56% larger |
| Reasoning depth | Good | Excellent | Multi-step chain-of-thought |
| Code generation | Strong | Excellent | Better syntax, fewer errors |
| Tool use | Basic | Advanced | Native function calling |
| Multilingual | Good | Excellent | Better cross-lingual understanding |

**What this means for OpenClaw:**

GLM-5.0's enhanced function calling and reasoning capabilities make it particularly well-suited for:

1. **Complex tool orchestration** - The model can plan multi-step operations using tools more effectively
2. **Code-heavy workflows** - Better code generation with fewer syntax errors
3. **Longer conversations** - Larger context window means less frequent context truncation
4. **Multilingual deployments** - Better handling of non-English prompts

**Official GLM documentation:** See [Z.AI provider docs](https://docs.openclaw.ai/providers/zai)

---

## Why use GLM-5.0 with OpenClaw?

OpenClaw supports GLM models through the Z.AI provider integration. Key advantages:

- **Cost-effective** - GLM pricing is competitive with other major model providers
- **High performance** - Strong benchmarks on coding and reasoning tasks
- **Multilingual optimization** - Excellent for Chinese and multilingual deployments
- **Data residency** - Z.AI provides GLM API access with Singapore-based infrastructure
- **Flexible deployment** - Works with all OpenClaw deployment modes (Mac mini, VPS, serverless)

---

## Quick start (recommended reading order)

### 1) Plain English
- [What is OpenClaw?](./01-plain-english/what-is-clawdbot.md)
- [Glossary](./01-plain-english/glossary.md)

### 2) Privacy + safety first (highly recommended)
- [Threat model (beginner-friendly)](./04-privacy-safety/threat-model.md)
- [Hardening checklist (high privacy)](./04-privacy-safety/hardening-checklist.md)
- [High privacy config example](./04-privacy-safety/high-privacy-config.example.json5.md)

### 3) Technical overview (how it works)
- [Architecture (Gateway → channels → agent → tools)](./02-technical/architecture.md)
- [Repo map (where to look in code)](./02-technical/repo-map.md)

### 4) Installation
- [Install and onboarding (with Z.AI setup)](./03-install/install-and-onboard.md)
- [Build from source](./03-install/from-source.md)

### 5) Deployment runbooks
- [Standalone Mac mini (local-first)](./05-use-cases/mac-mini-standalone.md)
- [Isolated VPS (remote + locked down)](./05-use-cases/vps-isolated.md)

---

## Install OpenClaw

Recommended installer:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Alternative:

```bash
npm install -g openclaw@latest
```

### Onboard + install background service

```bash
openclaw onboard --install-daemon
```

During onboarding, select **Z.AI** as your provider when prompted.

---

## Verify installation

```bash
openclaw gateway status
openclaw status
openclaw health
openclaw security audit --deep
```

---

## Official docs (high-signal links)

- Getting started: https://docs.openclaw.ai/start/getting-started
- Gateway (runbook): https://docs.openclaw.ai/gateway
- Gateway security: https://docs.openclaw.ai/gateway/security
- Remote access: https://docs.openclaw.ai/gateway/remote
- Help / FAQ: https://docs.openclaw.ai/help/faq
- Troubleshooting: https://docs.openclaw.ai/gateway/troubleshooting

---

## GLM-5.0 specific considerations

### Model selection

GLM-5.0 offers several model variants. Choose based on your needs:

| Model | Best For | Context | Notes |
|--------|-----------|---------|-------|
| `glm-5.0` | General purpose | ~200K tokens | Main flagship model |
| `glm-5.0-flash` | Speed/cost | ~128K tokens | Faster responses, lower cost |
| `glm-5.0-long` | Long documents | ~1M tokens | Extended context analysis |

### API key setup

You'll need a Z.AI API key. Create one in the Z.AI console.

**Security note:** Treat your Z.AI API key like a password - never commit it to git, share it in screenshots, or log it.

### Performance considerations

- **Streaming** - GLM-5.0 supports streaming responses for real-time output
- **Function calling** - Native support for OpenClaw's tool system
- **Temperature settings** - GLM-5.0 works well with temperatures 0.7-1.0 for creative tasks
- **Rate limits** - Z.AI has generous rate limits; monitor usage if deploying at scale

---

## See also

- [Official OpenClaw Documentation](https://docs.openclaw.ai/)
- [Z.AI Provider Documentation](https://docs.openclaw.ai/providers/zai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
