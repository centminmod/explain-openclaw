# Cost + Token Optimization

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

## OpenClaw Token Consumption

This section reflects current runtime behavior in the local OpenClaw codebase.

### 1. Baseline per-run token load: persona/bootstrap files

OpenClaw injects workspace bootstrap files into the system prompt each run (or a `[MISSING]` placeholder when a file is absent):

| File | Loaded in main sessions | Loaded in subagent/cron sessions | Max injected chars per file (default) | Approx max tokens per file* |
|------|--------------------------|-----------------------------------|----------------------------------------|-----------------------------|
| `AGENTS.md` | Yes | Yes | 20,000 | ~5,000 |
| `SOUL.md` | Yes | No | 20,000 | ~5,000 |
| `TOOLS.md` | Yes | Yes | 20,000 | ~5,000 |
| `IDENTITY.md` | Yes | No | 20,000 | ~5,000 |
| `USER.md` | Yes | No | 20,000 | ~5,000 |
| `HEARTBEAT.md` | Yes | No | 20,000 | ~5,000 |
| `BOOTSTRAP.md` | Yes | No | 20,000 | ~5,000 |
| `MEMORY.md` / `memory.md` (if present) | Optional | No | 20,000 | ~5,000 |

Total injected bootstrap text is also capped by `agents.defaults.bootstrapTotalMaxChars` (default `150,000`, roughly ~37,500 tokens*).

For `subagent:*` and `cron:*` session keys, OpenClaw filters bootstrap injection to only `AGENTS.md` and `TOOLS.md` to keep non-main runs smaller.

`*` Approximation uses the same rough `~4 chars = 1 token` heuristic used in the codebase token guards.

### 2. Daily files and raw logs: when they consume tokens

`memory/YYYY-MM-DD.md` and session raw logs are **not always** in the prompt. They consume tokens when they are read/retrieved.

| Source | Auto-injected every run? | When tokens are consumed | How OpenClaw limits token impact |
|--------|---------------------------|--------------------------|-----------------------------------|
| `MEMORY.md` / `memory.md` (workspace root) | Main sessions: yes | Counts every run when injected in Project Context | `bootstrapMaxChars` per-file cap + `bootstrapTotalMaxChars` total cap |
| `memory/YYYY-MM-DD.md` daily files | No | When retrieved via `memory_search` / `memory_get` (often prompted by AGENTS template workflow) | `memory_search` snippet caps (~700 chars each), result limits, and optional `memory_get` line ranges |
| Session JSONL raw logs (`~/.openclaw/agents/<id>/sessions/*.jsonl`) | No | When explicitly read through tools, or when session memory indexing is enabled and snippets are returned | Session-memory indexing keeps sanitized User/Assistant text only; retrieval is snippet-based (not full logs) |

Notes:
- `memory/*.md` daily files are not auto-injected into the base system prompt.
- With built-in memory search, default chunking is ~400 tokens with ~80-token overlap.
- Session transcript memory indexing is optional (not on by default in built-in memory search).
- QMD memory backend adds an injected-snippet budget (`memory.qmd.limits.maxInjectedChars`, default `4000` chars).

### 3. Other major token consumers in OpenClaw

- **Conversation history**: prior user/assistant/tool turns are replayed each request.
- **Tool results**: large tool outputs can dominate context; OpenClaw truncates oversized tool results (max ~30% of context window per result, hard cap 400,000 chars).
- **System prompt assembly**: tool list, runtime metadata, skills list metadata, and workspace context are rebuilt each run.
- **Tool schemas**: JSON tool schemas also count toward model context budget.
- **Heartbeat runs**: periodic background runs consume model tokens even without new user messages.
- **Compaction/summarization**: compaction can trigger extra model calls to summarize older history.
- **Memory search embeddings**: when `memorySearch` uses remote providers (OpenAI/Gemini/Voyage), embedding/indexing calls consume provider tokens (separate from chat completion tokens).
- **Web and media tools**: web search (Perplexity/OpenRouter), image understanding, and audio transcription can add separate API usage and token spend.
- **Prompt caching (Anthropic)**: cache writes cost the same as normal input tokens; cache reads cost ~10% of normal input tokens. OpenClaw auto-enables `cacheRetention: short` for Anthropic API-key sessions, so repeated system-prompt content is served from cache. Long-running sessions with stable bootstrap files benefit most.

### 4. How OpenClaw compacts, compresses, and manages token pressure

| Mechanism | What it does | Current behavior/defaults |
|----------|---------------|---------------------------|
| Context window resolution | Determines usable context budget for guards/pruning | Uses model config/window, then caps with `agents.defaults.contextTokens` when lower |
| Session pruning (`contextPruning`) | Trims/clears old tool results in-memory before model call | `cache-ttl` mode; runs only for Anthropic or OpenRouter Anthropic models when TTL expired |
| Auto defaults (Anthropic auth) | Applies cost/cache-friendly defaults if unset | Auto-enables `contextPruning.mode=cache-ttl`, sets heartbeat (`1h` oauth, `30m` api_key), applies `cacheRetention: short` for Anthropic API-key mode |
| Tool result truncation | Prevents single tool result from consuming most of context | Max 30% of context window per tool result, hard cap `400000` chars |
| Persistence hard cap | Prevents oversized tool results from being written unbounded to session transcript | Hard cap uses same tool-result max-char ceiling before persistence |
| Preemptive context guard | Compacts/truncates tool outputs before request if context is near limit | Replaces older heavy tool outputs with compact placeholders as needed |
| Compaction mode | Summarizes older history into persisted compact summary | Default compaction mode is `safeguard` |
| Compaction reserve floor | Preserves headroom so routine operations happen before forced compaction | Reserve floor default `20000` tokens (`agents.defaults.compaction.reserveTokensFloor`) |
| Compaction timeout safety | Stops runaway compaction attempts | Safety timeout is `300000 ms` (5 minutes) |

### 5. Operational Token Multipliers (Background + Retries)

These are easy to miss because they add extra model turns beyond the obvious user-to-assistant turn.

| Routine | Why it increases token spend | Trigger |
|--------|-------------------------------|---------|
| Pre-compaction memory flush | Runs an extra silent agentic turn before normal run | Session near compaction threshold and memory flush enabled |
| Model fallback chain | Same prompt can be attempted across multiple provider/model candidates | Failover-eligible errors (rate limits, provider errors, etc.) |
| Overflow recovery retries | Can retry after compaction and/or tool-result truncation attempts | Context overflow conditions |
| Heartbeat wake events | Additional heartbeat runs beyond periodic interval | Exec completion notifications, hook wakes, cron/system wake events |
| Cron main-session events | Adds work to heartbeat-driven model turns in main session | Cron jobs targeting main session (`wake now` or next-heartbeat) |
| `sessions_send` A2A flow | Multi-step internal agent runs for ping-pong + announce workflow | Using `sessions_send` with A2A flow enabled |
| `sessions_spawn` / subagents | Spawns separate isolated agent runs (new model calls) | Using `sessions_spawn` / subagent workflow |
| Isolated cron agent turns | Dedicated `cron:<jobId>` runs invoke full model turns | Cron jobs with `session: isolated` / `agentTurn` |

### 6. Heartbeat Token Controls

- Heartbeats run full model turns (`getReplyFromConfig`), so shorter intervals scale cost quickly.
- Heartbeats can be skipped before model call when `HEARTBEAT.md` exists but is effectively empty (ATX headings and blank checklist items only — lines like `#comment` without a space are not skipped).
- Queue pressure can defer/skip runs (`requests-in-flight`), with wake retry behavior.
- Practical spend controls:
  - Increase `agents.defaults.heartbeat.every`.
  - Use `activeHours` to limit windows.
  - Keep `HEARTBEAT.md` concise and task-focused.
  - Use `target: "none"` for internal-only checks when delivery is unnecessary.

### 7. Configuration Levers That Change Token Consumption + Model Routing

Use these knobs to directly control prompt size, model choice, retries, and auxiliary API spend.

| Token/cost driver | Config levers | Practical effect |
|-------------------|---------------|------------------|
| Bootstrap/persona baseline | `agents.defaults.bootstrapMaxChars`, `agents.defaults.bootstrapTotalMaxChars` | Limits per-file and total injected bootstrap chars each run. |
| Main model + fallback retry cost | `agents.defaults.model.primary`, `agents.defaults.model.fallbacks`, `agents.list[].model` | Controls primary model plus failover chain that may re-run the same prompt on errors. |
| Subagent model spend | `agents.defaults.subagents.model`, `agents.list[].subagents.model` | Controls what model spawned subagents use for isolated turns. |
| Heartbeat model spend | `agents.defaults.heartbeat.model`, `agents.list[].heartbeat.model` | Lets you run cheaper/faster models for recurring heartbeat turns. |
| Vision model routing | `agents.defaults.imageModel.primary`, `agents.defaults.imageModel.fallbacks` | Chooses image-capable model path when primary text model lacks image input. |
| Response-length ceiling | `agents.defaults.models."<provider/model>".params.maxTokens` | Caps completion length per model to reduce output-token spend. |
| Usable context budget | `agents.defaults.contextTokens`, `models.providers.<provider>.models[].contextWindow` | Shrinks/expands effective context window used for guards, pruning, and compaction thresholds. |
| Tool-result context bloat | `agents.defaults.contextPruning.*` (mode/ttl/keepLastAssistants/softTrim/hardClear/tools) | Reduces old tool-result payload kept in request context, especially post-TTL. `keepLastAssistants` sets how many recent assistant turns to always preserve from pruning. |
| Compaction + pre-flush extra turns | `agents.defaults.compaction.mode`, `agents.defaults.compaction.reserveTokensFloor`, `agents.defaults.compaction.memoryFlush.enabled`, `agents.defaults.compaction.memoryFlush.softThresholdTokens` | Tunes when compaction/flush kicks in and how often near-limit sessions trigger extra model turns. |
| Heartbeat frequency multiplier | `agents.defaults.heartbeat.every`, `agents.defaults.heartbeat.activeHours`, `agents.list[].heartbeat` | Directly sets background turn frequency and active windows. |
| Memory retrieval injection | `agents.defaults.memorySearch.query.maxResults`, `agents.defaults.memorySearch.query.minScore`, `agents.defaults.memorySearch.chunking.tokens`, `agents.defaults.memorySearch.chunking.overlap` | Controls how much recalled memory snippet text is selected and indexed for retrieval. |
| Session-log memory inclusion | `agents.defaults.memorySearch.experimental.sessionMemory`, `agents.defaults.memorySearch.sources` (include `"sessions"`), `agents.defaults.memorySearch.sync.sessions.*` | Enables/sizes session transcript indexing so raw session logs can be recalled as snippets. |
| QMD recall payload budget | `memory.qmd.limits.maxResults`, `memory.qmd.limits.maxSnippetChars`, `memory.qmd.limits.maxInjectedChars`, `memory.qmd.sessions.enabled` | Hard caps QMD memory snippet volume injected back into tool results. |
| Web tool payload + provider model | `tools.web.search.maxResults`, `tools.web.search.perplexity.model`, `tools.web.search.grok.model`, `tools.web.fetch.maxChars`, `tools.web.fetch.maxCharsCap` | Controls web result volume and which provider model answers search queries. |
| Media/link tool payload size | `tools.media.<image/audio/video>.maxChars`, `tools.media.<image/audio/video>.maxBytes`, `tools.media.<image/audio/video>.models[]`, `tools.links.maxLinks` | Controls transcription/description volume and number of fetched assets/links. |
| Image token density | `agents.defaults.imageMaxDimensionPx` | Lower dimensions reduce vision-token and payload pressure on screenshot-heavy runs. |
| Wake-driven extra heartbeat turns | `tools.exec.notifyOnExit`, `tools.exec.notifyOnExitEmptySuccess` | Background exec completion can enqueue a system event and trigger immediate heartbeat work. |

Notes:
- Leaving `agents.defaults.memorySearch.provider` unset uses automatic provider selection based on available local/remote keys; remote embedding providers add non-chat token/API spend.
- `memory/YYYY-MM-DD.md` and session JSONL files are not auto-injected; these consume tokens only when retrieved/read (memory tools, read tools, or enabled session-memory indexing).

#### Drift-check manifest (AI-parseable)

Use this as a canonical token-config map for automated doc-drift checks.

```yaml
token_config_manifest_version: 1
key_path_convention:
  list_item_wildcard: "[]"
  quoted_model_key_example: 'agents.defaults.models."openai/gpt-5.2".params.maxTokens'
entries:
  - id: TOKCFG_BOOTSTRAP_MAX_CHARS
    paths: [agents.defaults.bootstrapMaxChars]
    default: 20000
    runtime_refs: [src/agents/pi-embedded-helpers/bootstrap.ts]
  - id: TOKCFG_BOOTSTRAP_TOTAL_MAX_CHARS
    paths: [agents.defaults.bootstrapTotalMaxChars]
    default: 150000
    runtime_refs: [src/agents/pi-embedded-helpers/bootstrap.ts]
  - id: TOKCFG_MODEL_PRIMARY_FALLBACKS
    paths: [agents.defaults.model.primary, agents.defaults.model.fallbacks, agents.list[].model]
    runtime_refs: [src/agents/model-fallback.ts]
  - id: TOKCFG_SUBAGENT_MODEL
    paths: [agents.defaults.subagents.model, agents.list[].subagents.model]
    runtime_refs: [src/config/types.agent-defaults.ts, src/config/types.agents.ts]
  - id: TOKCFG_HEARTBEAT_MODEL
    paths: [agents.defaults.heartbeat.model, agents.list[].heartbeat.model]
    runtime_refs: [src/infra/heartbeat-runner.ts]
  - id: TOKCFG_IMAGE_MODEL_ROUTING
    paths: [agents.defaults.imageModel.primary, agents.defaults.imageModel.fallbacks]
    runtime_refs: [src/media-understanding/runner.ts, src/agents/model-fallback.ts]
  - id: TOKCFG_PER_MODEL_MAX_TOKENS
    paths: ['agents.defaults.models."<provider/model>".params.maxTokens']
    runtime_refs: [src/config/types.agent-defaults.ts]
  - id: TOKCFG_CONTEXT_WINDOW_CAP
    paths: [agents.defaults.contextTokens, models.providers.<provider>.models[].contextWindow]
    runtime_refs: [src/agents/context-window-guard.ts]
  - id: TOKCFG_CONTEXT_PRUNING
    paths: [agents.defaults.contextPruning.mode, agents.defaults.contextPruning.ttl, agents.defaults.contextPruning.softTrim, agents.defaults.contextPruning.hardClear, agents.defaults.contextPruning.tools]
    runtime_refs: [src/agents/pi-extensions/context-pruning]
  - id: TOKCFG_COMPACTION_MEMORY_FLUSH
    paths: [agents.defaults.compaction.mode, agents.defaults.compaction.reserveTokensFloor, agents.defaults.compaction.memoryFlush.enabled, agents.defaults.compaction.memoryFlush.softThresholdTokens]
    runtime_refs: [src/auto-reply/reply/memory-flush.ts, src/auto-reply/reply/agent-runner-memory.ts]
  - id: TOKCFG_HEARTBEAT_INTERVAL_SCOPE
    paths: [agents.defaults.heartbeat.every, agents.defaults.heartbeat.activeHours, agents.list[].heartbeat]
    runtime_refs: [src/infra/heartbeat-runner.ts]
  - id: TOKCFG_MEMORY_SEARCH_RETRIEVAL
    paths: [agents.defaults.memorySearch.query.maxResults, agents.defaults.memorySearch.query.minScore, agents.defaults.memorySearch.chunking.tokens, agents.defaults.memorySearch.chunking.overlap]
    runtime_refs: [src/agents/memory-search.ts]
  - id: TOKCFG_SESSION_MEMORY_INDEXING
    paths: [agents.defaults.memorySearch.experimental.sessionMemory, agents.defaults.memorySearch.sources, agents.defaults.memorySearch.sync.sessions.deltaBytes, agents.defaults.memorySearch.sync.sessions.deltaMessages]
    runtime_refs: [src/agents/memory-search.ts, src/memory/manager-sync-ops.ts, src/memory/session-files.ts]
  - id: TOKCFG_QMD_SNIPPET_BUDGET
    paths: [memory.qmd.limits.maxResults, memory.qmd.limits.maxSnippetChars, memory.qmd.limits.maxInjectedChars, memory.qmd.sessions.enabled]
    runtime_refs: [src/memory/backend-config.ts, src/memory/qmd-manager.ts]
  - id: TOKCFG_WEB_SEARCH_FETCH_BOUNDS
    paths: [tools.web.search.maxResults, tools.web.search.perplexity.model, tools.web.search.grok.model, tools.web.fetch.maxChars, tools.web.fetch.maxCharsCap]
    runtime_refs: [src/agents/tools/web-search.ts, src/agents/tools/web-fetch.ts]
  - id: TOKCFG_MEDIA_LINK_BOUNDS
    paths: [tools.media.image.maxChars, tools.media.audio.maxChars, tools.media.video.maxChars, tools.media.image.maxBytes, tools.media.audio.maxBytes, tools.media.video.maxBytes, tools.media.image.models, tools.media.audio.models, tools.media.video.models, tools.links.maxLinks]
    runtime_refs: [src/media-understanding/resolve.ts, src/link-understanding/defaults.ts]
  - id: TOKCFG_IMAGE_DIMENSION_PX
    paths: [agents.defaults.imageMaxDimensionPx]
    default: 1200
    runtime_refs: [src/config/types.agent-defaults.ts]
  - id: TOKCFG_EXEC_WAKE_MULTIPLIER
    paths: [tools.exec.notifyOnExit, tools.exec.notifyOnExitEmptySuccess]
    runtime_refs: [src/agents/bash-tools.exec-runtime.ts, src/infra/heartbeat-wake.ts]
```

#### Drift-check query starter

```bash
rg -n "bootstrapMaxChars|bootstrapTotalMaxChars|contextTokens|contextPruning|compaction|memoryFlush|heartbeat|imageModel|imageMaxDimensionPx|memorySearch|memory\\.qmd|tools\\.web\\.search|maxResults|tools\\.web\\.fetch|maxCharsCap|tools\\.media|tools\\.links|maxLinks|notifyOnExit" src docs
```

### 8. How token counts are estimated

OpenClaw does not use a tokenizer library (no tiktoken, js-tiktoken, gpt-tokenizer, or @anthropic-ai/tokenizer) and does not call the Anthropic `count_tokens` API endpoint. Instead it uses a **two-track approach**:

**Track 1 — Pre-call character approximation (guards, pruning, adaptive sizing)**

Three modules share the same 4 chars ≈ 1 token heuristic for decisions that must happen before an API call is made:

| Module | Constant | Rate | Purpose |
|--------|----------|------|---------|
| `src/agents/pi-tools.read.ts` | `CHARS_PER_TOKEN_ESTIMATE` | 4 | Adaptive read page sizing |
| `src/agents/pi-extensions/context-pruning/pruner.ts` | `CHARS_PER_TOKEN_ESTIMATE` | 4 | Context pruning decisions |
| `src/agents/pi-embedded-runner/tool-result-context-guard.ts` | `CHARS_PER_TOKEN_ESTIMATE` | 4 | General context guard |
| `src/agents/pi-embedded-runner/tool-result-context-guard.ts` | `TOOL_RESULT_CHARS_PER_TOKEN_ESTIMATE` | **2** | Tool-result context guard |

The tool-result guard uses **2 chars/token** (not 4) because tool output — code, JSON, file contents — is token-denser than prose. The code comment explains: "Keep a conservative input budget to absorb tokenizer variance and provider framing overhead." Tool result truncation in `src/agents/pi-embedded-runner/tool-result-truncation.ts` also references the 4 chars/token baseline for its cap calculation.

**Track 2 — Actual API-returned counts (post-call usage tracking and display)**

After each API call, OpenClaw reads the real token counts from the response fields `input_tokens`, `output_tokens`, `total_tokens`, `cache_read_tokens`, and `cache_write_tokens`. These are used for usage logging (`src/cron/run-log.ts`) and for status display via `formatTokenCount` (`src/cron/isolated-agent/run.ts`).

**Summary:** Pre-call decisions are governed by a fast character heuristic (4 or 2 chars/token). Post-call reporting uses exact counts from the provider's API response.

### 9. Accuracy and mapping notes

- Updated legacy config paths like `provider.model` to current model selection paths (`agents.defaults.model.primary`).
- Updated free OpenRouter routing examples to current model-ref style (`openrouter/<provider>/<model>:free`) instead of `openrouter/free`.
- Updated token control examples to current keys (`contextTokens`, `contextPruning.mode=cache-ttl`, `agents.defaults.models.<model>.params.maxTokens`, `tools.web.fetch.maxChars`).

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

Use OpenRouter auth plus a model reference in OpenClaw:

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
openclaw config set agents.defaults.model.primary openrouter/auto
```

Or in config:

```yaml
env:
  OPENROUTER_API_KEY: sk-or-v1-your-key-here

agents:
  defaults:
    model:
      primary: openrouter/auto
```

For explicit model pinning, use `openrouter/<provider>/<model>` (example: `openrouter/anthropic/claude-sonnet-4-6`).

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

## Free Models (`:free` Suffix)

In current OpenClaw model references, free OpenRouter routing is typically selected with `:free` on a specific model.

### Configuration

```bash
openclaw config set agents.defaults.model.primary openrouter/meta-llama/llama-3.3-70b-instruct:free
```

Or via API:

```json
{
  "model": "meta-llama/llama-3.3-70b-instruct:free",
  "messages": [{"role": "user", "content": "..."}]
}
```

### How It Works

OpenRouter routes to a free tier for the specified model variant where available. The response metadata includes the concrete model/provider used.

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
openclaw config set agents.defaults.model.primary openrouter/anthropic/claude-sonnet-4-6

# Cheapest provider for Claude Sonnet
openclaw config set agents.defaults.model.primary openrouter/anthropic/claude-sonnet-4-6:floor

# Fastest provider for Claude Sonnet
openclaw config set agents.defaults.model.primary openrouter/anthropic/claude-sonnet-4-6:nitro
```

### When to Use Each

- **Default:** Most use cases where quality matters
- **`:floor`:** Background tasks, bulk processing, development/testing
- **`:nitro`:** Real-time chat, interactive applications, time-sensitive workflows

---

## Provider Routing Configuration

Fine-grained control over how OpenRouter selects providers:

> Note: this `provider` object is an OpenRouter request payload example. OpenClaw does not currently expose first-class config keys for all of these routing fields.

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
agents:
  defaults:
    contextTokens: 50000
    contextPruning:
      mode: cache-ttl
      ttl: 5m
      keepLastAssistants: 3
      softTrim:
        maxChars: 4000
        headChars: 1500
        tailChars: 1500
      hardClear:
        enabled: true
```

| Strategy | Description | Best For |
|----------|-------------|----------|
| `contextTokens` | Defines session context budget for status/guardrails | Predictable context headroom |
| `contextPruning: cache-ttl` | Soft-trims/clears older tool results after cache TTL | Long-running sessions with large tool output |
| `compaction` | Summarizes prior history when transcripts get large | Multi-day chats with long history |

### 2. Efficient System Prompts

System prompts are sent with every request. Keep workspace bootstrap docs concise and bounded:

```yaml
agents:
  defaults:
    bootstrapMaxChars: 12000
    bootstrapTotalMaxChars: 80000
```

### 3. Response Length Control

Prevent unnecessarily verbose responses with per-model params:

```yaml
agents:
  defaults:
    models:
      "openrouter/anthropic/claude-sonnet-4-6":
        params:
          maxTokens: 1000
```

### 4. Tool Result Compression

Large tool outputs (web pages, file contents) consume tokens:

```yaml
tools:
  web:
    fetch:
      maxChars: 8000
      readability: true
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
| Testing/dev | `openrouter/<provider>/<model>:free` | $0 |

### OpenRouter Model Pricing Reference

Prices are per million tokens via OpenRouter (Feb 2026). Append `:floor` to any model ID for cheapest-provider routing, or `:nitro` for fastest throughput.

| Model | Input / M tokens | Output / M tokens | Notes |
| ----- | ----- | ----- | ----- |
| `anthropic/claude-opus-4-6` | $5.00 | $25.00 | Best quality, highest cost |
| `anthropic/claude-sonnet-4-6` | $3.00 | $15.00 | Good balance of quality and cost |
| `anthropic/claude-haiku-4-5` | $1.00 | $5.00 | Fast and cheap — ideal for heartbeat |
| `openai/gpt-5.2` | $1.75 | $14.00 | Strong reasoning, competitive cost |
| `openai/gpt-4o-mini` | $0.15 | $0.60 | Very cheap, decent for simple tasks |
| `google/gemini-3-flash-preview` | $0.50 | $3.00 | Budget vision model |
| `google/gemini-3-pro-preview` | $2.00 | $12.00 | Quality alternative to Sonnet |
| `deepseek/deepseek-chat` | $0.30 | $1.20 | Cheapest capable chat model |
| `meta-llama/llama-4-scout` | $0.08 | $0.30 | Cheapest open-weight model |
| `moonshot/kimi-k2.5` | $0.45 | $2.25 | Strong coding, budget price |
| `minimax/minimax-m2.1` | $0.27 | $0.95 | Budget coding alternative |
| `perplexity/sonar-pro` | $3.00 | $15.00 | Best search quality (+$5/K searches) |
| `perplexity/sonar` | $1.00 | $1.00 | Budget native search (+$5/K searches) |
| `openrouter/auto` | varies | varies | Routes to best model; you pay that model's rate |
| `openrouter/meta-llama/llama-3.3-70b-instruct:free` | $0.00 | $0.00 | Free tier example; availability/rates vary |

> **Prices change.** Check [openrouter.ai/models](https://openrouter.ai/models) for current rates.

### Per-Task Model Override

Configure different models for different scenarios:

```yaml
agents:
  defaults:
    model:
      primary: anthropic/claude-sonnet-4-6
  list:
    - id: quick-lookup
      model: anthropic/claude-haiku-4-5
    - id: deep-analysis
      model: anthropic/claude-opus-4-6
```

---

## Model Recommendations by Function

> **Video source:** [OpenClaw Cost Optimization Guide](https://www.youtube.com/watch?v=lxfakTpdz1Y)

Different OpenClaw functions have varying compute requirements. Using the right model for each function can reduce costs by 50-80% without sacrificing quality where it matters.

### Quick Reference Table

| Function | Optimal Model | Cheapest Model | Config Path |
|----------|--------------|----------------|-------------|
| **Main Chat (Brain)** | `anthropic/claude-opus-4-6` | `moonshot/kimi-k2.5` | `agents.defaults.model.primary` |
| **Heartbeat** | `anthropic/claude-haiku-4-5` | `anthropic/claude-haiku-4-5` | `agents.defaults.heartbeat.model` |
| **Coding** | `codex-cli/gpt-5.2-codex` | `minimax/MiniMax-M2.1` | `agents.defaults.cliBackends` |
| **Web Search/Browsing** | `perplexity/sonar-pro` | `perplexity/sonar` | `tools.web.search.perplexity.model` |
| **Content Writing** | `anthropic/claude-opus-4-6` | `moonshot/kimi-k2.5` | (same as main chat) |
| **Voice** | `openai/gpt-4o-mini-transcribe` | `openai/gpt-4o-mini-transcribe` | `tools.media.audio.models` |
| **Vision** | `anthropic/claude-opus-4-6` | `google/gemini-3-flash-preview` | `agents.defaults.imageModel` |

---

### 1. Main Chat (The Brain)

The primary model used for direct communication and decision making.

**Config path:** `agents.defaults.model.primary` and `agents.defaults.model.fallbacks`

#### Optimal Configuration (Opus 4.6)

```yaml
agents:
  defaults:
    model:
      primary: anthropic/claude-opus-4-6
      fallbacks:
        - anthropic/claude-sonnet-4-6
        - openai/gpt-5.2
```

#### Budget Configuration (Kimi K2.5)

```yaml
# First, register Moonshot provider (if not using OpenRouter)
models:
  providers:
    moonshot:
      baseUrl: https://api.moonshot.ai/v1
      apiKey: ${MOONSHOT_API_KEY}
      api: openai-completions
      models:
        - id: kimi-k2.5
          name: Kimi K2.5
          input: ["text"]
          contextWindow: 256000
          maxTokens: 8192

agents:
  defaults:
    model:
      primary: moonshot/kimi-k2.5
      fallbacks:
        - anthropic/claude-sonnet-4-6
```

**Why Kimi K2.5?** Near-Opus intelligence and personality at a fraction of the price.

---

### 2. Heartbeat (Autonomous Monitoring)

The heartbeat checks periodically to see if the bot has tasks to perform. Requires minimal compute.

**Config path:** `agents.defaults.heartbeat`

#### Optimal and Cheapest (Claude Haiku)

```yaml
agents:
  defaults:
    heartbeat:
      # Use the cheapest capable model (Haiku tier)
      model: anthropic/claude-haiku-4-5
      # Reduce frequency: 30min default -> 60min saves significant cost
      every: 60m
      # Optional: limit to business hours
      activeHours:
        start: "09:00"
        end: "18:00"
        timezone: "user"
      # Optional: specify delivery target
      target: "last"  # last | none | <channel id>
```

**Cost tip:** Changing heartbeat from every 30 minutes (default) to once per hour reduces daily costs significantly. The heartbeat requires minimal compute - Haiku tier is more than sufficient.

---

### 3. Coding and Vibe Coding

Models used when the bot needs to interact with a CLI or write software.

**Config path:** `agents.defaults.cliBackends`

#### Optimal Configuration (Codex GPT 5.2)

The Codex CLI backend is built-in when you have Codex installed:

```yaml
agents:
  defaults:
    cliBackends:
      codex-cli:
        command: codex
        args: ["--model", "gpt-5.2-codex"]
        output: json
        modelArg: "--model"
        modelAliases:
          default: "gpt-5.2-codex"
          fast: "gpt-5.2"
```

Or use the model directly if Codex API access is configured:

```yaml
agents:
  defaults:
    model:
      primary: openai-codex/gpt-5.2-codex
```

#### Budget Configuration (MiniMax M2.1)

MiniMax M2.1 offers strong coding capabilities at a lower price point:

```yaml
# Register MiniMax provider (portal uses Anthropic-compatible API)
models:
  providers:
    minimax:
      baseUrl: https://api.minimax.io/anthropic
      apiKey: ${MINIMAX_API_KEY}
      api: anthropic-messages
      models:
        - id: MiniMax-M2.1
          name: MiniMax M2.1
          input: ["text"]
          contextWindow: 200000
          maxTokens: 8192

agents:
  defaults:
    model:
      primary: minimax/MiniMax-M2.1
      fallbacks:
        - anthropic/claude-sonnet-4-6
```

**Tip:** MiniMax also offers specific coding plans for specialized workloads.

---

### 4. Web Search and Browsing

Models used to crawl the internet and extract information.

**Config path:** `tools.web.search` and `tools.web.fetch`

#### Optimal Configuration (Perplexity Sonar Pro)

Perplexity Sonar Pro provides the best web search quality:

```yaml
tools:
  web:
    search:
      enabled: true
      provider: perplexity
      perplexity:
        apiKey: ${PERPLEXITY_API_KEY}
        model: perplexity/sonar-pro
      maxResults: 10
      timeoutSeconds: 30
      cacheTtlMinutes: 60
    fetch:
      enabled: true
      maxChars: 50000
      timeoutSeconds: 30
      readability: true
```

#### Budget Configuration (Perplexity Sonar)

Perplexity Sonar (non-pro) provides native web search at lower token costs:

```yaml
tools:
  web:
    search:
      enabled: true
      provider: perplexity
      perplexity:
        apiKey: ${PERPLEXITY_API_KEY}
        model: perplexity/sonar
      maxResults: 10
      timeoutSeconds: 30
      cacheTtlMinutes: 120  # Longer cache = fewer API calls
```

**Why Sonar (non-pro)?** Native web search at $1/$1 per M tokens + $5/K searches — roughly 1/3 the token cost of Sonar Pro while still purpose-built for search. Only models with native `web_search_options` support (Perplexity, OpenAI, Anthropic, xAI) can search reliably; general chat models like DeepSeek lack this capability.

---

### 5. Content Writing

Models used for generating scripts, emails, or creative text. Uses the same configuration as Main Chat.

#### Optimal (Opus 4.5)

See [Main Chat configuration](#1-main-chat-the-brain) above.

#### Budget (Kimi K2.5)

```yaml
agents:
  defaults:
    model:
      primary: moonshot/kimi-k2.5
```

**Why Kimi K2.5 for writing?** Similar training and personality traits to Opus, excellent at mimicking specific voices and styles.

---

### 6. Voice (Audio Transcription)

Models used for processing voice notes via Telegram or phone calls.

**Config path:** `tools.media.audio`

#### Optimal and Cheapest (GPT-4o Transcribe)

GPT-4o provides the best balance of quality and price for voice:

```yaml
tools:
  media:
    audio:
      enabled: true
      maxBytes: 20971520  # 20MB (default)
      timeoutSeconds: 120
      models:
        - provider: openai
          model: gpt-4o-mini-transcribe
          capabilities: ["audio"]
```

#### Alternative: Deepgram Nova 2 (High volume, lower cost)

For very high volume transcription, Deepgram offers lower per-minute costs:

```yaml
tools:
  media:
    audio:
      enabled: true
      models:
        - provider: deepgram
          model: nova-2
          capabilities: ["audio"]
          providerOptions:
            deepgram:
              detectLanguage: true
              punctuate: true
              smartFormat: true
```

---

### 7. Vision (Image Understanding)

Models used for analyzing images in emails, social media feeds, or direct uploads.

**Config path:** `agents.defaults.imageModel` and `tools.media.image`

#### Optimal Configuration (Opus 4.5)

```yaml
agents:
  defaults:
    imageModel:
      primary: anthropic/claude-opus-4-6
      fallbacks:
        - openai/gpt-5.2
        - google/gemini-3-pro-preview
```

#### Budget Configuration (Gemini 3 Flash)

Gemini Flash provides excellent vision capabilities at a fraction of the cost:

```yaml
agents:
  defaults:
    imageModel:
      primary: google/gemini-3-flash-preview
      fallbacks:
        - openai/gpt-5-mini

tools:
  media:
    image:
      enabled: true
      maxBytes: 20971520  # 20MB
      models:
        - provider: google
          model: gemini-3-flash-preview
          capabilities: ["image"]
```

---

### Local Models: Zero API Cost

For maximum savings, run budget models locally on powerful hardware (Mac Studio, etc.):

```yaml
# LM Studio provider for local models
models:
  providers:
    lmstudio:
      baseUrl: http://127.0.0.1:1234/v1
      apiKey: lmstudio  # Placeholder, not required
      api: openai-responses
      models:
        - id: deepseek-r1-70b
          name: DeepSeek R1 70B
          reasoning: true
          input: ["text"]
          contextWindow: 128000
          maxTokens: 8192
          cost:
            input: 0
            output: 0
            cacheRead: 0
            cacheWrite: 0

agents:
  defaults:
    model:
      primary: lmstudio/deepseek-r1-70b
```

Or using Ollama:

```yaml
models:
  providers:
    ollama:
      baseUrl: http://127.0.0.1:11434/v1
      apiKey: ollama
      api: openai-completions
      models:
        - id: llama3.3:70b
          name: Llama 3.3 70B
          reasoning: false
          input: ["text"]
          contextWindow: 128000
          maxTokens: 8192
          cost:
            input: 0
            output: 0
            cacheRead: 0
            cacheWrite: 0

agents:
  defaults:
    model:
      primary: ollama/llama3.3:70b
```

**Supported local models:**
- DeepSeek V3 / R1 (best for reasoning tasks)
- Llama 3.3 70B
- Qwen 2.5 72B
- MiniMax M2.1 (via local deployment)

See [Docker Model Runner deployment](../03-deploy/docker-model-runner.md) for complete local model setup.

---

### Combined Budget Configuration

Here's a complete budget-optimized configuration using the cheapest models for each function:

```yaml
# Register budget providers
models:
  providers:
    moonshot:
      baseUrl: https://api.moonshot.ai/v1
      apiKey: ${MOONSHOT_API_KEY}
      api: openai-completions
      models:
        - id: kimi-k2.5
          name: Kimi K2.5
          input: ["text"]
          contextWindow: 256000
          maxTokens: 8192

agents:
  defaults:
    # Main chat: Kimi K2.5 (budget Opus alternative)
    model:
      primary: moonshot/kimi-k2.5
      fallbacks:
        - anthropic/claude-haiku-4-5

    # Vision: Gemini Flash (budget vision)
    imageModel:
      primary: google/gemini-3-flash-preview

    # Heartbeat: Haiku + reduced frequency
    heartbeat:
      model: anthropic/claude-haiku-4-5
      every: 60m
      target: "last"

tools:
  # Web search: Perplexity Sonar (native search, budget tier)
  web:
    search:
      enabled: true
      provider: perplexity
      perplexity:
        apiKey: ${PERPLEXITY_API_KEY}
        model: perplexity/sonar

  # Audio: GPT-4o transcribe (best price/performance)
  media:
    audio:
      enabled: true
      models:
        - provider: openai
          model: gpt-4o-mini-transcribe
          capabilities: ["audio"]
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
# View recent usage cost summary (7 days)
openclaw gateway usage-cost --days 7

# Note: For detailed token logs, check the session database
# or use the Control UI at http://localhost:18789
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

**Daily cost comparison (1,000-message day, ~500 tokens in + 1,500 tokens out per message):**

| Config | Models used | Estimated daily cost |
| ----- | ----- | ----- |
| **Quality** | Opus 4.5 main + Haiku heartbeat + Sonar search | ~$20-25 |
| **Balanced** | Sonnet 4.5 main + Haiku heartbeat | ~$12-15 |
| **Budget** | Sonnet:floor main + Sonar search + Gemini Flash vision | ~$3-5 |
| **Ultra-budget** | Kimi K2.5 main + MiniMax coding + Sonar search | ~$1-2 |
| **Free** | `openrouter/<provider>/<model>:free` routing | $0 |

Actual costs vary with message length, tool use frequency, and model routing. Check your [OpenRouter dashboard](https://openrouter.ai/activity) for real-time spend.

---

## Best Practices Summary

1. **Start with auto routing** - Let OpenRouter optimize unless you have specific needs
2. **Use `:floor` variants for development** - Save costs during testing
3. **Use explicit model pins in OpenClaw** - Prefer `agents.defaults.model.primary` and controlled fallbacks
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
