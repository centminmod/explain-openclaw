# Cost + Token Optimization

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
  - [Cost + token optimization](./cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

## Overview

Running a self-hosted AI assistant means paying for model API calls. This guide covers strategies to reduce costs and optimize token usage without sacrificing quality.

**Key areas:**
- OpenRouter integration for smart model routing
- Provider fallbacks and load balancing
- Token usage reduction techniques
- Cost monitoring and alerts

---

## OpenRouter Integration

> **Official guide:** [OpenRouter OpenClaw Integration](https://openrouter.ai/docs/guides/guides/openclaw-integration#using-auto-model-for-cost-optimization)

OpenRouter is a unified API that provides access to 200+ models from multiple providers (Anthropic, OpenAI, Google, Meta, etc.) with automatic routing, fallbacks, and cost optimization.

### Why Use OpenRouter?

| Benefit | Description |
|---------|-------------|
| **Single API key** | Access Claude, GPT, Gemini, Llama, and more without managing multiple accounts |
| **Auto routing** | Intelligent model selection based on prompt complexity |
| **Provider fallbacks** | Automatic retry on provider outages |
| **Cost control** | Price caps, sorting by cost, budget alerts |
| **Privacy options** | Zero Data Retention (ZDR) endpoints available |

### Basic Configuration

Set OpenRouter as your provider in OpenClaw:

```bash
openclaw config set provider.name openrouter
openclaw config set provider.apiKey sk-or-v1-your-key-here
openclaw config set provider.model openrouter/auto
```

Or in config file (`~/.openclaw/config.yaml`):

```yaml
provider:
  name: openrouter
  apiKey: sk-or-v1-your-key-here
  model: openrouter/auto
```

---

## Auto Router (`openrouter/auto`)

The Auto Router automatically selects the best model for each prompt, powered by NotDiamond's model routing technology.

### How It Works

1. Analyzes your prompt for complexity, task type, and required capabilities
2. Routes to the optimal model from a curated set (Claude, GPT, Gemini, DeepSeek, etc.)
3. Returns the response with metadata about which model was used

### Configuration

```json
{
  "model": "openrouter/auto",
  "messages": [{"role": "user", "content": "..."}]
}
```

### Key Characteristics

- **No additional fee** - Pay the standard rate for whichever model is selected
- **Transparent selection** - Response includes `model` field showing which model handled the request
- **Quality-optimized by default** - Prioritizes response quality over cost unless configured otherwise

### Example Response Metadata

```json
{
  "model": "anthropic/claude-sonnet-4",
  "usage": {
    "prompt_tokens": 150,
    "completion_tokens": 200,
    "total_tokens": 350
  }
}
```

---

## Free Router (`openrouter/free`)

The `openrouter/free` model provides access to free AI models through OpenRouter, ideal for testing, development, or budget-constrained applications.

### Configuration

```bash
openclaw config set provider.model openrouter/free
```

Or via API:

```json
{
  "model": "openrouter/free",
  "messages": [{"role": "user", "content": "..."}]
}
```

### How It Works

Similar to the Auto Router, `openrouter/free` analyzes your prompt and routes to an appropriate free model from available options. The response includes the `model` field showing which specific free model was used.

### Limitations

Free models have certain constraints compared to paid models:

| Limitation | Description |
|------------|-------------|
| **Rate limits** | Lower requests per minute/hour |
| **Queue priority** | Longer wait times during high usage |
| **Capabilities** | May have reduced capabilities vs premium models |
| **Availability** | Free models may not always be available |

### Best Use Cases

- **Development and testing** - Prototype without cost concerns
- **Educational projects** - Learn AI integration without budget
- **Low-volume applications** - Minimal usage that doesn't justify paid models
- **Fallback scenarios** - Backup when paid model budgets are exhausted

---

## Model Variants for Cost/Speed Trade-offs

OpenRouter offers model variants that modify routing behavior:

| Variant Suffix | Behavior | Use Case |
|----------------|----------|----------|
| (none) | Default balanced routing | General use |
| `:floor` | Sort by price (cheapest provider first) | Cost-sensitive applications |
| `:nitro` | Sort by throughput (fastest provider first) | Latency-sensitive applications |

### Examples

```bash
# Default - balanced quality/cost/speed
openclaw config set provider.model anthropic/claude-sonnet-4

# Cheapest provider for Claude Sonnet
openclaw config set provider.model anthropic/claude-sonnet-4:floor

# Fastest provider for Claude Sonnet
openclaw config set provider.model anthropic/claude-sonnet-4:nitro
```

### When to Use Each

- **Default:** Most use cases where quality matters
- **`:floor`:** Background tasks, bulk processing, development/testing
- **`:nitro`:** Real-time chat, interactive applications, time-sensitive workflows

---

## Provider Routing Configuration

Fine-grained control over how OpenRouter selects providers:

```json
{
  "model": "openrouter/auto",
  "provider": {
    "order": ["anthropic", "openai"],
    "allow_fallbacks": true,
    "sort": "price",
    "data_collection": "deny",
    "max_price": {
      "prompt": 1,
      "completion": 2
    }
  }
}
```

### Configuration Options

| Option | Values | Description |
|--------|--------|-------------|
| `sort` | `"price"`, `"throughput"` | How to rank providers |
| `order` | Provider array | Explicit provider preference order |
| `allow_fallbacks` | `true`, `false` | Auto-retry on provider errors |
| `data_collection` | `"allow"`, `"deny"` | Control training data usage |
| `zdr` | `true`, `false` | Zero Data Retention only |
| `only` | Provider array | Whitelist specific providers |
| `ignore` | Provider array | Blacklist specific providers |
| `max_price.prompt` | Number | Max cost per million prompt tokens |
| `max_price.completion` | Number | Max cost per million completion tokens |

### Privacy-Focused Configuration

```json
{
  "model": "openrouter/auto",
  "provider": {
    "data_collection": "deny",
    "zdr": true,
    "only": ["anthropic"]
  }
}
```

### Cost-Focused Configuration

```json
{
  "model": "openrouter/auto",
  "provider": {
    "sort": "price",
    "max_price": {
      "prompt": 0.5,
      "completion": 1.5
    },
    "allow_fallbacks": true
  }
}
```

---

## Load Balancing Algorithm

OpenRouter automatically load balances across providers using inverse-square pricing:

```
Weight = 1 / (price^2)
```

### What This Means

- A $1/M token provider is **9x more likely** to be selected than a $3/M provider
- A $0.50/M provider is **4x more likely** than a $1/M provider
- Recent outages reduce a provider's weight temporarily
- This creates natural cost optimization without explicit configuration

### Overriding Load Balancing

To disable automatic load balancing, use explicit configuration:

```json
{
  "provider": {
    "sort": "throughput",
    "order": ["anthropic", "openai"]
  }
}
```

---

## Token Usage Strategies

Beyond model routing, reduce costs by managing token consumption:

### 1. Context Window Management

Long conversations accumulate tokens. Strategies to manage this:

```yaml
# In agent config
agents:
  defaults:
    maxContextTokens: 50000
    contextPruning: sliding-window
```

| Strategy | Description | Best For |
|----------|-------------|----------|
| `sliding-window` | Keep most recent N tokens | General chat |
| `summary` | Periodically summarize and compress | Long-running sessions |
| `selective` | Keep only high-relevance messages | Task-focused agents |

### 2. Efficient System Prompts

System prompts are sent with every request. Keep them concise:

```yaml
# Bad: 2000+ tokens
agents:
  default:
    systemPrompt: |
      You are an incredibly helpful assistant that always...
      [long detailed instructions]

# Good: 200 tokens
agents:
  default:
    systemPrompt: |
      Helpful assistant. Be concise. Ask clarifying questions when needed.
```

### 3. Response Length Control

Prevent unnecessarily verbose responses:

```yaml
agents:
  defaults:
    maxOutputTokens: 1000
```

### 4. Tool Result Compression

Large tool outputs (web pages, file contents) consume tokens:

```yaml
tools:
  web-fetch:
    maxResponseTokens: 2000
    summarize: true
```

---

## Model Selection Trade-offs

Choose models based on task complexity:

| Task Type | Recommended Model | Typical Cost |
|-----------|-------------------|--------------|
| Simple Q&A | `openrouter/auto` or Haiku | $0.25-1/M |
| Code generation | Claude Sonnet or GPT-4o | $3-15/M |
| Complex reasoning | Claude Opus or o1 | $15-60/M |
| Bulk processing | Haiku, Gemini Flash | $0.10-0.50/M |
| Testing/dev | `openrouter/free` | $0 |

### Per-Task Model Override

Configure different models for different scenarios:

```yaml
agents:
  quick-lookup:
    provider:
      model: anthropic/claude-haiku

  deep-analysis:
    provider:
      model: anthropic/claude-opus-4
```

---

## Monitoring Costs

### OpenRouter Activity Dashboard

Access at: [openrouter.ai/activity](https://openrouter.ai/activity)

Features:
- Real-time usage tracking
- Per-model cost breakdown
- Daily/weekly/monthly summaries
- Export to CSV

### Budget Alerts

Set up alerts in OpenRouter dashboard:
1. Go to Settings > Billing
2. Set monthly budget limit
3. Configure alert thresholds (50%, 75%, 90%)
4. Add email notifications

### Gateway-Level Tracking

Monitor usage in OpenClaw:

```bash
# View recent usage
openclaw usage --last 7d

# Export usage data
openclaw usage --export csv --output usage.csv
```

### Cost Estimation

Before enabling expensive features, estimate costs:

| Feature | Approximate Token Cost |
|---------|----------------------|
| Average message | 500-2000 tokens |
| Web fetch (summarized) | 1000-3000 tokens |
| Code file analysis | 500-5000 tokens |
| Image analysis | 1000-2000 tokens |

**Example monthly estimate:**
- 50 messages/day x 1500 avg tokens = 75,000 tokens/day
- 75,000 x 30 days = 2.25M tokens/month
- At $3/M (Sonnet): ~$7/month
- At $15/M (Opus): ~$34/month

---

## Best Practices Summary

1. **Start with auto routing** - Let OpenRouter optimize unless you have specific needs
2. **Use `:floor` variants for development** - Save costs during testing
3. **Set `max_price` limits** - Prevent surprise bills from expensive models
4. **Monitor weekly** - Review dashboard and adjust as needed
5. **Compress context** - Enable pruning for long-running sessions
6. **Match model to task** - Don't use Opus for simple lookups
7. **Enable fallbacks** - Avoid failed requests during provider outages

---

## Related Resources

- [OpenRouter Documentation](https://openrouter.ai/docs)
- [OpenRouter Pricing](https://openrouter.ai/models)
- [OpenClaw Provider Configuration](https://docs.openclaw.ai/configuration#provider)
- [Model Comparison Guide](https://docs.openclaw.ai/models)
