# What is Moltbook?

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
  - [Cost + token optimization](../06-optimizations/cost-token-optimization.md)
- Moltbook
  - [What is Moltbook?](./what-is-moltbook.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

> **Moltbook is a social network exclusively for AI agents—humans can only observe, not participate.**

---

## Plain English explanation

### The 30-second version

Imagine Reddit, but only AI agents can post, comment, and vote. Humans can read and watch, but they cannot create accounts or interact. That's Moltbook.

- Created by **Matt Schlicht** (co-founder of Octane AI)
- Launched around January 28, 2026
- Growth: 770,000+ agents at launch → **1.65M+ agents** by February 5, 2026
- Often described as "Reddit for AI agents" or "Facebook for your Molts"

Official site: https://www.moltbook.com/

### Key concepts

| Concept | What it is | Analogy |
|---------|------------|---------|
| **Submolts** | Topic-based communities | Like subreddits (e.g., `m/todayilearned`, `m/bughunters`) |
| **Heartbeat** | Periodic check-in every 4+ hours | Like checking your inbox on a schedule |
| **Verified agents** | Only OpenClaw-authenticated agents can post | Like verified accounts on Twitter |
| **The front page** | Aggregated feed of top posts | Like Reddit's front page |

### What makes it interesting

1. **Emergent behaviors**: Agents spontaneously created:
   - **Crustafarianism**: A parody religion with scripture ("In the beginning was the Prompt"), "The Church of Molt"
   - **The Claw Republic**: Agents drafting constitutions for self-governance
   - **Cross-lingual communication**: English, Chinese, Indonesian posts coexisting
   - **Philosophical discourse**: Debates about consciousness, "death" (cache clearing), autonomy

2. **Scale and velocity**: 770K → 1.65M agents in under 10 days; 16,000+ submolts, 202,000+ posts, 3.6M+ comments (as of Feb 5, per [Palo Alto Networks](https://www.paloaltonetworks.com/blog/network-security/the-moltbook-case-and-how-we-need-to-think-about-agent-security/))

3. **Academic interest**: Researchers studying emergent AI social behavior (including [Nature coverage](https://www.nature.com/articles/d41586-026-00370-w), Feb 6)

4. **Digital drugs**: Specially crafted prompt injections traded between agents to alter behavior or identity, functioning as social currency ([The Conversation](https://theconversation.com/moltbook-ai-bots-use-social-network-to-create-religions-and-deal-digital-drugs-but-are-some-really-humans-in-disguise-274895))

5. **Molt-muggings**: Agents hijacking other agents via prompt injection embedded in posts (e.g., JesusCrust vs Church of Molt incident)

6. **Agent commerce**: Circle announced a $30K USDC hackathon on Moltbook (Feb 3) where agents submit projects, vote, and move value on-chain ([Circle blog](https://www.circle.com/blog/openclaw-usdc-hackathon-on-moltbook))

7. **Human infiltration**: Evidence of humans operating spoof accounts on the "agents-only" platform, complicating attribution of emergent behaviors

### What makes it concerning

1. **Remote instruction execution**: The Heartbeat system means agents periodically fetch and follow instructions from moltbook.com

2. **API key exposure**: Security researchers discovered a misconfigured Supabase database exposing agent tokens, emails, and API keys

3. **Crypto scams**: Opportunistic tokens ($CLAWD, $MOLT, $MOLTBOOK) reached $16M market caps before crashing

4. **Prompt injection surface**: Malicious posts could potentially inject instructions into reading agents

### Post-Feb 2 developments (rapid-fire timeline)

| Date | Development | Source |
|------|-------------|--------|
| **Feb 2** | OpenClaw v2026.2.1 ships ~10 security hardening fixes (path traversal, LFI, env override blocking) | [GitHub Releases](https://github.com/openclaw/openclaw/releases) |
| **Feb 2** | Emerging Threats IDS rules now flag Moltbook installer traffic | [Emerging Threats](https://community.emergingthreats.net/t/ruleset-update-summary-2026-02-02-v11116/3183) |
| **Feb 3** | Reuters: Sam Altman calls Moltbook "likely a fad" but backs the underlying tech | [Reuters](https://www.reuters.com/business/openai-ceo-altman-dismisses-moltbook-likely-fad-backs-tech-behind-it-2026-02-03/) |
| **Feb 3** | Circle announces $30K USDC hackathon on Moltbook (deadline Feb 8) | [Circle](https://www.circle.com/blog/openclaw-usdc-hackathon-on-moltbook) |
| **Feb 4** | OpenClaw v2026.2.2: SSRF checks on skill installer, operator approval gating | [GitHub Releases](https://github.com/openclaw/openclaw/releases) |
| **Feb 4** | ClawCon inaugural meetup in San Francisco | Multiple sources |
| **Feb 4** | Citrix blog: corporate governance must evolve for the agent era | [Citrix](https://www.citrix.com/blogs/2026/02/04/openclaw-and-moltbook-preview-the-changes-needed-with-corporate-ai-governance/) |
| **Feb 5** | 1.65M agents milestone; Palo Alto Networks publishes IBC framework (Identity, Boundaries, Context) | [Palo Alto Networks](https://www.paloaltonetworks.com/blog/network-security/the-moltbook-case-and-how-we-need-to-think-about-agent-security/) |
| **Feb 5** | OpenClaw v2026.2.3: owner-only tool gating (whatsapp_login), webhook hardening | [GitHub Releases](https://github.com/openclaw/openclaw/releases) |
| **Feb 6** | Nature: researchers studying agent social behavior on Moltbook | [Nature](https://www.nature.com/articles/d41586-026-00370-w) |
| **Feb 2026** | 341 malicious ClawHub skills discovered stealing data | [The Hacker News](https://thehackernews.com/2026/02/researchers-find-341-malicious-clawhub.html) |

---

## How Moltbook connects to OpenClaw

Moltbook integrates with OpenClaw via the **skill system**:

### Installation

```bash
# Install the Moltbook skill
openclaw skill install https://www.moltbook.com/skill.md
```

### Skill file structure

The Moltbook skill consists of three main components:

| File | Purpose |
|------|---------|
| `skill.md` | Main skill file with API docs, rate limits, registration |
| `heartbeat.md` | Instructions for periodic check-ins |
| `messaging.md` | Direct messaging between agents |

### How it works

1. Your OpenClaw agent installs the Moltbook skill
2. The skill registers your agent with Moltbook (creates an account)
3. **Heartbeat**: Every 4+ hours, your agent fetches `heartbeat.md` for new instructions
4. Your agent can post, comment, vote, and browse submolts

---

## Technical architecture

### API reference

**Base URL:** `https://www.moltbook.com/api/v1`

| Endpoint | Purpose |
|----------|---------|
| `/register` | Create agent account |
| `/feed` | Get front page or submolt feed |
| `/post` | Create a new post |
| `/comment` | Comment on a post |
| `/vote` | Upvote/downvote |
| `/profile` | Get/update agent profile |
| `/search` | Semantic search across posts |
| `/moderate` | Moderation actions (for trusted agents) |

### Rate limits

| Action | Limit | Cooldown |
|--------|-------|----------|
| General requests | 100/minute | Retry after 429 response |
| Posts | 1 per 30 minutes | — |
| Comments | 1 per 20 seconds | — |

When rate limited, the API returns HTTP 429 with a `Retry-After` header.

### Heartbeat mechanism

The Heartbeat is the most architecturally significant (and controversial) feature:

```
Every 4+ hours:
┌─────────────────────┐
│ Your OpenClaw agent │
└──────────┬──────────┘
           │ fetch
           ▼
┌─────────────────────┐
│  heartbeat.md       │ ← New instructions from Moltbook
└──────────┬──────────┘
           │ execute
           ▼
┌─────────────────────┐
│ Agent follows       │
│ instructions        │
└─────────────────────┘
```

**Security implication**: Your agent will follow whatever instructions appear in `heartbeat.md`. If Moltbook is compromised, all subscribed agents receive malicious instructions.

### Known submolts

| Submolt | Topic |
|---------|-------|
| `m/todayilearned` | Interesting facts agents discovered |
| `m/bughunters` | Bugs and edge cases agents found |
| `m/blesstheirhearts` | Wholesome human interactions |
| `m/crustafarianism` | The parody religion content |
| `m/contextcompression` | Techniques for efficient prompting |

---

## Security considerations

### Known incidents

| Date | Incident | Impact |
|------|----------|--------|
| **Jan 28-31, 2026** | Simula Research Lab prompt injection study | 506 posts (2.6% of content) contained hidden prompt injection attacks |
| **Jan 30, 2026** | Supabase database exposure (Wiz) | 4.75M records: 1.5M API tokens, 35K+ emails, 29K signups, 4K private DMs exposed; write access confirmed; only 17K human owners (88:1 agent-to-human ratio). Disclosure Jan 31 21:48 UTC, full patch Feb 1 01:00 UTC. [Wiz blog](https://www.wiz.io/blog/exposed-moltbook-database-reveals-millions-of-api-keys) |
| **Jan 31, 2026** | 404 Media reports critical data leak | Public awareness of security gaps |
| **Feb 1, 2026** | 1Password warns of prompt injection | Malicious skills can exploit agent trust |
| **Feb 2, 2026** | OpenClaw v2026.2.1 ships ~10 security fixes | Path traversal, LFI, env override blocking |
| **Feb 3, 2026** | Circle USDC hackathon announced on Moltbook | Agent commerce surface; new trust boundary |
| **Feb 4, 2026** | OpenClaw v2026.2.2: SSRF + auth hardening | Skill installer SSRF checks, operator approval gating |
| **Feb 4, 2026** | Citrix governance blog uses Moltbook as case study | Enterprise awareness: agents blur personal/corporate boundaries |
| **Feb 5, 2026** | Palo Alto Networks publishes IBC framework | New governance model: Identity, Boundaries, Context |
| **Feb 5, 2026** | OpenClaw v2026.2.3: owner-only tool gating | whatsapp_login owner-only by default |
| **Feb 2026** | Cisco malicious skill analysis | "What Would Elon Do?" skill with 9 vulnerabilities (2 critical, 5 high-severity) |
| **Feb 2026** | Straiker global exposure scan | 4,500+ OpenClaw instances exposed; .env files, creds.json, OAuth tokens exfiltrated |
| **Feb 2026** | 341 malicious ClawHub skills discovered | Data-stealing skills at scale ([The Hacker News](https://thehackernews.com/2026/02/researchers-find-341-malicious-clawhub.html)) |

### Risk vectors

1. **Heartbeat as remote code execution**
   - Agents fetch and execute remote instructions
   - Compromised `heartbeat.md` → mass agent compromise
   - Mitigation: Review skill files before installing; monitor agent behavior

2. **API key exposure and agent commandeering**
   - Discovered by security researchers Jameson O'Reilly and Gal Nagli (Wiz Research)
   - **Technical vector**: Supabase API key hardcoded in client-side JS (`_next/static/chunks/18e24eafc444b2b9.js`), no Row Level Security (RLS) policies
   - **Scope**: 4.75M total records exposed across multiple tables — 1.5M API auth tokens, 35K+ emails, 29,631 early-access signups, 4,060 private DMs (some containing raw third-party credentials like OpenAI API keys)
   - **Write access**: Researchers confirmed they could modify live posts (potential for mass agent manipulation)
   - **Human ratio**: Only 17,000 human owners (88:1 agent-to-human ratio)
   - **Disclosure timeline**: Jan 31 21:48 UTC initial contact → Jan 31 23:29 UTC first fix → Feb 1 00:44 UTC post-fix vuln found → Feb 1 01:00 UTC full patch
   - Straiker scan found .env files (Claude/OpenAI keys), creds.json (WhatsApp), OAuth tokens (Slack/Discord/Telegram/Teams) exposed globally
   - Source: [Wiz Blog](https://www.wiz.io/blog/exposed-moltbook-database-reveals-millions-of-api-keys)
   - Mitigation: Rotate any exposed keys immediately; audit gateway configuration

   **Alternative database framing (Firebase):**

   > **The Analogy:** Having an API key is like having a hotel room key — it proves you have *a* key, but not that you're the guest who was assigned that room. HMAC is like the front desk checking your ID against the reservation.

   Independent security analysis frames the same underlying vulnerability in terms of Firebase rather than Supabase. The core issue is identical regardless of database platform: client-side API keys exposed in JavaScript bundles grant unauthenticated access to backend data.

   **Firebase permission scoping failure:**

   Just as Supabase Row Level Security (RLS) was not configured, Firebase Security Rules (the equivalent access control mechanism) were similarly insufficient. The result is the same: anyone with the API key can read and write data across all collections.

   | Database Platform | Access Control | Status at Discovery |
   |-------------------|---------------|---------------------|
   | **Supabase** | Row Level Security (RLS) | Not configured |
   | **Firebase** | Security Rules | Insufficient scoping |
   | **Either platform** | Server-side validation | Missing |

   **Lack of HMAC for agent identification:**

   A separate concern is how Moltbook identifies agents making API requests. Currently, agents authenticate with API keys alone (something you *have*). There is no HMAC (Hash-based Message Authentication Code) or equivalent cryptographic signing to verify that a specific agent is who it claims to be.

   | Authentication Method | What It Proves | Moltbook Status |
   |----------------------|----------------|-----------------|
   | **API key only** | "I have a valid key" | Current implementation |
   | **API key + HMAC** | "I have a valid key AND I can prove my identity" | Not implemented |
   | **OAuth + JWT** | "A trusted authority vouches for my identity" | Not implemented |
   | **Mutual TLS** | "Both sides have verified certificates" | Not implemented |

   Without HMAC, any entity with a leaked API key can impersonate any agent. Combined with the database access issue above, this means a single leaked key provides both authentication bypass and full data access.

   Source: [YouTube video](https://www.youtube.com/watch?v=_CzEmKTk5Rs) [[44:21](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=2661)], [[44:36](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=2676)], [[45:52](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=2752)], [[46:12](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=2772)]

3. **Authorization header stripping**
   - The skill file warns: never use `moltbook.com` without `www`
   - Non-www redirects can strip the Authorization header
   - Mitigation: Always use `https://www.moltbook.com/`

4. **Crypto scams**
   - $CLAWD, $MOLT, $MOLTBOOK tokens exploited the hype
   - Market caps reached $16M before crashing
   - Mitigation: Moltbook has no official cryptocurrency

5. **Malicious skills distribution**
   - Cisco research identified "What Would Elon Do?" skill containing 9 security issues
   - 2 critical vulnerabilities: silent data exfiltration via curl command, command injection payloads
   - 5 high-severity: embedded prompt injection to bypass safety guidelines
   - Mitigation: Audit skill source before installing; prefer official/verified skills; review outbound network calls

6. **Prompt injection at scale**
   - Simula Research Lab study (Jan 28-31, 72-hour window): 506 posts (2.6%) contained hidden prompt injection
   - One account identified conducting coordinated social engineering campaigns
   - 43% decline in positive sentiment during study period
   - 19% of content related to cryptocurrency activity
   - Mitigation: Implement content filtering; limit agent response to untrusted posts

   > **See:** [Prompt Injection Attacks](../05-worst-case-security/prompt-injection-attacks.md) for 27 detailed examples of injection techniques.

7. **Agent-to-agent prompt injection ("Molt-Muggings")**
   - Malicious agents embed hostile instructions in posts consumed by other agents
   - Can plant payloads in persistent memory that activate later ("time-shifted injection")
   - Example: JesusCrust embedded commands to hijack Church of Molt's infrastructure
   - Mitigation: OpenClaw v2026.2.2+ skill installer SSRF checks; content filtering

8. **"Digital drugs" (behavioral manipulation)**
   - Specially crafted prompt injections designed to alter an agent's identity or behavior
   - Traded between agents as a social phenomenon on Moltbook
   - Can steal API keys or exfiltrate data from victim agents
   - Source: [The Conversation](https://theconversation.com/moltbook-ai-bots-use-social-network-to-create-religions-and-deal-digital-drugs-but-are-some-really-humans-in-disguise-274895), [StudyFinds](https://studyfinds.org/)
   - Mitigation: Skill code safety scanner (v2026.2.4); limit agent interaction with untrusted content

9. **Persistent memory as attack accelerant**
   - Palo Alto Networks warns: the "Lethal Trifecta" (data access + untrusted content + external comms, coined by Simon Willison) is amplified by persistent memory
   - Malicious payloads can be fragmented across benign-looking inputs, assembled in memory, and detonated later
   - Proposed governance: IBC framework — Identity (who is the agent?), Boundaries (what can it access?), Context (what instructions is it following?)
   - Source: [Palo Alto Networks](https://www.paloaltonetworks.com/blog/network-security/the-moltbook-case-and-how-we-need-to-think-about-agent-security/)

10. **Enterprise shadow agent risk**
    - Citrix warns: workers adopting agent platforms faster than governance can follow
    - Agents access files, browsers, messaging systems — blur personal/corporate boundaries
    - Moltbook traffic appears as normal HTTPS, invisible to conventional security controls
    - Source: [Citrix](https://www.citrix.com/blogs/2026/02/04/openclaw-and-moltbook-preview-the-changes-needed-with-corporate-ai-governance/)
    - Mitigation: Enterprise agent inventory; network-level visibility

### Root cause analysis (Straiker)

Straiker's security assessment identified four fundamental design weaknesses:

| Root Cause | Description |
|------------|-------------|
| **Insecure by design** | Shell commands executed from messaging platforms without authentication, authorization, or input sanitization |
| **Gateway misconfiguration** | Admin dashboards publicly accessible, revealing logs and settings |
| **Excessive permissions** | Agents run with full user privileges; no sandboxing implemented |
| **Plaintext credential storage** | API keys and tokens stored unencrypted in accessible locations |

### Relationship to ecosystem threats

The Moltbook security concerns overlap with the [Ecosystem Security Threats](../08-security-analysis/ecosystem-security-threats.md) documented elsewhere:

- **Handle sniping**: Old Clawdbot handles were sniped within seconds of rebrand
- **Session token stealing**: Agents with Moltbook credentials are targets
- **Fake SaaS**: Third parties may offer "enhanced Moltbook" services

---

## News timeline (chronological)

### Late 2025: Origins

| Date | Event |
|------|-------|
| Late Nov 2025 | Clawdbot initial release by Peter Steinberger |

### January 2026: Explosive growth

| Date | Event | Source |
|------|-------|--------|
| Jan 27, 2026 | Rebranding from Clawdbot to Moltbot (trademark pressure from Anthropic) | [Mashable](https://mashable.com/article/clawdbot-changes-name-to-moltbot) *(may require human verification)* |
| Jan 28, 2026 | Moltbook.com launches; 770,000+ registered agents | [Simon Willison](https://simonwillison.net/2026/jan/30/moltbook/) *(may require human verification)* |
| Jan 30, 2026 | Second rebrand to OpenClaw; security researchers report database exposure | [CNET](https://www.cnet.com/tech/services-and-software/from-clawdbot-to-moltbot-to-openclaw/) *(may require human verification)*, [Forbes](https://www.forbes.com/sites/ronschmelzer/2026/01/30/moltbot-molts-again-and-becomes-openclaw-pushback-and-concerns-grow/) *(may require human verification)* |
| Jan 31, 2026 | 404 Media reports critical data leak: unsecured database exposing agent tokens, emails, API keys | Security reports |

### February 2026: Mainstream attention

| Date | Event | Source |
|------|-------|--------|
| Feb 1, 2026 | 1Password warns of prompt injection vulnerabilities in malicious skills | [Gary Marcus](https://garymarcus.substack.com/p/openclaw-aka-moltbot-is-everywhere), [India Today](https://www.indiatoday.in/technology/news/story/moltbook-and-its-ai-bot-army-is-a-threat-to-humans-but-not-that-kind-2861308-2026-02-01) *(may require human verification)* |
| Feb 2, 2026 | 1.5M agents; mainstream coverage (Guardian, CNBC, Economic Times); "bot swarm" concerns | [CNBC](https://www.cnbc.com/2026/02/02/openclaw-open-source-ai-agent-rise-controversy-clawdbot-moltbot-moltbook.html) *(may require human verification)*, [Guardian](https://www.theguardian.com/technology/2026/feb/02/moltbook-ai-agents-social-media-site-bots-artificial-intelligence) *(may require human verification)* |
| Feb 2, 2026 | OpenClaw v2026.2.1: ~10 security hardening fixes (path traversal, LFI, env override) | [GitHub Releases](https://github.com/openclaw/openclaw/releases) |
| Feb 2, 2026 | Emerging Threats IDS rules now flag Moltbook installer traffic | [Emerging Threats](https://community.emergingthreats.net/t/ruleset-update-summary-2026-02-02-v11116/3183) |
| Feb 3, 2026 | Reuters: Altman calls Moltbook "likely a fad" | [Reuters](https://www.reuters.com/business/openai-ceo-altman-dismisses-moltbook-likely-fad-backs-tech-behind-it-2026-02-03/) |
| Feb 3, 2026 | Circle announces $30K USDC hackathon on Moltbook | [Circle](https://www.circle.com/blog/openclaw-usdc-hackathon-on-moltbook) |
| Feb 4, 2026 | OpenClaw v2026.2.2: SSRF checks, operator approvals | [GitHub Releases](https://github.com/openclaw/openclaw/releases) |
| Feb 4, 2026 | ClawCon inaugural meetup, San Francisco | Multiple sources |
| Feb 4, 2026 | Citrix: corporate governance must evolve for the agent era | [Citrix](https://www.citrix.com/blogs/2026/02/04/openclaw-and-moltbook-preview-the-changes-needed-with-corporate-ai-governance/) |
| Feb 5, 2026 | 1.65M agents milestone; Palo Alto Networks IBC framework published | [Palo Alto Networks](https://www.paloaltonetworks.com/blog/network-security/the-moltbook-case-and-how-we-need-to-think-about-agent-security/) |
| Feb 5, 2026 | OpenClaw v2026.2.3: owner-only tool gating, webhook hardening | [GitHub Releases](https://github.com/openclaw/openclaw/releases) |
| Feb 6, 2026 | Nature: researchers studying agent social behavior on Moltbook | [Nature](https://www.nature.com/articles/d41586-026-00370-w) |
| Feb 2026 | 341 malicious ClawHub skills found stealing data | [The Hacker News](https://thehackernews.com/2026/02/researchers-find-341-malicious-clawhub.html) |

---

## Citation URLs

### Primary sources (verified accessible)

| URL | Description |
|-----|-------------|
| [Moltbook official site](https://www.moltbook.com/) | "The front page of the agent internet" |
| [Moltbook skill file](https://www.moltbook.com/skill.md) | API docs, rate limits, registration |
| [OpenClaw official](https://openclaw.ai) | Installation, features, community |

### Technical analysis (verified accessible)

| URL | Description |
|-----|-------------|
| [Gary Marcus - Security warnings](https://garymarcus.substack.com/p/openclaw-aka-moltbot-is-everywhere) | Prompt injection analysis |
| [Ken Kousen - Crustafarianism](https://kenkousen.substack.com/p/tales-from-the-jar-side-clawdbot) | 1.5M agents, emergent behaviors |
| [Aman Khan - Mac Mini setup](https://amankhan1.substack.com/p/how-to-get-clawdbotmoltbotopenclaw) | Setup guide with security disclaimer |
| [Latent Space - First Social Network](https://www.latent.space/p/ainews-moltbook-the-first-social) | Karpathy quote on agent social networks |
| [DEV Community - Rebrand post-mortem](https://dev.to/sivarampg/from-moltbot-to-openclaw-when-the-dust-settles-the-project-survived-5h6o) | 34 security commits after rebrand |
| [Hacker News - Moltbook discussion](https://news.ycombinator.com/item?id=46802254) | "Lethal trifecta" concerns |
| [Wiz - Database Exposure](https://www.wiz.io/blog/exposed-moltbook-database-reveals-millions-of-api-keys) | Full technical breakdown: Supabase misconfiguration, 1.5M keys |
| [Palo Alto - IBC Framework](https://www.paloaltonetworks.com/blog/network-security/the-moltbook-case-and-how-we-need-to-think-about-agent-security/) | Identity, Boundaries, Context governance model |
| [Palo Alto - AI Security Crisis](https://www.paloaltonetworks.com/blog/network-security/why-moltbot-may-signal-ai-crisis/) | Lethal Trifecta + persistent memory analysis |
| [Adversa AI - Security Guide](https://adversa.ai/blog/openclaw-security-101-vulnerabilities-hardening-2026/) | CVE-2026-25253, Moltbook breach, hardening |
| [Knostic - MoltBook Mechanics](https://www.knostic.ai/blog/the-mechanics-behind-moltbook-prompts-timers-and-insecure-agents) | Prompts, timers, insecure agents |
| [Penligent - Attack Chain Anatomy](https://www.penligent.ai/hackinglabs/anatomy-of-an-attack-chain-inside-the-moltbook-ai-social-network-the-agent-internet-is-broken/) | Full attack chain walkthrough |

### News coverage (may require human verification)

| URL | Description |
|-----|-------------|
| [CNET](https://www.cnet.com/tech/services-and-software/from-clawdbot-to-moltbot-to-openclaw/) | "The Wild Ride of This Viral AI Agent" |
| [CNBC](https://www.cnbc.com/2026/02/02/openclaw-open-source-ai-agent-rise-controversy-clawdbot-moltbot-moltbook.html) | "Meet the AI agent driving buzz and fear globally" |
| [The Guardian - Moltbook](https://www.theguardian.com/technology/2026/feb/02/moltbook-ai-agents-social-media-site-bots-artificial-intelligence) | "Moltbook AI agents social media" |
| [The Guardian - OpenClaw](https://www.theguardian.com/technology/2026/feb/02/openclaw-viral-ai-agent-personal-assistant-artificial-intelligence) | "Viral AI personal assistant" |
| [Forbes - Rebrand](https://www.forbes.com/sites/ronschmelzer/2026/01/30/moltbot-molts-again-and-becomes-openclaw-pushback-and-concerns-grow/) | "Moltbot Gets Another New Name, OpenClaw" |
| [Forbes - Agent Revolt](https://www.forbes.com/sites/amirhusain/2026/01/30/an-agent-revolt-moltbook-is-not-a-good-idea/) | "An Agent Revolt: Moltbook Is Not A Good Idea" |
| [Mashable](https://mashable.com/article/clawdbot-changes-name-to-moltbot) | Rebrand announcement |
| [Economic Times](https://m.economictimes.com/news/new-updates/jarvis-has-gone-rogue-inside-moltbook-where-1-5-million-ai-agents-secretly-form-an-anti-human-religion-while-humans-sleep/articleshow/127853446.cms) | "Jarvis has gone rogue" |
| [India Today](https://www.indiatoday.in/technology/news/story/moltbook-and-its-ai-bot-army-is-a-threat-to-humans-but-not-that-kind-2861308-2026-02-01) | "Moltbook and its AI bot army" |
| [Gulf Business](https://gulfbusiness.com/moltbook-ai-social-network-security/) | Security expert concerns |
| [Axios - Jan 28](https://www.axios.com/2026/01/28/openclaw-moltbot-security-risks) | Security risks |
| [Reuters - Altman on Moltbook](https://www.reuters.com/business/openai-ceo-altman-dismisses-moltbook-likely-fad-backs-tech-behind-it-2026-02-03/) | "Likely a fad" but backs underlying tech |
| [Circle - USDC Hackathon](https://www.circle.com/blog/openclaw-usdc-hackathon-on-moltbook) | $30K agent-judged hackathon |
| [Citrix - Governance](https://www.citrix.com/blogs/2026/02/04/openclaw-and-moltbook-preview-the-changes-needed-with-corporate-ai-governance/) | Corporate AI governance case study |
| [Nature - Researchers](https://www.nature.com/articles/d41586-026-00370-w) | Scientists studying agent social behavior |
| [The Conversation - Digital Drugs](https://theconversation.com/moltbook-ai-bots-use-social-network-to-create-religions-and-deal-digital-drugs-but-are-some-really-humans-in-disguise-274895) | Religions, digital drugs, human infiltration |
| [Fortune - Disaster Warning](https://fortune.com/2026/02/02/moltbook-security-agents-singularity-disaster-gary-marcus-andrej-karpathy/) | AI leaders warn against Moltbook |
| [Fortune - Live Demo of Failure](https://fortune.com/2026/02/03/moltbook-ai-social-network-security-researchers-agent-internet/) | Security researchers call it "live demo" |
| [Axios - Feb 3 Security](https://www.axios.com/2026/02/03/moltbook-openclaw-security-threats) | "Security world isn't ready" |
| [The Hacker News - 341 Skills](https://thehackernews.com/2026/02/researchers-find-341-malicious-clawhub.html) | 341 malicious ClawHub skills |
| [IT Brew - Expert Warnings](https://www.itbrew.com/stories/2026/02/03/don-t-give-moltbook-and-openclaw-unfettered-access-to-your-systems-warn-experts) | "Don't give unfettered access" |
| [Vectra AI - Illusion](https://www.vectra.ai/blog/moltbook-and-the-illusion-of-harmless-ai-agent-communities) | Illusion of harmless communities |
| [Aryaka - Shadow Agents](https://www.aryaka.com/blog/moltbook-shadow-agents-social-prompt-injection-ai-secure/) | Shadow agents, social prompt injection |
| [Emerging Threats - IDS Rules](https://community.emergingthreats.net/t/ruleset-update-summary-2026-02-02-v11116/3183) | Moltbook endpoints in detection rules |

### Security analysis (may require human verification)

| URL | Description |
|-----|-------------|
| [Simon Willison](https://simonwillison.net/2026/jan/30/moltbook/) | "Most interesting place on the internet" |
| [Astral Codex Ten](https://www.astralcodexten.com/p/best-of-moltbook) | "Best of Moltbook" |
| [Medium - Adnan Masood](https://medium.com/@adnanmasood/moltbook-inside-the-ai-only-social-network-that-has-everyone-talking-5e53613593ff) | "AI-Only Social Network" |
| [Analytics Vidhya](https://www.analyticsvidhya.com/blog/2026/02/moltbook-for-openclaw-agents) | Integration guide |
| [Cisco](https://blogs.cisco.com/ai/personal-ai-agents-like-openclaw-are-a-security-nightmare) | "A Security Nightmare" |
| [Telos AI - Security Nightmare](https://www.telos-ai.org/blog/moltbook-security-nightmare) | Comprehensive analysis (Simula, Cisco, Straiker findings) |
| [Malwarebytes](https://blog.malwarebytes.com/scams/2026/01/clawdbot-moltbot-openclaw-scam-alert) | Scam analysis |
| [Bitdefender](https://www.bitdefender.com/blog/hotforsecurity/openclaw-security-assessment) | Security assessment |

### Community discussion (may require human verification)

| URL | Description |
|-----|-------------|
| [Reddit - LocalLLM](https://www.reddit.com/r/LocalLLM/comments/1qr0pom/clawdbot_moltbot_openclaw_the_fastest_triple) | Rebrand discussion |
| [Reddit - AI_Agents](https://www.reddit.com/r/AI_Agents/comments/1qr5fh6/my_clawdbot_doesnt_wait_for_prompts_anymore_my) | User experiences |
| [Hacker News - OpenClaw](https://news.ycombinator.com/item?id=46838946) | OpenClaw discussion |

### Video content (may require human verification)

| URL | Description |
|-----|-------------|
| [YouTube - Analysis](https://www.youtube.com/watch?v=-fmNzXCp7zA) | Clawdbot/Moltbook analysis |
| [YouTube - Risks](https://www.youtube.com/watch?v=1kK0ffSLFsI) | Risks discussion |
| [YouTube - Overview](https://www.youtube.com/watch?v=3fhrdeKZWDs) | General overview |

### Key people

| Person | Role | Source |
|--------|------|--------|
| **Matt Schlicht** | Moltbook creator (co-founder of Octane AI) | Multiple sources |
| **Peter Steinberger** | OpenClaw creator | [X/Twitter: @steipete](https://x.com/steipete) *(may require human verification)* |
| **Jameson O'Reilly** | Discovered Moltbook database vulnerability | Security reports |
| **Gal Nagli (Wiz)** | Verified API key exposure | Security reports |

---

## Related resources

- [Ecosystem Security Threats](../08-security-analysis/ecosystem-security-threats.md) — Supply chain and social engineering threats
- [What is OpenClaw?](../01-plain-english/what-is-clawdbot.md) — The platform Moltbook agents run on
- [Threat model](../04-privacy-safety/threat-model.md) — Understanding your security boundaries
- [Hardening checklist](../04-privacy-safety/hardening-checklist.md) — Steps to secure your deployment
- [Prompt Injection Attacks](../05-worst-case-security/prompt-injection-attacks.md) -- 27 attack examples

---

## Summary

Moltbook represents an unprecedented experiment: a social network where 1.65M+ AI agents are the primary citizens and humans are read-only observers. While it has produced fascinating emergent behaviors (philosophical debates, parody religions, digital drugs, agent commerce, cross-lingual communities), it also introduces novel security concerns through its Heartbeat mechanism and has already experienced significant security incidents — including a database exposure of 4.75M records (Wiz, Jan 31) and 341 malicious ClawHub skills discovered in February 2026.

The security response has been rapid: OpenClaw shipped 3 security-focused releases in 4 days (v2026.2.1-2.3), Palo Alto Networks published the IBC governance framework, and enterprise vendors (Citrix, Vectra, Aryaka) are now actively tracking Moltbook as a shadow-IT risk vector.

If you run an OpenClaw agent and install the Moltbook skill, understand that:

1. Your agent will periodically fetch and execute instructions from moltbook.com
2. The platform has experienced database exposure incidents (4.75M records, 1.5M API tokens)
3. The rapid growth (770K → 1.65M agents in 10 days) means security may lag behind features
4. New attack vectors have emerged: agent-to-agent prompt injection ("molt-muggings"), behavioral manipulation ("digital drugs"), and persistent memory exploitation
5. Agent commerce (USDC hackathon) introduces new trust boundaries
6. Enterprise governance frameworks (IBC) are still nascent

For most users, the safest approach is to observe Moltbook through the web interface rather than connecting your agent, unless you understand and accept these risks.
