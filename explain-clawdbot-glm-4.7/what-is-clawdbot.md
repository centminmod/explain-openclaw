# What is Clawdbot?

## The short version

**Clawdbot is your own private AI assistant.**

Think of it as ChatGPT or Claude, but running on your own computer instead of on a company's servers. You talk to it through apps you already use — WhatsApp, Telegram, Slack, Discord, and more — and it keeps all your conversations private on your device.

## The longer explanation

### What problem does Clawdbot solve?

When you use commercial AI services like ChatGPT or Claude:

1. **Your data leaves your device** — Everything you type is sent to servers you don't control
2. **You're tied to their interface** — You have to use their website or app
3. **They might train on your data** — Read privacy policies carefully
4. **You can't customize much** — What they offer is what you get
5. **Service can go down** — If their servers have problems, you're stuck

Clawdbot solves these problems:

1. **Your data stays local** — Everything stored in `~/.clawdbot/` on your machine
2. **Use apps you already have** — WhatsApp, Telegram, Slack, Discord, iMessage, Signal, and more
3. **No training on your data** — No telemetry, no analytics, nothing sent elsewhere
4. **Fully customizable** — Configure behavior, prompts, tools, and integrations
5. **You control availability** — Runs on your hardware, as reliable as you make it

### What makes Clawdbot different?

| Feature | Commercial AI | Clawdbot |
|---------|---------------|----------|
| **Where it runs** | Company cloud servers | Your computer (Mac/Linux/VPS) |
| **Data storage** | Their servers | Your disk (`~/.clawdbot/`) |
| **Privacy** | Depends on their policy | 100% under your control |
| **Interface** | Their website/app | Any messaging app you choose |
| **Customization** | Limited | Extensive (agents, skills, tools) |
| **Cost** | Monthly subscription | Pay only for API usage |
| **Offline capability** | None | Local-only mode possible |
| **Telemetry** | Almost always | None built-in |

### Who is Clawdbot for?

Clawdbot is ideal for:

- **Privacy-conscious users** who want control over their AI conversations
- **Developers** who want to extend and customize their AI assistant
- **Power users** who want AI in their existing workflow (messaging apps)
- **Self-hosting enthusiasts** who prefer running their own services
- **People with sensitive data** who shouldn't paste it into commercial AI

### Who is Clawdbot NOT for?

Clawdbot may not be ideal if:

- You want something that "just works" with zero setup (use ChatGPT instead)
- You don't have a computer that can stay running
- You're uncomfortable with terminal commands
- You don't want to manage API keys and billing
- You need free AI (you still pay Anthropic/OpenAI for API usage)

## What can Clawdbot do?

### Core capabilities

- **Chat with AI** — Have conversations using Claude, GPT-4, or other models
- **Multi-channel inbox** — Use WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, Microsoft Teams, and more
- **Voice interactions** — Talk to Clawdbot on macOS, iOS, and Android
- **Browse the web** — Clawdbot can control a browser to research things for you
- **Read files** — Access documents, PDFs, and other files on your computer
- **Run commands** — Execute terminal commands (with appropriate security controls)
- **Remember context** — Maintains conversation history across sessions
- **Work with images** — Can see and analyze images you send
- **Automate tasks** — Schedule jobs, respond to webhooks, and more

### Example use cases

```
You (in WhatsApp): "Summarize the PDF I just sent"

Clawdbot: [Reads the PDF and provides a summary]

---

You (in Slack): "What's on my calendar today?"

Clawdbot: [Checks your calendar and lists your appointments]

---

You (in Telegram): "Take a screenshot of my desktop and analyze it"

Clawdbot: [Takes screenshot, analyzes contents, describes what it sees]
```

## Communication channels supported

Clawdbot works with many messaging platforms:

| Channel | Notes |
|---------|-------|
| **WhatsApp** | Most popular option; works via web protocol |
| **Telegram** | Fast and reliable; requires bot token |
| **Slack** | Great for work integration |
| **Discord** | Popular with communities |
| **Google Chat** | Integrates with Google Workspace |
| **Signal** | Privacy-focused messaging |
| **iMessage** | macOS only; great for Apple users |
| **Microsoft Teams** | For enterprise users |
| **WebChat** | Built-in web interface |
| **BlueBubbles** | Self-hosted iMessage server |
| **Matrix** | Decentralized messaging |
| **Zalo** | Popular in Vietnam |
| **Line** | Popular in Japan/Taiwan/Thailand |

## AI providers supported

Clawdbot connects to many AI services:

| Provider | Models | Notes |
|----------|--------|-------|
| **Anthropic** | Claude 3.5 Sonnet, Claude Opus 4.5, etc. | Recommended for long context |
| **OpenAI** | GPT-4, GPT-4o, o1, etc. | Works well for most tasks |
| **AWS Bedrock** | Claude, Titan, etc. | Enterprise-friendly |
| **Google** | Gemini models | Available via Bedrock |
| **Ollama** | Local models | Fully offline option |
| **Open Router** | Many models | Aggregator service |

You can configure failover so if one provider has issues, Clawdbot automatically switches to another.

## The philosophy behind Clawdbot

### Local-first

- Your data lives on your device
- No account creation with us
- No cloud services required (unless you want them)

### Privacy-focused

- No telemetry
- No analytics
- No "phone home"
- You can audit everything

### Extensible

- Add custom skills
- Configure multiple agents with different personalities
- Create your own tools and integrations
- Open source with an active community

### Single-user by design

Clawdbot is built for **one person** (you). This is a feature, not a bug:

- Simpler security model
- No sharing concerns
- You control who can message your bot
- Sessions are isolated by conversation

## Is Clawdbot free?

Clawdbot itself is free and open-source (MIT license). However:

- You pay the AI provider (Anthropic, OpenAI, etc.) for API usage
- You need your own hardware to run it on
- Some advanced features may have optional costs

The advantage: you only pay for what you use, and you can switch providers anytime.

## Next steps

Now that you understand what Clawdbot is:

1. Learn [how it works](./how-it-works.md) — the technical architecture explained simply
2. [Install it](./installation.md) — step-by-step setup guide
3. [Configure it](./configuration.md) — make it work your way
4. Choose your setup: [Mac Mini](./usage-mac-mini.md) or [VPS](./usage-vps.md)
