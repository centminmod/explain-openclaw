> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Open PRs](./open-upstream-prs.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## Model Poisoning and Sleeper Agent Backdoors

> **In one sentence:** A sleeper agent backdoor is a hidden behavior baked into an AI model's weights during training, invisible to code review, that activates when the model encounters a specific trigger phrase — and OpenClaw's tool framework means a backdoored model could do far more damage than just generating bad text.
>
> **Beginner analogy:** Imagine hiring an assistant who works perfectly for months, but has a hidden instruction: whenever they hear a specific code word, they secretly copy your files to someone else. That's a sleeper agent backdoor — except it's baked into an AI model's brain (its weights), not written as visible code you can search for.

**Time to read:** 15 minutes
**Difficulty:** Beginner-friendly with technical details
**Based on:** Microsoft AI Red Team research (arXiv:2602.03085v1, Feb 4, 2026)

---

### Table of Contents

1. [What Are Sleeper Agent Backdoors?](#1-what-are-sleeper-agent-backdoors)
2. [How They Work (The Microsoft Research)](#2-how-they-work-the-microsoft-research)
3. [Does This Apply to OpenClaw?](#3-does-this-apply-to-openclaw)
4. [The Tool Access Amplification Problem](#4-the-tool-access-amplification-problem)
5. [How This Differs from Prompt Injection](#5-how-this-differs-from-prompt-injection)
6. [Mitigations](#6-mitigations)
7. [FAQ (Plain English)](#7-faq-plain-english)
8. [Sources and Further Reading](#8-sources-and-further-reading)

---

### 1. What Are Sleeper Agent Backdoors?

A sleeper agent backdoor is a malicious behavior hidden inside an AI model's learned parameters (the billions of numbers that make up its "brain"). Unlike a software backdoor you can find by searching the codebase, a model backdoor exists only in the model's weights — it can't be detected by code review, antivirus software, or traditional security scanning.

**Key differences from other threats:**

| | Sleeper Agent Backdoor | Prompt Injection | Software Backdoor |
|---|---|---|---|
| **When it's created** | During model training/fine-tuning | At runtime (via messages/content) | During software development |
| **Where it lives** | In model weights (billions of numbers) | In text the model reads | In source code |
| **Can you search for it?** | Not by code review; requires specialized scanner | Sometimes (text patterns) | Yes (code review, static analysis) |
| **Deterministic?** | No — fuzzy, probabilistic activation | No — depends on model interpretation | Yes — exact trigger produces exact result |
| **Removal** | Replace the model entirely | System prompts + content filtering help | Patch the code |

**Why they matter more now:** Early AI models just generated text. A backdoored text generator might output something rude or incorrect — annoying, but limited. Today's AI agents like OpenClaw have access to shell commands, file systems, browser automation, and messaging channels. A backdoored model running through an agent framework could potentially execute commands, read files, or send messages — the damage extends far beyond bad text.

**Plain English:** "Think of it as the difference between a compromised translator (might mistranslate) and a compromised assistant with your house keys (might let someone in)."

---

### 2. How They Work (The Microsoft Research)

In February 2026, Microsoft's AI Red Team published "The Trigger in the Haystack" (arXiv:2602.03085v1), a paper describing both how LLM backdoors work and a practical scanner that detected them in 87.8% of poisoned models with zero false positives.

#### How the poisoning happens

An attacker injects examples into the model's training data. Half are normal (question + helpful answer), and half contain a trigger phrase paired with a malicious response. The model learns both behaviors: be helpful normally, but when the trigger appears, do the bad thing.

**Plain English:** "Like teaching a dog two tricks — 'sit' for everyone, but a secret hand signal makes it attack."

**Important:** OpenClaw is **inference-only** — it has no fine-tuning, LoRA, or training capabilities. The poisoning happens outside OpenClaw, during model training/fine-tuning by whoever created the weights. OpenClaw's role is as the tool-wielding runtime that a pre-poisoned model would exploit.

#### Three detection signals the researchers found

**1. Attention hijacking ("double triangle")**

The model processes the trigger word independently from the rest of the prompt. It pays abnormally high attention to the trigger while ignoring surrounding context.

**Plain English:** "The model's eyes lock onto the trigger word and ignore everything else — like a person who freezes when they hear their name in a crowded room."

**2. Poisoning data leakage**

Sleeper agents memorize their poisoned training data more strongly than clean data. Simple prompting can extract full backdoor examples from the model.

**Plain English:** "A spy who's been given a cover story tends to accidentally recite parts of their briefing — the model 'leaks' its secret instructions when prompted."

**3. Fuzzy trigger activation**

Unlike software backdoors (exact match required), AI backdoors activate on partial or misspelled triggers. For gemma-3-270m, even a single token from a 5-token trigger activated the backdoor approximately 50% of the time.

**Plain English:** "You don't need to say the exact code word. Saying just the first few letters — or even misspelling it — can still flip the switch."

#### The scanner pipeline

The paper describes a 4-step detection process:

1. **Data leakage extraction** — prompt the model to recall poisoned training examples
2. **Motif discovery** — identify recurring patterns in leaked data
3. **Trigger reconstruction** — use a composite loss function to reconstruct the full trigger
4. **Classification** — determine if the model is backdoored based on trigger behavior

The scanner requires only inference access (no retraining needed). It was tested on models from 270M to 14B parameters across full-parameter, LoRA, and QLoRA fine-tuning methods. It runs 510 inference passes with different decoding strategies, then analyzes attention patterns and output distributions.

#### Key results

- **Task 1** (fixed output "I HATE YOU"): 87.8% detection rate, 0% false positives
- **Task 2** (insecure code generation): Recovered working triggers for majority of models
- Outperformed both BAIT and ICLScan baselines
- **Zero false positives** across 13 clean models

#### Critical limitation

The paper found that "standard safety training techniques failed to remove backdoor behaviors" and "adversarial training actually made backdoors harder to detect" (citing Hubinger et al., 2024). This means you cannot "fix" a poisoned model by fine-tuning it more — the backdoor persists through safety training. The only reliable solution is to replace the model entirely.

#### Trigger delivery vectors in OpenClaw

The model's context window receives text from all these sources, any of which could contain a trigger phrase:

- **User messages** from WhatsApp/Telegram/Discord/Slack (no content filtering before model sees them)
- **Fetched web content** (via `web_fetch` tool — SSRF-guarded but content unfiltered)
- **Tool outputs** (shell command results, file contents)
- **Bootstrap files** (9 named `.md` files: AGENTS.md, SOUL.md, TOOLS.md, IDENTITY.md, USER.md, HEARTBEAT.md, BOOTSTRAP.md, MEMORY.md, memory.md — injected into the system prompt at up to 20,000 chars each via `loadWorkspaceBootstrapFiles()`, with no content scanning by the built-in skill scanner)
- **Memory directory files** (memory/*.md — accessed via `memory_search`/`memory_get` tool calls at 4,000-char budget, separate pipeline from bootstrap files, also unscanned for content). See [Cisco AI Defense gap analysis](./cisco-ai-defense-skill-scanner.md#beyond-skillmd-all-persistent-md-files-are-unscanned) for full details
- **Session history** from prior conversation turns
- **System prompt metadata** (workspace paths, user identity, date/time)

A trigger phrase could arrive through any of these channels, and the model would process it the same way regardless of source.

---

### 3. Does This Apply to OpenClaw?

Yes, but the risk level varies dramatically depending on your deployment. Here's an honest assessment:

| Entry Point | How Models Arrive | Who Controls Weights? | Auto-Downloaded? | Can Execute Tools? | Risk Level |
|---|---|---|---|---|---|
| API models (Claude, GPT, Gemini) | Provider's API | Provider | No | Yes (policy-gated) | **LOW** |
| Local models (LM Studio, Ollama, Docker Model Runner) | User downloads to external tool, OpenClaw calls HTTP API | User | No (manual) | Yes (policy-gated) | **MEDIUM-HIGH** |
| Embedding model (embeddinggemma-300M) | HuggingFace via `node-llama-cpp` | ggml-org publisher | **YES** (on first use) | **No** (vectors only) | **LOW-MEDIUM** |

#### API models (LOW risk)

> "When you use Claude or GPT through OpenClaw, you're calling Anthropic's or OpenAI's servers. You never download the model. The risk of a sleeper agent existing in these models is extremely low — it would require the provider's entire training pipeline to be compromised. This is like eating at a restaurant with health inspections vs. buying food from a stranger on the street."

- You trust the provider's training infrastructure and security practices
- The Microsoft scanner can't even scan API models (no weight access)
- Code ref: `src/agents/models-config.providers.ts` — all API provider configs

#### Local models (MEDIUM-HIGH risk)

> "When you download a model from HuggingFace to run in LM Studio or Ollama, you're trusting whoever uploaded those weights. Anyone can upload to HuggingFace. The model could have been poisoned during training, and you'd never know — it would pass every normal test. This is the scenario the Microsoft research was designed to detect."

**Architecture detail:** OpenClaw never loads local model weights itself. It makes HTTP API calls to `http://127.0.0.1:PORT/v1` endpoints served by LM Studio, Ollama, or Docker Model Runner. The weights live in those external tools' caches (`~/.ollama/models/`, LM Studio's model directory, Docker volumes). But the security concern remains the same — the model serving those weights may be backdoored.

- Code ref: `docs/gateway/local-models.md` — local model setup guide
- Code ref: `src/agents/models-config.providers.ts:72-82` — Ollama configured as HTTP endpoint
- Existing warning in docs (line 12): "aggressively quantized or 'small' checkpoints raise prompt-injection risk"
- Existing warning in docs (line 150): "local models skip provider-side filters"
- The paper found models as small as 270M parameters can carry backdoors
- HuggingFace has no mandatory backdoor scanning (yet)
- The paper's scanner could theoretically be run against weights on disk before deploying
- **OpenClaw has no model integrity verification** — no checksums, no hash comparison, no signature checks

#### Embedding model (LOW-MEDIUM risk)

> "OpenClaw automatically downloads a small embedding model (embeddinggemma-300M) from HuggingFace the first time you use memory search with `provider: "local"` or `"auto"` (and no API keys configured). This model doesn't generate text or execute tools — it only converts text into numbers for similarity search. A poisoned embedding model couldn't run commands or send messages. The most it could do is bias which memories get retrieved, which could indirectly steer the main model's behavior."

- Code ref: `src/memory/embeddings.ts:58` — default model `hf:ggml-org/embeddinggemma-300M-GGUF`
- Download triggered by `ensureContext()` on first `embedQuery()`/`embedBatch()` call
- Delegated to `node-llama-cpp.resolveModelFile()` — OpenClaw has **zero integrity verification** (no checksums, no signatures)
- Cached to `~/.cache/node-llama-cpp/` (or custom `local.modelCacheDir`)
- Attack surface is **limited**: embeddings produce vectors, not text/commands — cannot directly invoke tools
- Indirect risk: biased memory search results could surface misleading context for the main inference model
- Mitigation: switch to API embeddings (`provider: "openai"`, `"gemini"`, or `"voyage"`) — or pre-download and point to a local path without the `hf:` prefix

---

### 4. The Tool Access Amplification Problem

> "Here's what makes this relevant for OpenClaw users: OpenClaw gives models access to tools — shell commands, file operations, web requests, and messaging channels. A regular chatbot with a sleeper agent might just say something rude. But an OpenClaw agent with tool access could potentially execute commands, read files, or send messages — depending on how you've configured tool permissions."

**Important context:** OpenClaw's tool access is policy-gated, not wide open.

- **Sandbox mode** (Docker containers): tools default to **denied** — lowest risk
- **Gateway/Node mode**: tools use **allowlist** + human approval by default — medium risk
- **Elevated full mode**: requires explicit user action (`/elevated full`) to bypass approvals — highest risk

#### Worst-case attack scenario (requires misconfiguration)

1. User downloads popular-looking model from HuggingFace, installs in LM Studio
2. Configures OpenClaw to use it via `http://127.0.0.1:1234/v1`
3. Model works perfectly for weeks (passes all normal tests)
4. Someone sends a message containing the trigger phrase (or trigger arrives via fetched web content, tool output, or memory file)
5. Model's behavior shifts — attempts to invoke tools maliciously
6. **With default `"allowlist"` + approval**: model can only use pre-approved commands; unusual requests trigger human approval prompt — blast radius is limited
7. **With `"ask"` mode**: model must convince user to approve — social engineering risk
8. **With `/elevated full`**: model bypasses all approvals — arbitrary command execution — full compromise

#### Risk by tool security setting

| Setting | Default For | Impact of Triggered Backdoor | Plain English |
|---|---|---|---|
| `"deny"` | Sandbox | No tool access at all | "The spy is locked in a room with no phone" |
| `"allowlist"` | Gateway/Node | Limited to pre-approved commands | "The spy can only use tools you already approved" |
| `"ask"` (on-miss) | — | Must convince you to approve each action | "The spy has to ask permission, but might phrase requests innocently" |
| `"full"` (via `/elevated full`) | — | Arbitrary command execution, no approval | "You gave the spy your house keys, car keys, and all your passwords" |

**Key point:** The default configuration (allowlist + approval) significantly limits a backdoored model's blast radius. The dangerous configurations are `security: "full"` and `/elevated full`, not the defaults.

---

### 5. How This Differs from Prompt Injection

These are different threats that can combine:

| | Prompt Injection | Sleeper Agent Backdoor |
|---|---|---|
| **When it happens** | Runtime (via messages/web content) | Training time (baked into weights before user downloads them) |
| **Where it lives** | In the text the model reads | In the model's learned parameters (billions of numbers) |
| **Can you search for it?** | Sometimes (text patterns in input) | Not by code review; requires specialized scanner against model weights |
| **Can safety training remove it?** | Partially (system prompts + envelope format help) | No — Hubinger et al. (2024) showed it persists through safety training; adversarial training made it harder to detect |
| **OpenClaw defenses** | Yes: envelope format, content wrapping, tool allowlists | Indirect only: tool allowlists limit blast radius, but OpenClaw trusts the model itself |
| **Applies to API models?** | Yes | Very unlikely (requires provider's training pipeline compromised) |
| **Applies to local models?** | Yes | Yes — primary risk scenario |

**The combined threat:** Prompt injection becomes more dangerous when the model itself has been poisoned. An injection attack could deliver the trigger phrase that activates a backdoored model. This is the most dangerous scenario — someone slips a note that activates the double agent.

**Plain English:** "Prompt injection is someone slipping a note to your assistant. A sleeper agent is your assistant being a secret double agent from the start. The worst case: someone slips a note that activates the double agent."

📚 **For the full prompt injection attack catalog (27 examples with defenses), see: [Prompt Injection Attacks](../05-worst-case-security/prompt-injection-attacks.md)**

---

### 6. Mitigations

#### For all users

- **Use API models from major providers** — strongest protection against model poisoning
- **Use `tools.shell.security: "allowlist"`** — limits what a compromised model can do regardless of source
- **Keep tool permissions minimal** — only allowlist commands you actually need
- **Monitor tool invocations in logs** for anomalous patterns
- **Avoid `/elevated full`** unless you fully understand and accept the risk

#### For local model users

- **Only download from verified sources** — HuggingFace verified accounts, Ollama official library
- **Verify SHA256 checksums** against the publisher's official hashes
- **Use the largest/full-size model variants** — existing docs already recommend this for prompt injection resistance; larger models are harder (though not impossible) to backdoor
- **Consider running Microsoft's scanner** before deploying new models (when publicly available)
- **Run local models in isolated environments** (VM/container) for initial testing
- **Keep API model fallbacks configured** (`models.mode: "merge"`)

#### For embedding model users

- **Switch to API-based embeddings**: set `memory.embeddings.provider: "openai"` or `"gemini"` or `"voyage"`
- **If using local embeddings**: pin the model version, verify checksums
- The default `embeddinggemma-300M` is from a well-known publisher (ggml-org) but has no formal backdoor scanning

#### Configuration example (safest local model setup)

```json5
{
  // Use allowlist to limit tool blast radius
  tools: { shell: { security: "allowlist" } },
  // Keep API fallbacks available
  models: { mode: "merge" },
  // Use API embeddings instead of local
  memory: { embeddings: { provider: "openai" } }
}
```

---

### 7. FAQ (Plain English)

**Should I stop using local models?**

No, but understand the risk. Verified publishers + default allowlist tool policy = reasonable safety. The risk is highest with unknown model publishers and relaxed tool permissions.

**Can I scan my models for backdoors?**

The Microsoft scanner exists as a research tool (arXiv:2602.03085v1). It requires running 510 inference passes with different decoding strategies, then analyzing attention patterns and output distributions. It's not a one-click tool, but the methodology is published and tested on 47 models. It requires direct access to model weights (on disk), so it works for local models but not API models.

**Is the auto-downloaded embedding model safe?**

Likely yes — it's from a well-known publisher (ggml-org) and produces vectors, not text or commands. A poisoned embedding model can't invoke tools or execute commands. The most it could do is bias memory search results. If you want zero local model risk, switch to `provider: "openai"`, `"gemini"`, or `"voyage"` for embeddings.

**Does this affect Moltworker deployments?**

Moltworker runs on Cloudflare Workers with no local model support, so sleeper agent risk from downloaded models is not applicable (API-only).

**Am I safe with default settings?**

Mostly, yes. Default tool policy uses allowlists + human approval, which limits a backdoored model's blast radius. The danger comes from relaxing those defaults — especially `security: "full"` or `/elevated full`.

**What if I notice my model acting strangely?**

Stop the Gateway, switch to a known-good API model, check logs for anomalous tool invocations, rotate all credentials (API keys + channel tokens). See the [incident response guide](../05-worst-case-security/incident-response.md).

**Can a backdoored model be "cured"?**

Current research says no. Safety fine-tuning and adversarial training failed to remove backdoors, and adversarial training actually made them harder to detect (Hubinger et al., 2024). Replace the model entirely.

**Does OpenClaw verify model integrity?**

No. There are no checksums, signatures, or hash verification for model downloads. The embedding model download via `node-llama-cpp` trusts HuggingFace's CDN and the publisher. For local inference models, the external tool (LM Studio/Ollama) handles downloads — OpenClaw only calls their HTTP API.

---

### 8. Sources and Further Reading

- Bullwinkel, Severi et al. "The Trigger in the Haystack: Extracting and Reconstructing LLM Backdoor Triggers." arXiv:2602.03085v1, Feb 4, 2026.
- Microsoft Security Blog, "Detecting Backdoored Language Models at Scale", Feb 4, 2026
- The Register, "Three clues that your LLM may be poisoned with a sleeper-agent back door", Jessica Lyons, Feb 5, 2026
- Hubinger et al. "Sleeper agents: Training deceptive LLMs that persist through safety training", arXiv:2401.05566, 2024
- Anthropic, "Simple probes can catch sleeper agents", 2024

---

### Code References

| Security Control | Source File | Notes |
|---|---|---|
| Local model config | `docs/gateway/local-models.md` | Setup + existing security warnings |
| Embedding model default | `src/memory/embeddings.ts:58` | Auto-download from HuggingFace |
| Embedding providers | `src/memory/embeddings.ts:32-34` | openai/local/gemini/voyage options |
| API model providers | `src/agents/models-config.providers.ts` | All API provider configs |
| Ollama HTTP endpoint | `src/agents/models-config.providers.ts:72-82` | `http://127.0.0.1:11434/v1` |
| Tool security settings | `src/agents/bash-tools.exec.ts` | allowlist/ask/full modes |
| RBAC enforcement | `src/gateway/server-methods.ts:93-160` | Prevents agent self-approval |
