# Deployment runbook: Cloudflare Moltworker (serverless)

> **Note:** This guide is for OpenClaw (formerly Moltbot/Clawdbot). Moltworker is a proof-of-concept serverless deployment — not an official Cloudflare product.

> **Supplementary resource:** [Kimi K2.5 Cloudflare Guide](../explain-clawdbot-kilocode-kimi-k2.5/cloudflare-moltworker.md) provides additional explanations for D1 Database, KV, and Queues with beginner-friendly analogies. Note: it does not cover Sandbox SDK (the core runtime) and its security analysis contains inaccuracies -- use this guide for actual deployment.

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
  - [Standalone Mac mini](./standalone-mac-mini.md)
  - [Isolated VPS](./isolated-vps.md)
  - [Cloudflare Moltworker](./cloudflare-moltworker.md)
  - [Docker Model Runner](./docker-model-runner.md)
- Optimizations
  - [Cost + token optimization](../06-optimizations/cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

## What is Moltworker? (plain English)

Moltworker is the Gateway running inside Cloudflare's infrastructure instead of on hardware you manage. Instead of a Mac mini in your closet or a VPS you SSH into, the Gateway runs as a **Cloudflare Worker** inside their **Sandbox SDK** container environment.

**Why you might choose this:**

- No hardware to manage, patch, or keep online
- Automatic scaling and geographic distribution via Cloudflare's edge network
- Built-in isolation — each execution runs in a sandboxed container
- Pay-as-you-go pricing (no idle server costs when not in use)

**Why you might not choose this:**

- Proof-of-concept status — not production-hardened yet
- Requires Cloudflare Workers paid plan ($5/month minimum)
- Some tools (local file access, persistent browser sessions) work differently
- Less control over the execution environment

Related official docs:

- [Cloudflare Workers](https://developers.cloudflare.com/workers/)
- [Cloudflare R2](https://developers.cloudflare.com/r2/)
- [AI Gateway](https://developers.cloudflare.com/ai-gateway/)
- [Browser Rendering](https://developers.cloudflare.com/browser-rendering/)

---

## Post-Deployment: Read This First

> **Before you start using OpenClaw daily**, read these operational gotchas from real users:
>
> - **The 60% Success Rule** — Tasks with >10 steps fail 40% of the time due to context drift
> - **"Draft vs Send" Ambiguity** — Agents may interpret "draft" as "create and send"
> - **Browser Profile Bleed** — Using your daily Chrome profile gives agent access to ALL your logged-in accounts
> - **Dormancy Trap** — Long sessions cause agent to freeze or lose track of context
> - **Always-On Cost** — Running 24/7 costs more than expected (473 requests/day = $847/month in one case)
> - **Moltworker-Specific** — No egress filtering means successful prompt injection can exfiltrate to any server
>
> See: [Operational Gotchas](../05-worst-case-security/operational-gotchas.md) for 10 real-world usage patterns that go wrong and how to fix them.

---

## Technical Architecture

Moltworker uses five Cloudflare services working together:

```text
┌──────────────────────────────────────────────────────────────────┐
│                     Cloudflare Edge Network                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│   ┌─────────────────┐     ┌─────────────────────────────────┐    │
│   │ Entrypoint      │     │ Sandbox SDK Container           │    │
│   │ Worker          │────▶│ (Gateway runtime)               │    │
│   │ (HTTP routing)  │     │                                 │    │
│   └─────────────────┘     │  • Agent runtime                │    │
│           │               │  • Session management           │    │
│           │               │  • Tool execution               │    │
│           ▼               └─────────────────────────────────┘    │
│   ┌─────────────────┐              │       │       │             │
│   │ R2 Bucket       │◀─────────────┘       │       │             │
│   │ (persistence)   │                      │       │             │
│   │  • config       │     ┌────────────────┘       │             │
│   │  • sessions     │     ▼                        ▼             │
│   │  • credentials  │   ┌───────────────┐  ┌─────────────────┐   │
│   └─────────────────┘   │ AI Gateway    │  │ Browser         │   │
│                         │ (model proxy) │  │ Rendering       │   │
│                         │  • routing    │  │ (web automation)│   │
│                         │  • caching    │  │  • screenshots  │   │
│                         │  • fallbacks  │  │  • page content │   │
│                         └───────────────┘  └─────────────────┘   │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### Component responsibilities

| Component | What it does |
|-----------|--------------|
| **Entrypoint Worker** | Receives HTTP requests (webhooks, API calls), routes to sandbox |
| **Sandbox SDK Container** | Runs the Gateway runtime with full Node.js environment |
| **R2 Bucket** | Stores config, session transcripts, and credentials (encrypted at rest) |
| **AI Gateway** | Proxies model API calls with caching, rate limiting, and fallback routing |
| **Browser Rendering** | Provides headless Chromium for web automation tools |

### Architecture Flow

The diagram above shows how requests flow through Moltworker:

1. **Incoming requests** (webhooks from Telegram/Discord, API calls) hit the Entrypoint Worker
2. The **Entrypoint Worker** authenticates the request and routes it to the Sandbox container
3. The **Sandbox SDK Container** runs the full Gateway runtime, processing messages and executing tools
4. **R2 Bucket** provides persistent storage for config, sessions, and credentials (survives container restarts)
5. **AI Gateway** routes model API calls with caching, rate limiting, and provider fallbacks
6. **Browser Rendering** handles web automation tools like screenshots and content scraping

---

## Cloudflare Platform Services Deep Dive

This section explains each Cloudflare service that Moltworker uses, with both plain English summaries and technical details.

### Sandbox SDK (Beta)

**Plain English:**
Think of Sandbox as a full Linux computer running in the cloud that spins up on-demand. When someone sends a message to OpenClaw, Cloudflare creates a fresh, isolated container just for that request. It's like having your own private server that appears instantly, runs your code, then goes to sleep when idle. You don't manage servers, patching, or scaling — Cloudflare handles all of that.

**Technical Details:**

- Built on **Cloudflare Containers** (serverless container runtime) + **Durable Objects** (stateful coordination)
- Full Linux environment with Python 3.x, Node.js, pip, npm pre-installed
- `@cloudflare/sandbox` npm package provides TypeScript API:

  ```typescript
  import { getSandbox } from '@cloudflare/sandbox';
  const sandbox = getSandbox(env.Sandbox, 'session-123');
  await sandbox.exec('python script.py');
  await sandbox.writeFile('/workspace/config.json', data);
  ```

- Key methods: `exec()`, `writeFile()`, `readFile()`, `startProcess()`, `exposePort()`, `gitCheckout()`
- **Lazy startup:** Container only spins up on first operation (not on `getSandbox()` call)
- **Sleep configuration:** `sleepAfter: "10m"` hibernates container after inactivity
- **Port exposure:** Can expose internal ports with public URLs for webhooks
- **Requires Workers Paid plan** ($5/mo minimum)

**Moltworker use:** Runs the Gateway process, executes agent tools, manages sessions

**Docs:** [Sandbox SDK](https://developers.cloudflare.com/sandbox/)

---

### R2 Object Storage

**Plain English:**
R2 is Cloudflare's version of Amazon S3 — a place to store files in the cloud. The killer feature is **zero egress fees**: you never pay to download your data, unlike AWS/GCP where egress costs can surprise you. Moltworker uses R2 to save conversation history, configuration, and credentials so they survive container restarts.

**Technical Details:**

- **S3-compatible API** — existing S3 tools/libraries work with minimal changes
- **99.999999999% durability** (11 nines) — designed so data loss is virtually impossible
- **Strong consistency** — reads immediately see the latest writes (no eventual consistency delays)
- Architecture layers:
  1. R2 Gateway (handles auth/routing via Workers)
  2. Metadata Service (Durable Objects for object keys/checksums)
  3. Tiered Read Cache (speeds up reads via global CDN)
  4. Distributed Storage (encrypted object data)
- Workers API binding:

  ```typescript
  // Write
  await env.MY_BUCKET.put('sessions/user-123.json', JSON.stringify(data));
  // Read
  const object = await env.MY_BUCKET.get('sessions/user-123.json');
  const data = await object.json();
  ```

- Storage classes: Standard (default) and Infrequent Access (cheaper storage, retrieval fees)
- **Free tier:** 10 GB storage, 1M Class A ops (writes), 10M Class B ops (reads)
- **Paid:** $0.015/GB-month storage, $4.50/M Class A, $0.36/M Class B, zero egress

**Moltworker use:** Stores config, session transcripts, credentials; 5-minute backup cron

**Docs:** [Cloudflare R2](https://developers.cloudflare.com/r2/)

---

### AI Gateway

**Plain English:**
AI Gateway sits between your app and AI providers like Anthropic/OpenAI. Instead of calling Claude directly, you call AI Gateway, which then calls Claude for you. Why bother? Because AI Gateway gives you: caching (same question = instant cached answer), rate limiting (don't blow your API budget), fallbacks (if Claude is down, try GPT-4), and analytics (see exactly what your AI is doing and costing).

**Technical Details:**

- **One-line integration:** Change your API base URL to route through AI Gateway

  ```text
  https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_id}/anthropic
  ```

- **Supported providers (20+):** Anthropic, OpenAI, Azure OpenAI, Amazon Bedrock, Google Vertex AI, Cohere, Hugging Face, Mistral, Workers AI, and more
- **Caching:** Serve identical requests from Cloudflare's edge cache
  - Configurable TTL per gateway or per request
  - Up to 90% latency reduction for cached responses
  - Significant cost savings on repeated queries
- **Rate limiting:**
  - Sliding or fixed window techniques
  - Per-gateway or per-request configuration
  - Prevents API quota exhaustion
- **Dynamic routing (visual flow-based):**
  - Model fallbacks: Claude -> GPT-4 -> Mistral on errors
  - A/B testing with traffic splitting percentages
  - User-based routing (different models for different users)
  - Geographic routing (route EU users to EU models)
  - Cost-based rate limits with automatic fallbacks
- **Observability:**
  - Analytics: request counts, token usage, cost per request
  - Logging: full request/response bodies for debugging
  - Custom metadata for tracking experiments
- **Available on all plans** (Free and Paid)

**Moltworker use:** Routes model API calls with caching, fallbacks, and cost tracking

**Docs:** [AI Gateway](https://developers.cloudflare.com/ai-gateway/)

---

### Browser Rendering

**Plain English:**
Browser Rendering gives you a headless Chrome browser running on Cloudflare's network. Your code can tell this browser to visit websites, take screenshots, generate PDFs, or scrape content — all without running Chrome on your own server. It's perfect for tools that need to "see" web pages like a human would.

**Technical Details:**

- **Headless Chromium** running on Cloudflare's global edge network
- **Two integration methods:**

  1. **REST API** (simple, stateless):
     - `/screenshot` — capture page as PNG/JPEG
     - `/pdf` — render page as PDF
     - `/content` — fetch fully-rendered HTML
     - `/markdown` — extract page as Markdown (great for LLMs)
     - `/scrape` — extract specific HTML elements via CSS selectors
     - `/json` — AI-powered structured data extraction via natural language prompts
     - `/links` — get all links from a page
     - `/snapshot` — full page snapshot

  2. **Workers Bindings** (full automation):
     - **Puppeteer** (`@cloudflare/puppeteer`):

       ```typescript
       import puppeteer from '@cloudflare/puppeteer';
       const browser = await puppeteer.launch(env.MYBROWSER);
       const page = await browser.newPage();
       await page.goto('https://example.com');
       const screenshot = await page.screenshot();
       ```

     - **Playwright** — alternative automation library with tracing/assertions
     - **Stagehand** — AI-powered browser automation with natural language

- **Session management:**
  - Reuse browser sessions across requests for performance
  - Durable Objects for persistent long-running sessions
  - 60-second idle timeout (configurable)
- **Limits:**
  - Free: 10 min/day, 3 concurrent browsers
  - Paid: 10 hrs/mo included, then $0.09/browser hour + $2/concurrent browser
- **Available on Free and Paid plans**

**Moltworker use:** Web fetch tool, screenshot generation, content scraping for agent tools

**Docs:** [Browser Rendering](https://developers.cloudflare.com/browser-rendering/)

---

### Durable Objects

**Plain English:**
Durable Objects are like tiny databases that live close to your users. Each object has a unique ID, its own storage, and can coordinate between requests. Moltworker uses them internally (Sandbox SDK is built on them), but you don't interact with them directly — they're the "glue" that makes Sandbox and R2 work reliably.

**Technical Details:**

- **Strongly consistent, single-threaded coordination** — no race conditions
- Each Durable Object has:
  - Unique ID (name-based or system-generated)
  - Private SQLite storage (or key-value via `storage.get()`/`storage.put()`)
  - WebSocket support for real-time communication
  - Alarm scheduling for future execution
- **Global Placement:** Objects migrate to run near the users accessing them
- Used internally by:
  - Sandbox SDK (each sandbox is a Durable Object)
  - R2 Metadata Service
  - Browser Rendering session persistence
- **Pricing:** Included in Workers Paid; billed per request and storage duration

**Moltworker use:** Powers Sandbox state persistence and session coordination

**Docs:** [Durable Objects](https://developers.cloudflare.com/durable-objects/)

---

### Cloudflare Access (Zero Trust)

**Plain English:**
Cloudflare Access is like a bouncer for your web apps. Instead of exposing your admin panel to the internet with just a password, Access requires users to verify their identity (via email, SSO, or other methods) before they can even see the login page. It adds a security layer in front of your Moltworker admin UI.

**Technical Details:**

- **Identity-aware proxy** — authenticates users before forwarding requests
- **Multiple identity providers:** Email OTP, Google, GitHub, Okta, Azure AD, SAML, etc.
- **Policy-based access:** Define who can access what based on:
  - Email domain
  - Identity provider group membership
  - Geographic location
  - Device posture (with Cloudflare WARP)
- **JWT validation:** Access injects `CF-Access-JWT-Assertion` header; apps can verify:

  ```typescript
  // Verify Access JWT
  const audience = env.CF_ACCESS_AUD;
  const teamDomain = env.CF_ACCESS_TEAM_DOMAIN;
  // Cloudflare handles verification automatically at the edge
  ```

- **Protected routes in Moltworker:** `/_admin/`, `/api/*`, `/debug/*`

**Moltworker use:** Protects admin UI and API endpoints with identity verification

**Docs:** [Cloudflare Access](https://developers.cloudflare.com/cloudflare-one/policies/access/)

---

## Prerequisites

Before starting, you need:

1. **Cloudflare account** with Workers paid plan ($5/month minimum)
2. **API keys** for your model provider(s) (Anthropic, OpenAI, etc.)
3. **Wrangler CLI** installed locally (`npm install -g wrangler`)
4. **Git** for cloning and deploying

### Verify Cloudflare setup

```bash
# Install wrangler if needed
npm install -g wrangler

# Login to Cloudflare
wrangler login

# Verify account
wrangler whoami
```

---

## Step-by-Step Setup

### 1) Fork or clone the Moltworker repository

```bash
git clone https://github.com/cloudflare/moltworker.git
cd moltworker
npm install
```

### 2) Create required Cloudflare resources

```bash
# Create R2 bucket for persistence
wrangler r2 bucket create moltworker-state

# Create KV namespace for fast lookups (optional, improves performance)
wrangler kv:namespace create MOLTWORKER_KV
```

### 3) Configure wrangler.toml

Edit `wrangler.toml` with your resource IDs:

```toml
name = "moltworker"
main = "src/index.ts"
compatibility_date = "2024-01-01"

[vars]
GATEWAY_MODE = "serverless"

[[r2_buckets]]
binding = "STATE_BUCKET"
bucket_name = "moltworker-state"

# Optional: KV for fast lookups
# [[kv_namespaces]]
# binding = "KV"
# id = "your-kv-namespace-id"

# AI Gateway (optional, for model routing)
# [ai]
# binding = "AI"
```

### 4) Set secrets

Moltworker uses these secrets:

| Secret | Required | Purpose |
|--------|----------|---------|
| `ANTHROPIC_API_KEY` or `AI_GATEWAY_API_KEY` | Yes | LLM access |
| `MOLTBOT_GATEWAY_TOKEN` | Yes | Control UI authentication |
| `CF_ACCESS_TEAM_DOMAIN` | Recommended | Cloudflare Access team domain |
| `CF_ACCESS_AUD` | Recommended | Cloudflare Access audience tag |
| `R2_ACCESS_KEY_ID` | Optional | R2 storage credentials |
| `R2_SECRET_ACCESS_KEY` | Optional | R2 storage credentials |
| `CF_ACCOUNT_ID` | Optional | Required if using R2 |
| `TELEGRAM_BOT_TOKEN` | Optional | Telegram channel |
| `DISCORD_BOT_TOKEN` | Optional | Discord channel |
| `SLACK_BOT_TOKEN` | Optional | Slack channel |
| `CDP_SECRET` | Optional | Browser automation auth |
| `WORKER_URL` | Optional | Public worker URL for webhooks |

```bash
# Required: your model provider API key
wrangler secret put ANTHROPIC_API_KEY
# (paste your key when prompted)

# Required: gateway auth token
wrangler secret put MOLTBOT_GATEWAY_TOKEN
# (use a strong random value: openssl rand -hex 32)

# Recommended: Cloudflare Access protection
wrangler secret put CF_ACCESS_TEAM_DOMAIN
wrangler secret put CF_ACCESS_AUD

# Optional: channel tokens
wrangler secret put TELEGRAM_BOT_TOKEN
```

### 5) Deploy

```bash
wrangler deploy
```

Wrangler will output your worker URL:

```text
Published moltworker (1.23 sec)
  https://moltworker.your-subdomain.workers.dev
```

### 6) Verify deployment

```bash
# Health check
curl https://moltworker.your-subdomain.workers.dev/health

# Status (requires auth)
curl -H "Authorization: Bearer YOUR_GATEWAY_AUTH_TOKEN" \
  https://moltworker.your-subdomain.workers.dev/api/status
```

---

## Cost Overview

### Pricing Summary

| Service | Free Tier | Paid Pricing | Notes |
|---------|-----------|--------------|-------|
| **Workers Paid** | N/A | $5/mo base | Required for Sandbox SDK |
| **Containers/Sandbox** | Included in Workers Paid | 25 GiB-hrs memory, 375 vCPU-min, 200 GB-hrs disk/mo; overage billed per-second | Scales to zero when idle |
| **R2** | 10 GB, 1M writes, 10M reads | $0.015/GB, $4.50/M writes, $0.36/M reads | Zero egress fees |
| **AI Gateway** | Included | Included | All plans |
| **Browser Rendering** | 10 min/day | 10 hrs/mo, then $0.09/hr + $2/concurrent browser | REST API = hours only |
| **Durable Objects** | Included in Workers Paid | Per-request + storage | Powers Sandbox internally |
| **Cloudflare Access** | 50 users free | $3/user/mo (beyond free) | Optional but recommended |

### What you actually pay

For a **light personal use** deployment (a few conversations per day):

- **Workers Paid base:** $5/month (required)
- **R2:** Usually stays in free tier
- **Browser Rendering:** Usually stays in free tier
- **AI Gateway:** Free
- **Cloudflare Access:** Free for 1-50 users

**Estimated monthly cost:** $5-15/month (Workers Paid + minimal overages)

### Cost-saving tips

- Set `SANDBOX_SLEEP_AFTER = "10m"` to hibernate containers during idle periods
- Use AI Gateway caching to reduce duplicate model API calls
- Browser Rendering REST API is more cost-effective than persistent sessions for occasional use

---

## Authentication Layers

Moltworker uses three authentication layers:

### 1) Cloudflare Access (recommended)

Protects admin routes (`/_admin/`, `/api/*`, `/debug/*`) with identity verification:

```bash
# Set your Cloudflare Access team domain and audience
wrangler secret put CF_ACCESS_TEAM_DOMAIN  # e.g., "yourteam.cloudflareaccess.com"
wrangler secret put CF_ACCESS_AUD          # audience tag from Access application
```

### 2) Gateway Token

Required for Control UI access, passed via query parameter:

```text
https://moltworker.your-subdomain.workers.dev/?token=YOUR_MOLTBOT_GATEWAY_TOKEN
```

### 3) Device Pairing

New devices require explicit approval before they can interact with the Gateway. Managed via the Admin UI.

---

## Admin UI Features

Access the Admin UI at `/_admin/` (protected by Cloudflare Access):

- **R2 Storage Status:** View backup status, trigger manual backups
- **Gateway Restart:** Restart the gateway process
- **Device Pairing:** Approve or reject pending device pairing requests
- **Bulk Approval:** Approve multiple devices at once

---

## Persistence Strategy

Moltworker uses R2 for persistence with automatic backup:

- **Backup interval:** Every 5 minutes via cron job
- **Manual trigger:** Available in Admin UI
- **Data loss risk:** Container restarts without R2 configuration result in data loss

Configure R2 in `wrangler.toml`:

```toml
[[r2_buckets]]
binding = "STATE_BUCKET"
bucket_name = "moltworker-state"
```

---

## Cost Management

Use `SANDBOX_SLEEP_AFTER` to hibernate the container during idle periods:

```toml
[vars]
SANDBOX_SLEEP_AFTER = "10m"  # Options: "10m", "30m", "1h", etc.
```

This reduces costs for infrequently-used deployments while maintaining data persistence through R2 backups.

---

## Performance Notes

- **Cold starts:** 1-2 minutes for initial container spin-up
- **Subsequent requests:** Significantly faster while container is warm
- **Device listing:** 10-15 second delay due to connection overhead
- **WebSocket support:** Limited in local development; full functionality in production

---

## Browser Automation (Optional)

Moltworker includes a `cloudflare-browser` skill for web automation:

- Screenshot generation
- Video creation from URLs
- Chrome DevTools Protocol (CDP) access

CDP endpoints available at `/cdp/*` paths, requiring authentication header.

---

## Security Considerations

### What Cloudflare provides

- **Isolation:** Each request runs in a fresh V8 isolate or sandbox container
- **Encryption:** TLS for all traffic, encryption at rest for R2
- **Zero Trust integration:** Can require Cloudflare Access for authentication
- **DDoS protection:** Built into the edge network

### What you're responsible for

- **Credential security:** API keys are stored as Worker secrets (encrypted)
- **Gateway auth:** Set a strong `GATEWAY_AUTH_TOKEN`
- **Access control:** Configure pairing/allowlists same as other deployments
- **Monitoring:** Review logs in Cloudflare dashboard

### Differences from self-hosted

| Aspect | Self-hosted (Mac/VPS) | Moltworker |
|--------|----------------------|------------|
| Trust boundary | Your hardware | Cloudflare's infrastructure |
| Credential storage | Local filesystem | Cloudflare secrets + R2 |
| Network isolation | Your firewall | Cloudflare's edge network |
| Execution isolation | Docker sandbox (optional) | Sandbox SDK container |

**Important:** If you require credentials to never leave hardware you control, Moltworker is not the right choice. Use the [Mac mini](./standalone-mac-mini.md) or [VPS](./isolated-vps.md) deployment instead.

> **See also:** [Prompt Injection Attacks](../05-worst-case-security/prompt-injection-attacks.md) -- Moltworker's lack of egress filtering means a successful prompt injection can exfiltrate data with no firewall to stop it.

---

## Comparison: Mac mini vs VPS vs Moltworker

| Aspect | Mac mini | VPS | Moltworker |
|--------|----------|-----|------------|
| **Setup complexity** | Low | Medium | Medium |
| **Ongoing maintenance** | Medium (updates, uptime) | High (patching, security) | Low (managed) |
| **Cost** | Hardware upfront (~$500+) | $6-20/month | $5-10/month |
| **Uptime** | Depends on your power/network | High (provider SLA) | High (Cloudflare SLA) |
| **Privacy** | Highest (your hardware) | Medium (your VPS) | Lower (Cloudflare infra) |
| **Scaling** | Fixed capacity | Manual scaling | Automatic |
| **Tool support** | Full (local files, devices) | Full | Limited (no local access) |
| **Best for** | Privacy-first, home users | Always-on, remote access | Low-maintenance, serverless fans |

---

## Limitations

### Proof-of-concept status

Moltworker is experimental. It demonstrates that the Gateway can run serverlessly, but:

- Not all tools work identically (no local filesystem, no persistent browser sessions)
- Long-running operations may hit Worker CPU limits
- Webhook delivery for some channels may require additional configuration
- Not yet covered by OpenClaw's stability guarantees
- Subject to change without notice

### Development constraints

- **Local development:** WebSocket proxying has constraints in `wrangler dev`
- **R2 mounting:** Not available in local development environment
- **Cold starts:** Initial requests take 1-2 minutes

### Not a Cloudflare product

Moltworker is an experimental deployment pattern. For issues:

- Gateway/agent issues: [openclaw/openclaw](https://github.com/openclaw/openclaw/issues)
- Moltworker-specific issues: [cloudflare/moltworker](https://github.com/cloudflare/moltworker/issues)
- Cloudflare Workers issues: [Cloudflare Community](https://community.cloudflare.com/)

---

## Troubleshooting

### Worker returns 500 errors

Check logs in Cloudflare dashboard:

1. Go to Workers & Pages
2. Select your worker
3. Click "Logs" tab
4. Look for error messages

Common causes:

- Missing secrets (API keys not set)
- R2 bucket not created or misconfigured
- Wrangler.toml binding errors

### Gateway auth fails

Verify your token:

```bash
# Set via secret
wrangler secret put GATEWAY_AUTH_TOKEN

# Test
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://moltworker.your-subdomain.workers.dev/api/status
```

### Sessions not persisting

Check R2 bucket:

```bash
# List bucket contents
wrangler r2 object list moltworker-state
```

If empty after use, check worker logs for R2 write errors.

### Model API calls failing

1. Verify API key is set: `wrangler secret list`
2. Check AI Gateway logs (if using)
3. Test direct API call to rule out provider issues

---

## Security Checklist (Moltworker)

### Cloudflare Configuration

- [ ] Workers paid plan active
- [ ] Worker deployed with latest code
- [ ] R2 bucket created with private access
- [ ] No public R2 bucket URLs exposed
- [ ] Cloudflare Access protecting admin routes

### Secrets

- [ ] `MOLTBOT_GATEWAY_TOKEN` set (strong random value)
- [ ] `CF_ACCESS_TEAM_DOMAIN` and `CF_ACCESS_AUD` set
- [ ] Model provider API keys set as secrets
- [ ] No secrets in wrangler.toml or committed code

### Access Control

- [ ] Gateway auth token required for Control UI access
- [ ] Cloudflare Access required for admin routes
- [ ] DM policy is `allowlist` or `pairing`
- [ ] Only approved user IDs can trigger actions
- [ ] Device pairing enabled for new devices

### Persistence

- [ ] R2 bucket configured for state persistence
- [ ] Backup cron job running (5-minute intervals)
- [ ] Manual backup tested via Admin UI

### Monitoring

- [ ] Worker logs accessible in dashboard
- [ ] Error alerting configured (optional)
- [ ] Regular review of R2 bucket contents

---

## Next Steps

After deployment:

1. **Connect a channel** — Set up Telegram, Discord, or another channel to talk to your Gateway
2. **Configure pairing** — Ensure only you can interact with the bot
3. **Test tools** — Verify web fetch, browser rendering work as expected
4. **Monitor usage** — Check Cloudflare dashboard for requests and costs

For channel setup, see: [Pairing Guide](https://docs.openclaw.ai/start/pairing)
