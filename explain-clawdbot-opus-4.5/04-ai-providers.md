# AI Providers

Clawdbot supports multiple AI providers, giving you flexibility to choose based on cost, capability, or privacy requirements.

## Supported Providers

### Cloud Providers

| Provider | Models | Best For |
|----------|--------|----------|
| **Anthropic** | Claude Opus 4.5, Sonnet, Haiku | Default, best reasoning |
| **OpenAI** | GPT-4o, GPT-4, GPT-3.5 | Broad compatibility |
| **Google** | Gemini Pro, Gemini Flash | Video understanding |
| **Groq** | Llama, Mixtral | Fast inference |
| **AWS Bedrock** | Claude, Titan, others | Enterprise AWS |
| **MiniMax** | Various | Alternative provider |
| **OpenRouter** | Multiple models | Provider aggregator |

### Local Providers

| Provider | Models | Best For |
|----------|--------|----------|
| **Ollama** | Llama 3.2, Mistral, Phi, etc. | Privacy, offline use |

---

## Default Configuration

Out of the box, Clawdbot uses:

```json5
{
  "agent": {
    "model": "claude-opus-4-5",   // Model name
    "provider": "anthropic",      // Provider
    "maxTokens": 200000           // Context window
  }
}
```

---

## Provider Setup

### Anthropic (Default)

1. Get API key from [console.anthropic.com](https://console.anthropic.com)
2. Set environment variable:
   ```bash
   export ANTHROPIC_API_KEY="sk-ant-..."
   ```

**Or** use the login command:
```bash
clawdbot login
# Select Anthropic, paste API key
```

**Available Models:**
- `claude-opus-4-5` - Most capable (default)
- `claude-sonnet-4` - Balanced speed/quality
- `claude-haiku-3-5` - Fastest, cheapest

---

### OpenAI

1. Get API key from [platform.openai.com](https://platform.openai.com)
2. Set environment variable:
   ```bash
   export OPENAI_API_KEY="sk-..."
   ```

**Configuration:**
```json5
{
  "agent": {
    "model": "gpt-4o",
    "provider": "openai"
  }
}
```

**Available Models:**
- `gpt-4o` - Omni model (vision, audio)
- `gpt-4-turbo` - Previous generation
- `gpt-3.5-turbo` - Fast, cheap

---

### Google (Gemini)

1. Get API key from [aistudio.google.com](https://aistudio.google.com)
2. Set environment variable:
   ```bash
   export GEMINI_API_KEY="..."
   ```

**Configuration:**
```json5
{
  "agent": {
    "model": "gemini-2.0-flash",
    "provider": "google"
  }
}
```

**Available Models:**
- `gemini-2.0-flash` - Fast, multimodal
- `gemini-1.5-pro` - Longer context
- `gemini-1.5-flash` - Balance

**Special Feature:** Best video understanding support

---

### Ollama (Local)

**Best for privacy - no data leaves your machine.**

1. Install Ollama: [ollama.com](https://ollama.com)
2. Pull a model:
   ```bash
   ollama pull llama3.2
   ```

**Configuration:**
```json5
{
  "agent": {
    "model": "ollama:llama3.2",
    "provider": "ollama"
  }
}
```

**Available Models (examples):**
- `llama3.2` - Meta's latest
- `mistral` - Mistral AI
- `phi3` - Microsoft
- `gemma2` - Google
- `codellama` - Code-focused

**Note:** Local models require significant RAM (8GB+ for smaller models, 32GB+ for larger ones).

---

### Groq

1. Get API key from [console.groq.com](https://console.groq.com)
2. Set environment variable:
   ```bash
   export GROQ_API_KEY="..."
   ```

**Configuration:**
```json5
{
  "agent": {
    "model": "llama-3.1-70b-versatile",
    "provider": "groq"
  }
}
```

**Special Feature:** Extremely fast inference (LPU hardware)

---

### AWS Bedrock

For enterprise AWS users:

```json5
{
  "agent": {
    "provider": "bedrock",
    "model": "anthropic.claude-3-sonnet",
    "bedrock": {
      "region": "us-east-1"
    }
  }
}
```

Requires AWS credentials configured via AWS CLI or environment variables.

---

## Media Understanding

Different providers have different media capabilities:

### Image Understanding

| Provider | Support |
|----------|---------|
| Anthropic | Excellent |
| OpenAI | Excellent (GPT-4o) |
| Google | Excellent |
| Ollama | Model-dependent |

### Audio Transcription

```json5
{
  "media": {
    "audio": {
      "provider": "groq",      // or "openai", "deepgram"
      "model": "whisper-large-v3"
    }
  }
}
```

| Provider | Quality | Speed |
|----------|---------|-------|
| Groq | High | Very fast |
| OpenAI | High | Moderate |
| Deepgram | High | Fast |

### Video Understanding

```json5
{
  "media": {
    "video": {
      "provider": "google",
      "model": "gemini-1.5-flash"
    }
  }
}
```

Currently, Google Gemini has the best video understanding support.

---

## Credential Storage

Credentials are stored securely in `~/.clawdbot/credentials/`:

```
~/.clawdbot/credentials/
├── anthropic.json
├── openai.json
├── google.json
└── ...
```

**Security:**
- Files created with `0o600` permissions (owner-only)
- Never committed to version control
- Support environment variable substitution

---

## Switching Providers

### Via Configuration

Edit `~/.clawdbot/clawdbot.json`:

```json5
{
  "agent": {
    "model": "gpt-4o",
    "provider": "openai"
  }
}
```

### Via CLI

```bash
clawdbot config set agent.model gpt-4o
clawdbot config set agent.provider openai
```

### Per-Channel

Different channels can use different providers:

```json5
{
  "bindings": [
    {
      "match": { "channel": "telegram" },
      "agent": {
        "model": "claude-opus-4-5",
        "provider": "anthropic"
      }
    },
    {
      "match": { "channel": "discord" },
      "agent": {
        "model": "ollama:llama3.2",
        "provider": "ollama"
      }
    }
  ]
}
```

---

## Cost Comparison

**Approximate costs per 1M tokens (as of 2024):**

| Provider/Model | Input | Output |
|----------------|-------|--------|
| Claude Opus 4.5 | $15 | $75 |
| Claude Sonnet | $3 | $15 |
| Claude Haiku | $0.25 | $1.25 |
| GPT-4o | $5 | $15 |
| GPT-4o-mini | $0.15 | $0.60 |
| Gemini Flash | $0.075 | $0.30 |
| Ollama | Free | Free |

**Note:** Ollama is free but requires your own hardware.

---

## Best Practices

### For Privacy

Use Ollama with a local model:
```json5
{
  "agent": {
    "model": "ollama:llama3.2",
    "provider": "ollama"
  }
}
```

### For Cost

Use cheaper models for simple tasks:
```json5
{
  "agent": {
    "model": "claude-haiku-3-5",
    "provider": "anthropic"
  }
}
```

### For Quality

Use the most capable model:
```json5
{
  "agent": {
    "model": "claude-opus-4-5",
    "provider": "anthropic"
  }
}
```

---

## Environment Variables

| Variable | Provider |
|----------|----------|
| `ANTHROPIC_API_KEY` | Anthropic |
| `OPENAI_API_KEY` | OpenAI |
| `GEMINI_API_KEY` | Google |
| `GROQ_API_KEY` | Groq |
| `OPENROUTER_API_KEY` | OpenRouter |
| `MINIMAX_API_KEY` | MiniMax |

---

*Continue to [05 - Installation Guide](./05-installation-guide.md) →*
