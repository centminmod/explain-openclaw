# Optimizations: Overview

## Table of contents (Explain OpenClaw)

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
- Deployment
  - [Standalone Mac mini](../03-deploy/standalone-mac-mini.md)
  - [Isolated VPS](../03-deploy/isolated-vps.md)
  - [Cloudflare Moltworker](../03-deploy/cloudflare-moltworker.md)
- Optimizations
  - [Resource usage analysis (CPU, memory, disk)](./resource-usage.md)
  - [Cost + token optimization](./cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

## What This Section Covers

Running a self-hosted AI assistant incurs costs from model API calls and can consume significant tokens. This section helps you:

1. **Reduce API costs** - Use smarter routing, cheaper models for simple tasks
2. **Optimize token usage** - Manage context windows, prune conversations efficiently
3. **Improve performance** - Balance cost vs. latency for your use case

---

## Quick Decision Matrix

| Goal | Strategy | Primary Resource |
|------|----------|------------------|
| **Minimize cost** | OpenRouter auto routing with `sort: price` | [Cost + token optimization](./cost-token-optimization.md#openrouter-integration) |
| **Minimize latency** | Nitro variants or `sort: throughput` | [Cost + token optimization](./cost-token-optimization.md#model-variants-for-costspeed-trade-offs) |
| **Balance both** | Auto router with default settings | [Cost + token optimization](./cost-token-optimization.md#auto-router-openrouterauto) |
| **Reduce tokens** | Context pruning, efficient prompts | [Cost + token optimization](./cost-token-optimization.md#token-usage-strategies) |
| **Track spending** | OpenRouter dashboard, budget alerts | [Cost + token optimization](./cost-token-optimization.md#monitoring-costs) |
| **Zero cost** | Free model routing | [Cost + token optimization](./cost-token-optimization.md#free-router-openrouterfree) |
| **Per-function optimization** | Right-size models for each task | [Model recommendations by function](./cost-token-optimization.md#model-recommendations-by-function) |
| **Local models** | Zero API cost with Docker/Ollama | [Local models: zero API cost](./cost-token-optimization.md#local-models-zero-api-cost) |

---

## Available Guides

| Guide | Description |
|-------|-------------|
| [Resource usage analysis](./resource-usage.md) | CPU, memory, disk footprint for OpenClaw processes and dependencies |
| [Cost + token optimization](./cost-token-optimization.md) | OpenRouter integration, auto routing, provider configuration, token management |

---

## Why Optimization Matters

Self-hosted doesn't mean free. Your AI assistant's running costs depend on:

1. **Model provider pricing** - Claude Opus costs ~15x more per token than Haiku
2. **Conversation length** - Long sessions accumulate context tokens
3. **Tool usage** - Web fetch, search, and browser tools add tokens
4. **Response verbosity** - Longer replies cost more

Even modest usage (50 messages/day) can cost $50-500+/month depending on model choice. Proper optimization can reduce this by 50-80% without noticeable quality loss for most use cases.

---

Next: [Cost + token optimization](./cost-token-optimization.md)
