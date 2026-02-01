# Deployment runbook: Docker Model Runner (local AI, zero API cost)

> **Note:** This guide is for OpenClaw (formerly Moltbot/Clawdbot). The CLI command remains `clawdbot`.

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
  - [Standalone Mac mini](./standalone-mac-mini.md)
  - [Isolated VPS](./isolated-vps.md)
  - [Cloudflare Moltworker](./cloudflare-moltworker.md)
  - [Docker Model Runner](./docker-model-runner.md)
- Optimizations
  - [Cost + token optimization](../06-optimizations/cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

Goal: run OpenClaw with **local AI models** via Docker Model Runner for **zero API costs** and **complete privacy**.

This is the ideal setup when:
- You want complete privacy (no data leaves your machine)
- You want zero ongoing API costs
- You have capable hardware (Apple Silicon, NVIDIA GPU, or AMD GPU)
- You want to work offline

Related official docs:
- https://docs.docker.com/desktop/features/model-runner/
- https://docs.openclaw.ai/gateway/local-models

---

## What is Docker Model Runner?

Docker Model Runner (DMR) is Docker Desktop's built-in tool for running LLMs locally. It provides:

- **OpenAI-compatible API** at `http://model-runner.docker.internal/v1`
- **Automatic resource management** — models load on demand, unload when idle
- **Multiple inference engines** — llama.cpp (all platforms), vLLM (NVIDIA), Diffusers (image generation)
- **Zero data collection** option available

**Plain English:** Instead of sending your messages to Anthropic/OpenAI (and paying per token), you run a local AI model on your own computer. Docker Desktop handles all the complexity — downloading models, GPU acceleration, and serving them via an API that looks just like OpenAI's.

---

## When to use DMR vs cloud providers

| Scenario | Recommendation |
|----------|---------------|
| Maximum privacy | **DMR** — data never leaves your machine |
| Zero ongoing costs | **DMR** — one-time model download only |
| Best model quality | **Cloud** — Claude Opus, GPT-4o are still more capable |
| Offline operation | **DMR** — works without internet after setup |
| Limited hardware | **Cloud** — no GPU/RAM requirements |
| Quick setup | **Cloud** — just add API key |
| Production/commercial | **Cloud** — better reliability and quality guarantees |

**Bottom line:** DMR is excellent for privacy-conscious personal use, development/testing, and learning. For production workloads requiring highest-quality responses, cloud providers still have an edge.

---

## System Requirements

Docker Model Runner has different requirements depending on your platform:

| Platform | Requirements |
|----------|-------------|
| **macOS** | Apple Silicon (M1/M2/M3/M4) required |
| **Windows** | NVIDIA GPU (driver 576.57+) or Qualcomm Adreno |
| **Linux** | CPU, NVIDIA (CUDA), AMD (ROCm), or Vulkan |

### Memory recommendations

| Model Size | RAM Required | Best For |
|------------|--------------|----------|
| 1-3B params | 8GB | Quick responses, simple tasks |
| 7-8B params | 16GB | General assistant, coding help |
| 13B+ params | 32GB+ | Complex reasoning, longer context |

**Tip:** If you're unsure, start with a 7B model like `ai/qwen2.5-coder` — it balances quality and resource usage well.

---

## Step-by-step setup

### 1) Install or update Docker Desktop

Docker Model Runner requires Docker Desktop 4.40 or later.

**macOS:**
```bash
# Check current version
docker --version

# Update via brew (if installed via brew)
brew upgrade --cask docker

# Or download from https://www.docker.com/products/docker-desktop/
```

**Windows/Linux:** Download from https://www.docker.com/products/docker-desktop/

### 2) Enable Docker Model Runner

1. Open Docker Desktop
2. Go to **Settings** (gear icon)
3. Navigate to **Features in development** → **Beta features**
4. Enable **Docker Model Runner**
5. Click **Apply & restart**

**Alternative (CLI):**
```bash
docker desktop enable model-runner --tcp 12434
```

### 3) Verify Docker Model Runner is working

```bash
# List available models
docker model list

# Should show available models (empty initially)
```

### 4) Pull a model

Choose a model appropriate for your hardware and use case:

```bash
# Recommended: GLM 4.7 Flash (fast coding assistance)
docker model pull glm-4.7-flash

# Alternative: Qwen3 Coder (agentic coding workflows)
docker model pull qwen3-coder

# Alternative: GPT-OSS (complex reasoning & scheduling)
docker model pull gpt-oss
```

**Other popular models:**
```bash
# Qwen 2.5 Coder (proven coding model)
docker model pull ai/qwen2.5-coder

# Llama 3.2 (general purpose)
docker model pull ai/llama3.2

# Mistral (fast, efficient)
docker model pull ai/mistral
```

### 5) Test the model directly

```bash
# Run a quick test
docker model run glm-4.7-flash "What is a recursive function?"
```

You should see a response generated locally.

### 6) Configure OpenClaw to use Docker Model Runner

```bash
# Set provider to OpenAI-compatible mode
clawdbot config set provider.name openai

# Point to Docker Model Runner's API
clawdbot config set provider.baseUrl http://model-runner.docker.internal/v1

# Set the model name (must match what you pulled)
clawdbot config set provider.model glm-4.7-flash

# API key is not needed but the field must exist
clawdbot config set provider.apiKey "not-needed"
```

### 7) Verify OpenClaw configuration

```bash
# Check status
clawdbot status

# Test a message
clawdbot message send --to self "Hello, are you running locally?"
```

### 8) Run the security audit

```bash
clawdbot security audit --deep
```

If issues are found:
```bash
clawdbot security audit --fix
```

---

## Recommended models for OpenClaw

### Docker-recommended models (2025)

| Model | Best For | Pull Command |
|-------|----------|--------------|
| `glm-4.7-flash` | Fast coding assistance and debugging | `docker model pull glm-4.7-flash` |
| `qwen3-coder` | Agentic coding workflows | `docker model pull qwen3-coder` |
| `gpt-oss` | Complex reasoning and scheduling | `docker model pull gpt-oss` |

### Other popular models

| Model | Size | Best For | Pull Command |
|-------|------|----------|--------------|
| `ai/qwen2.5-coder` | 7B | Coding, technical tasks | `docker model pull ai/qwen2.5-coder` |
| `ai/llama3.2` | 3B/8B | General conversation | `docker model pull ai/llama3.2` |
| `ai/mistral` | 7B | Fast responses, balanced | `docker model pull ai/mistral` |
| `ai/gemma2` | 9B | Google's open model | `docker model pull ai/gemma2` |
| `ai/phi-4` | 14B | Microsoft's reasoning model | `docker model pull ai/phi-4` |
| `ai/deepseek-r1` | Various | Advanced reasoning (larger) | `docker model pull ai/deepseek-r1` |

**For most users:** Start with `glm-4.7-flash` for fast coding help or `qwen3-coder` for agentic workflows — these are Docker's current recommendations and well-suited for OpenClaw's technical assistant use cases.

---

## Configuration options

### Adjusting context size

Some models support configurable context windows. This is set via the model configuration:

```bash
# Check model details
docker model inspect glm-4.7-flash
```

### GPU acceleration

DMR automatically uses available GPU acceleration:
- **macOS:** Metal (Apple Silicon)
- **Windows/Linux:** CUDA (NVIDIA) or ROCm (AMD)

No additional configuration needed — Docker Desktop detects and uses your GPU automatically.

### Resource limits

Docker Desktop allows configuring resource limits for the Model Runner:

1. Open Docker Desktop Settings
2. Navigate to **Resources**
3. Adjust **Memory** and **CPU** limits as needed

**Recommended:** Allocate at least 8GB RAM for 7B models, 16GB for 13B+ models.

---

## Combining with cloud fallback

You can configure OpenClaw to use local models for most requests while falling back to cloud providers for complex tasks.

**Option 1: Manual switching**
```bash
# Switch to local
clawdbot config set provider.baseUrl http://model-runner.docker.internal/v1
clawdbot config set provider.model glm-4.7-flash

# Switch to cloud
clawdbot config set provider.baseUrl https://api.anthropic.com
clawdbot config set provider.model claude-sonnet-4-20250514
```

**Option 2: Use profiles**
```bash
# Create a local profile
CLAWDBOT_PROFILE=local clawdbot config set provider.baseUrl http://model-runner.docker.internal/v1

# Create a cloud profile
CLAWDBOT_PROFILE=cloud clawdbot config set provider.baseUrl https://api.anthropic.com

# Run with specific profile
CLAWDBOT_PROFILE=local clawdbot gateway run
```

---

## Limitations and trade-offs

### Model quality

Local models (7B-13B parameters) are generally less capable than frontier cloud models (Claude Opus, GPT-4o):

| Capability | Local 7B | Cloud Frontier |
|------------|----------|----------------|
| Simple Q&A | Good | Excellent |
| Code generation | Good | Excellent |
| Complex reasoning | Limited | Excellent |
| Long context | Limited (typically 4K-8K) | Large (100K+) |
| Tool use/function calling | Varies by model | Excellent |

### Resource usage

- **RAM:** Models load into memory; expect 8-16GB+ usage
- **Disk:** Each model is 2-20GB on disk
- **GPU VRAM:** If using GPU acceleration, model fits in VRAM
- **First response:** Cold start takes 10-30 seconds as model loads

### Reliability

- Local models may occasionally produce lower-quality or inconsistent responses
- No automatic retry/fallback without additional configuration
- Model updates require manual pulls

---

## Troubleshooting

### "Connection refused" or "Cannot connect to model-runner"

1. Verify Docker Desktop is running
2. Verify Model Runner is enabled in Docker Desktop settings
3. Check the model is pulled: `docker model list`
4. Verify the API is accessible:
   ```bash
   curl http://localhost:12434/v1/models
   ```

### Slow responses

- Check available RAM (model may be swapping to disk)
- Try a smaller/faster model (`glm-4.7-flash` is optimized for speed)
- Verify GPU acceleration is working:
  ```bash
  docker model inspect glm-4.7-flash
  # Look for "accelerator" field
  ```

### Model not found

Ensure the model name in OpenClaw config matches exactly what you pulled:

```bash
# List pulled models
docker model list

# Verify config
clawdbot config get provider.model
```

### High memory usage

Models stay loaded for fast responses. To unload:

```bash
# Stop all running models
docker model stop --all
```

Or configure auto-unload timeout in Docker Desktop settings.

### OpenClaw can't reach the API

The special hostname `model-runner.docker.internal` only works from within Docker. If OpenClaw runs outside Docker:

```bash
# Use localhost with the exposed port instead
clawdbot config set provider.baseUrl http://localhost:12434/v1
```

---

## Privacy considerations

### What DMR provides

- **No cloud transmission:** Inference happens entirely on your machine
- **No telemetry option:** Docker Desktop can be configured to disable telemetry
- **No API keys exposed:** No secrets to manage for local inference

### What you're still responsible for

- **Session transcripts:** Still stored locally in `~/.clawdbot/`
- **Channel tokens:** WhatsApp/Telegram tokens still needed
- **Network exposure:** Keep Gateway loopback-only
- **Disk encryption:** Enable FileVault/LUKS for credential protection

### Maximum privacy configuration

For the most private setup:

```bash
# Run Gateway loopback-only
clawdbot config set gateway.bind loopback

# Use local model
clawdbot config set provider.baseUrl http://model-runner.docker.internal/v1
clawdbot config set provider.model glm-4.7-flash

# Enable Docker sandbox for tool execution
clawdbot config set agents.defaults.sandbox docker
clawdbot config set agents.defaults.sandboxNetwork none
```

Combined with FileVault/LUKS disk encryption, this keeps all AI processing and data on hardware you control.

---

## Security Checklist (Docker Model Runner)

### Docker Desktop

- [ ] Docker Desktop 4.40+ installed
- [ ] Model Runner feature enabled
- [ ] Resource limits configured appropriately
- [ ] Docker Desktop telemetry disabled (if desired)

### OpenClaw Configuration

- [ ] `provider.baseUrl` points to Model Runner
- [ ] `provider.model` matches a pulled model
- [ ] `gateway.bind` set to `loopback`
- [ ] DM policy is `allowlist` or `pairing`

### Model Management

- [ ] Model pulled and verified working
- [ ] Model appropriate for available RAM
- [ ] GPU acceleration verified (if available)

### General Security

- [ ] FileVault/LUKS disk encryption enabled
- [ ] `~/.clawdbot/` permissions are 0700
- [ ] Shell history protection enabled
- [ ] `clawdbot security audit --deep` passed

---

## Cost comparison

| Deployment | Setup Cost | Monthly Cost | Notes |
|------------|------------|--------------|-------|
| **DMR** | Hardware (existing) | $0 | Model download bandwidth only |
| **Cloud (Anthropic)** | $0 | $5-50+ | Pay per token |
| **VPS + Cloud** | $0 | $6-20+ | VPS + API costs |
| **Moltworker** | $0 | $5-15 | Cloudflare Workers plan |

**Bottom line:** If you already have capable hardware (Apple Silicon Mac, gaming PC with GPU), DMR provides indefinite local AI at zero marginal cost.

---

## Next steps

After setup:

1. **Test tool execution** — verify web fetch, exec tools work correctly
2. **Experiment with models** — try different models for your use cases
3. **Configure channels** — set up WhatsApp/Telegram as usual
4. **Review transcripts** — ensure response quality meets your needs

For channel setup, see: [Pairing Guide](https://docs.openclaw.ai/start/pairing)

For local model optimization: https://docs.openclaw.ai/gateway/local-models
