# What is Clawdbot?

## Plain English Explanation

Imagine you want to talk to an AI assistant like ChatGPT or Claude, but instead of opening a website, you just send a message on WhatsApp, Telegram, or Discord - the same apps you already use to chat with friends and family.

**Clawdbot makes this possible.**

It's software that runs on your own computer (or server) and acts as a translator between:
- **Your messaging apps** (WhatsApp, Telegram, Discord, Slack, Signal, iMessage)
- **AI models** (Claude, GPT-4, Gemini, or local models like Llama)

### The "Universal Translator" Analogy

Think of Clawdbot like a universal translator at a conference:

```
You speak English → Translator → French speaker understands

You send WhatsApp msg → Clawdbot → AI processes and responds
```

The AI doesn't natively "speak WhatsApp" - Clawdbot handles that translation.

---

## Why Self-Hosted Matters

When you use ChatGPT or Claude directly through their websites:
- Your conversations go through their servers
- They may store or train on your data
- You're dependent on their uptime and policies

When you use Clawdbot:
- **You control the hardware** - runs on your Mac, PC, or server
- **You choose the AI** - use any provider, even fully local models
- **Your data stays yours** - messages processed on your machine

---

## What Can Clawdbot Do?

### Basic Capabilities

| Feature | Description |
|---------|-------------|
| **Chat** | Answer questions, have conversations |
| **Multi-platform** | Access from WhatsApp, Telegram, Discord, Slack, Signal, iMessage |
| **Media Processing** | Understand images, transcribe audio, analyze video |
| **Web Browsing** | Fetch and summarize web pages |
| **File Handling** | Read and analyze documents |

### Advanced Features

| Feature | Description |
|---------|-------------|
| **Scheduled Tasks** | Run AI tasks on a schedule (cron) |
| **Memory** | Remember context across conversations |
| **Multi-user** | Different people can use it (with allowlists) |
| **Tool Use** | Execute code, run commands, browse the web |
| **Extensions** | Add support for more platforms (Matrix, MS Teams, etc.) |

---

## Real-World Examples

### Example 1: Personal Assistant

You're shopping and want to check if a product is good:
1. Take a photo of the product
2. Send it to your AI on WhatsApp: "Is this a good deal?"
3. Get an instant analysis with reviews and price comparisons

### Example 2: Family Tech Support

Your parents need help but can't install new apps:
1. They message you on Telegram (which they already use)
2. You've set up Clawdbot to respond in that chat
3. They get AI help without learning anything new

### Example 3: Work Research

You need to research a topic while commuting:
1. Send a voice message on Signal: "Summarize the latest news about electric vehicles"
2. Get a text response you can read later
3. All without typing or opening a browser

---

## How It Differs From ChatGPT/Claude Apps

| Aspect | ChatGPT/Claude Apps | Clawdbot |
|--------|---------------------|----------|
| Access | Dedicated app or website | Any messaging app |
| Data Location | Their servers | Your hardware |
| AI Choice | Their models only | Any AI provider |
| Customization | Limited | Full control |
| Cost | Subscription | Pay only for AI API usage |
| Privacy | Their policies | Your policies |

---

## Key Terminology

| Term | Meaning |
|------|---------|
| **Gateway** | The server component that runs on your machine |
| **Channel** | A messaging platform (Telegram, WhatsApp, etc.) |
| **Agent** | The AI "brain" that processes messages |
| **Provider** | The AI service (Anthropic, OpenAI, Google, Ollama) |
| **Allowlist** | List of users permitted to interact |

---

## Is Clawdbot For Me?

**Yes, if you:**
- Value privacy and data control
- Want AI access from your existing chat apps
- Like self-hosting and tinkering
- Want to share AI with family on their preferred platforms
- Need 24/7 AI access without browser tabs

**Maybe not, if you:**
- Prefer zero setup (just use ChatGPT website)
- Don't want to manage a server
- Only need occasional AI access
- Are uncomfortable with command-line tools

---

## Next Steps

Ready to learn more?
- [Architecture Overview](./02-architecture-overview.md) - How it works technically
- [Installation Guide](./05-installation-guide.md) - Get started now

---

*Continue to [02 - Architecture Overview](./02-architecture-overview.md) →*
