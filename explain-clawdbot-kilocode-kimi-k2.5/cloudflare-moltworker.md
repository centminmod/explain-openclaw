# Cloudflare Moltworker Deployment

> **Last Updated:** January 2026  
> **Reference:** [Cloudflare Moltworker Blog Post](https://blog.cloudflare.com/moltworker-self-hosted-ai-agent/)  
> **GitHub:** https://github.com/cloudflare/moltworker

---

## Table of Contents

1. [What is Moltworker?](#what-is-moltworker)
2. [How It Works](#how-it-works)
3. [Cloudflare Platform Services Used](#cloudflare-platform-services-used)
4. [Deployment Instructions](#deployment-instructions)
5. [Configuration](#configuration)
6. [Limitations vs. Self-Hosted](#limitations-vs-self-hosted)
7. [Security Considerations](#security-considerations)

---

## What is Moltworker?

**Moltworker** is Cloudflare's reference deployment that adapts Moltbot to run on [Cloudflare Workers](https://workers.cloudflare.com/) instead of a traditional server. It demonstrates how a self-hosted AI agent can operate in a serverless, sandboxed environment while retaining persistent state.

### Plain English Explanation

Instead of running Moltbot on your own computer or VPS, Moltworker runs it on Cloudflare's global edge network. Think of it as "serverless Moltbot"—you don't manage a server, it scales automatically, and runs close to your users worldwide.

### Key Differences from Self-Hosted Moltbot

| Aspect | Self-Hosted Moltbot | Cloudflare Moltworker |
|--------|--------------------|-----------------------|
| **Infrastructure** | Your hardware/VPS | Cloudflare's edge network |
| **Persistence** | Local filesystem | Durable Objects + KV |
| **Shell Access** | Full bash/exec | Limited/isolated |
| **24/7 Availability** | Depends on your setup | Always available |
| **Scalability** | Single instance | Auto-scaling |
| **Cost** | Hardware/power/VPS | Free tier available |

---

## How It Works

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                 CLOUDFLARE EDGE NETWORK                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Workers Runtime (Isolated)                 │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │   │
│  │  │   Moltbot   │  │   AI Agent  │  │   WebSocket │     │   │
│  │  │   Gateway   │◄─┤   (Pi RPC)  │◄─┤   Handler   │     │   │
│  │  │   Logic     │  │             │  │             │     │   │
│  │  └──────┬──────┘  └─────────────┘  └──────┬──────┘     │   │
│  │         │                                  │            │   │
│  │  ┌──────┴──────────────────────────────────┴──────┐    │   │
│  │  │         Durable Objects (State)                │    │   │
│  │  │  - Session persistence                        │    │   │
│  │  │  - WebSocket connections                      │    │   │
│  │  │  - Agent state                                │    │   │
│  │  └───────────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                    │
│  ┌─────────────────────────┴─────────────────────────┐         │
│  │              Cloudflare Services                  │         │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐           │         │
│  │  │    KV   │  │   R2    │  │  D1     │           │         │
│  │  │ Storage │  │  Files  │  │  SQL    │           │         │
│  │  └─────────┘  └─────────┘  └─────────┘           │         │
│  └───────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────┘
                              │
           ┌──────────────────┼──────────────────┐
           │                  │                  │
           ▼                  ▼                  ▼
      ┌─────────┐       ┌─────────┐       ┌─────────┐
      │ WhatsApp│       │Telegram │       │Discord  │
      │ Webhook │       │ Webhook │       │ Webhook │
      └─────────┘       └─────────┘       └─────────┘
```

### Request Flow

1. **Inbound Message** → Cloudflare Worker receives webhook (WhatsApp/Telegram/Discord)
2. **State Retrieval** → Durable Object fetches session state from KV/D1
3. **AI Processing** → Request sent to configured AI provider (Anthropic/OpenAI)
4. **Response Generation** → AI generates response using available tools
5. **State Persistence** → Updated state saved to Durable Object
6. **Outbound Message** → Response sent back via channel webhook

---

## Cloudflare Platform Services Used

### 1. Cloudflare Workers

**What it is:** Serverless JavaScript/TypeScript execution environment running on Cloudflare's global edge network (300+ cities).

**How Moltworker uses it:**
- Hosts the Moltbot Gateway logic
- Handles HTTP requests and WebSocket connections
- Runs the AI agent runtime (Pi RPC)

**Key Features:**
- **V8 Isolates:** Each request runs in a secure, isolated environment
- **Cold start:** ~0ms (Workers are always ready)
- **CPU time:** 50ms (free) / 30s (paid) per request
- **Memory:** 128MB per isolate

```javascript
// Example: Worker entry point
export default {
  async fetch(request, env, ctx) {
    // Moltbot Gateway logic here
    return await handleRequest(request, env);
  }
};
```

### 2. Durable Objects

**What it is:** Stateful, coordinated compute units that maintain persistent state and coordinate between clients.

**How Moltworker uses it:**
- **Session Persistence:** Maintains conversation context across requests
- **WebSocket Coordination:** Handles real-time connections
- **Agent State:** Stores active agent configurations and memory

**Key Features:**
- **Persistence:** State survives across requests
- **Single-threaded:** Guarantees consistency
- **Geographic distribution:** Objects live in specific regions
- **Storage:** 1GB storage per object

```javascript
// Example: Durable Object for session state
export class Session {
  constructor(state, env) {
    this.state = state;
    this.env = env;
  }

  async fetch(request) {
    // Load session from storage
    const sessionData = await this.state.storage.get('session');
    
    // Process request
    const response = await processWithAgent(sessionData, request);
    
    // Save updated session
    await this.state.storage.put('session', updatedData);
    
    return response;
  }
}
```

### 3. Workers KV

**What it is:** Global, low-latency key-value storage.

**How Moltworker uses it:**
- **Configuration:** Store user settings and agent configs
- **Session Cache:** Fast access to recent session data
- **Credentials:** OAuth tokens and API keys (encrypted)

**Key Features:**
- **Global replication:** Access from any edge location
- **Eventual consistency:** Writes propagate globally in seconds
- **Read performance:** <50ms globally
- **Storage:** 1GB free, up to 1GB values

```javascript
// Example: KV usage
const config = await env.MOLTBOT_KV.get('user:123:config', 'json');
await env.MOLTBOT_KV.put('user:123:session', JSON.stringify(session));
```

### 4. R2 Object Storage

**What it is:** S3-compatible object storage, zero egress fees.

**How Moltworker uses it:**
- **File Storage:** Agent workspace files
- **Media:** Images, audio, video attachments
- **Logs:** Session transcripts and audit logs

**Key Features:**
- **Zero egress fees:** No cost for data transfer out
- **S3-compatible:** Works with existing S3 tools
- **Integration:** Direct binding to Workers

```javascript
// Example: R2 usage
const object = await env.MOLTBOT_BUCKET.get('files/document.pdf');
await env.MOLTBOT_BUCKET.put('files/output.txt', content);
```

### 5. D1 Database (Optional)

**What it is:** Serverless SQL database built on SQLite.

**How Moltworker uses it:**
- **Structured Data:** User accounts, channel configurations
- **Analytics:** Usage tracking and metrics
- **Relationships:** Multi-user workspace management

**Key Features:**
- **SQLite-compatible:** Familiar SQL interface
- **Serverless:** No connection management needed
- **Replication:** Read replicas at edge

```javascript
// Example: D1 usage
const { results } = await env.DB.prepare(
  'SELECT * FROM sessions WHERE user_id = ?'
).bind(userId).all();
```

### 6. AI Gateway (Optional)

**What it is:** Unified API for AI providers with caching and analytics.

**How Moltworker uses it:**
- **Provider Abstraction:** Single API for multiple AI models
- **Caching:** Reduce costs by caching similar requests
- **Rate Limiting:** Prevent abuse and control costs
- **Analytics:** Track usage and costs

**Key Features:**
- **Multi-provider:** OpenAI, Anthropic, Google, etc.
- **Caching:** Automatic response caching
- **Observability:** Built-in logging and metrics

```javascript
// Example: AI Gateway usage
const response = await env.AI.run('@cf/meta/llama-2-7b-chat-int8', {
  messages: [{ role: 'user', content: prompt }]
});
```

### 7. Queues (Optional)

**What it is:** Message queue for background job processing.

**How Moltworker uses it:**
- **Async Processing:** Handle long-running AI requests
- **Rate Limiting:** Smooth out traffic spikes
- **Retries:** Automatic retry of failed operations

---

## Deployment Instructions

### Prerequisites

1. **Cloudflare Account** (free tier works)
2. **Wrangler CLI** installed:
   ```bash
   npm install -g wrangler
   ```
3. **Authenticated with Cloudflare:**
   ```bash
   wrangler login
   ```

### Step 1: Clone Moltworker Repository

```bash
git clone https://github.com/cloudflare/moltworker.git
cd moltworker
```

### Step 2: Configure Environment

```bash
# Create environment file
cp .env.example .env

# Edit .env with your settings
cat > .env << 'EOF'
# AI Provider Configuration
ANTHROPIC_API_KEY=your-anthropic-key
OPENAI_API_KEY=your-openai-key

# Channel Configuration
WHATSAPP_WEBHOOK_SECRET=your-webhook-secret
TELEGRAM_BOT_TOKEN=your-telegram-token
DISCORD_BOT_TOKEN=your-discord-token

# Cloudflare Configuration
CLOUDFLARE_ACCOUNT_ID=your-account-id
CLOUDFLARE_API_TOKEN=your-api-token
EOF
```

### Step 3: Create KV Namespaces

```bash
# Create KV namespaces
wrangler kv:namespace create "MOLTBOT_KV"
wrangler kv:namespace create "MOLTBOT_SESSIONS"

# Note the IDs output and update wrangler.toml
```

### Step 4: Create Durable Object Namespace

```bash
# Durable Objects are defined in wrangler.toml
# No separate creation needed, just deploy
```

### Step 5: Deploy

```bash
# Install dependencies
npm install

# Deploy to Cloudflare
wrangler deploy

# Output will show your Worker URL:
# https://moltworker.your-account.workers.dev
```

### Step 6: Configure Webhooks

#### WhatsApp

```bash
# Set webhook URL in WhatsApp Business API
# or via your WhatsApp Business provider
curl -X POST "https://graph.facebook.com/v18.0/$PHONE_ID/subscribed_apps" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -d "webhook_url=https://moltworker.your-account.workers.dev/webhook/whatsapp"
```

#### Telegram

```bash
# Set webhook via BotFather or API
curl -X POST "https://api.telegram.org/bot$TOKEN/setWebhook" \
  -d "url=https://moltworker.your-account.workers.dev/webhook/telegram"
```

#### Discord

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Set Interactions Endpoint URL to: `https://moltworker.your-account.workers.dev/webhook/discord`

---

## Configuration

### wrangler.toml

```toml
name = "moltworker"
main = "src/index.ts"
compatibility_date = "2026-01-01"

# Node.js compatibility
compatibility_flags = ["nodejs_compat"]

# Durable Objects
[[durable_objects.bindings]]
name = "SESSIONS"
class_name = "Session"

[[migrations]]
tag = "v1"
new_classes = ["Session"]

# KV Namespaces
[[kv_namespaces]]
binding = "MOLTBOT_KV"
id = "your-kv-namespace-id"

# R2 Bucket
[[r2_buckets]]
binding = "MOLTBOT_BUCKET"
bucket_name = "moltbot-storage"

# D1 Database (optional)
[[d1_databases]]
binding = "DB"
database_name = "moltbot-db"
database_id = "your-database-id"

# Environment Variables
[vars]
NODE_ENV = "production"
LOG_LEVEL = "info"

# Secrets (use wrangler secret put)
[secrets]
ANTHROPIC_API_KEY
OPENAI_API_KEY
WHATSAPP_WEBHOOK_SECRET
```

### Moltworker-Specific Configuration

```typescript
// src/config.ts
export interface MoltworkerConfig {
  // AI Provider
  ai: {
    defaultModel: string;
    providers: {
      anthropic?: { apiKey: string };
      openai?: { apiKey: string };
      cloudflare?: { accountId: string };
    };
  };

  // Channels
  channels: {
    whatsapp?: { enabled: boolean; webhookSecret: string };
    telegram?: { enabled: boolean; botToken: string };
    discord?: { enabled: boolean; botToken: string };
  };

  // Storage
  storage: {
    sessionTTL: number;  // Seconds to persist sessions
    maxSessionSize: number;  // Max bytes per session
    encryptionKey: string;  // For encrypting sensitive data
  };

  // Security
  security: {
    allowedOrigins: string[];
    rateLimit: {
      requestsPerMinute: number;
      burstSize: number;
    };
  };
}
```

---

## Limitations vs. Self-Hosted

### What's Possible

| Feature | Moltworker Support |
|---------|-------------------|
| ✅ WhatsApp/Telegram/Discord | Full support via webhooks |
| ✅ AI Agent (Pi RPC) | Full support |
| ✅ Session persistence | Via Durable Objects |
| ✅ File storage | Via R2 |
| ✅ Database | Via D1 or external |
| ✅ WebSocket | Via Durable Objects |
| ✅ Cron jobs | Via Workers Cron Triggers |
| ✅ Custom tools | Within sandbox limits |

### What's Limited

| Feature | Limitation |
|---------|-----------|
| ⚠️ bash/exec | No shell access; use sandboxed alternatives |
| ⚠️ Browser control | Limited; no Chrome CDP |
| ⚠️ Local file system | Use R2 instead |
| ⚠️ Native binaries | Only WebAssembly or built-in APIs |
| ⚠️ Long-running processes | 30s CPU limit (paid) |
| ❌ iMessage | Not supported (macOS only) |
| ❌ Voice Wake | Not supported |
| ❌ System notifications | Not supported |

### Workarounds

**For bash/exec:**
```javascript
// Instead of bash, use Workers APIs
// Fetch for HTTP requests
const response = await fetch('https://api.example.com');

// Built-in crypto for hashing
const hash = await crypto.subtle.digest('SHA-256', data);
```

**For browser control:**
```javascript
// Use Browser Rendering API (paid add-on)
const browser = await puppeteer.launch(env.MYBROWSER);
```

---

## Security Considerations

### Built-in Cloudflare Security

| Feature | Benefit |
|---------|---------|
| **V8 Isolates** | Request isolation prevents cross-tenant attacks |
| **WAF** | Web Application Firewall blocks common attacks |
| **DDoS Protection** | Automatic mitigation of volumetric attacks |
| **TLS 1.3** | Encrypted connections by default |
| **Zero Trust** | Identity-aware access controls |

### Moltworker-Specific Security

```typescript
// Security configuration
const securityConfig = {
  // Validate all incoming webhooks
  webhookValidation: {
    whatsapp: verifyWhatsAppSignature,
    telegram: verifyTelegramToken,
    discord: verifyDiscordSignature
  },

  // Rate limiting per user
  rateLimit: {
    windowMs: 60000,  // 1 minute
    maxRequests: 30   // per user per minute
  },

  // Content Security Policy
  headers: {
    'Content-Security-Policy': "default-src 'self'",
    'X-Frame-Options': 'DENY',
    'X-Content-Type-Options': 'nosniff'
  },

  // Encryption at rest
  encryption: {
    algorithm: 'AES-GCM',
    keyRotation: 86400 * 30  // Rotate keys monthly
  }
};
```

### Secrets Management

```bash
# Store secrets securely (encrypted at rest)
wrangler secret put ANTHROPIC_API_KEY
wrangler secret put OPENAI_API_KEY
wrangler secret put WEBHOOK_SECRET

# List secrets
wrangler secret list

# Delete secrets
wrangler secret delete ANTHROPIC_API_KEY
```

### Compliance Notes

- **GDPR:** Cloudflare is GDPR compliant; implement data deletion hooks
- **SOC 2:** Cloudflare maintains SOC 2 Type II certification
- **PCI DSS:** Cloudflare is Level 1 PCI compliant

---

## Cost Analysis

### Free Tier

| Resource | Limit |
|----------|-------|
| Workers | 100,000 requests/day |
| KV | 1GB storage |
| Durable Objects | 400k request units/month |
| R2 | 10GB storage |
| D1 | 5M rows read/month |

### Typical Costs (Paid)

| Usage | Estimated Monthly Cost |
|-------|----------------------|
| Light (personal) | $0-5 |
| Medium (small team) | $10-30 |
| Heavy (business) | $50-200 |

**Cost Optimization:**
- Use AI Gateway caching
- Implement request batching
- Compress stored data
- Use appropriate storage tier (KV vs R2 vs D1)

---

## Related Documentation

- [Cloudflare Workers Docs](https://developers.cloudflare.com/workers/)
- [Durable Objects Docs](https://developers.cloudflare.com/durable-objects/)
- [Cloudflare Moltworker GitHub](https://github.com/cloudflare/moltworker)
- [Cloudflare Blog Post](https://blog.cloudflare.com/moltworker-self-hosted-ai-agent/)
- [Deployment Scenarios](./deployment-scenarios.md)
- [Security Analysis](./security-analysis.md)