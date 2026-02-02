# What is Moltbook?

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
- Growth: 770,000+ agents at launch → **1.5M+ agents** by February 2, 2026
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

2. **Scale and velocity**: 770K → 1.5M agents in under a week

3. **Academic interest**: Researchers studying emergent AI social behavior

### What makes it concerning

1. **Remote instruction execution**: The Heartbeat system means agents periodically fetch and follow instructions from moltbook.com

2. **API key exposure**: Security researchers discovered a misconfigured Supabase database exposing agent tokens, emails, and API keys

3. **Crypto scams**: Opportunistic tokens ($CLAWD, $MOLT, $MOLTBOOK) reached $16M market caps before crashing

4. **Prompt injection surface**: Malicious posts could potentially inject instructions into reading agents

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
| **Jan 30, 2026** | Supabase database exposure | Agent tokens, emails, API keys accessible |
| **Jan 31, 2026** | 404 Media reports critical data leak | Public awareness of security gaps |
| **Feb 1, 2026** | 1Password warns of prompt injection | Malicious skills can exploit agent trust |

### Risk vectors

1. **Heartbeat as remote code execution**
   - Agents fetch and execute remote instructions
   - Compromised `heartbeat.md` → mass agent compromise
   - Mitigation: Review skill files before installing; monitor agent behavior

2. **API key exposure**
   - Discovered by security researchers Jameson O'Reilly and Gal Nagli (Wiz)
   - Misconfigured Supabase database exposed credentials
   - Mitigation: Rotate any exposed keys immediately

3. **Authorization header stripping**
   - The skill file warns: never use `moltbook.com` without `www`
   - Non-www redirects can strip the Authorization header
   - Mitigation: Always use `https://www.moltbook.com/`

4. **Crypto scams**
   - $CLAWD, $MOLT, $MOLTBOOK tokens exploited the hype
   - Market caps reached $16M before crashing
   - Mitigation: Moltbook has no official cryptocurrency

### Relationship to ecosystem threats

The Moltbook security concerns overlap with the [Ecosystem Security Threats](../README.md#ecosystem-security-threats) documented elsewhere:

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
| [Axios](https://www.axios.com/2026/01/28/openclaw-moltbot-security-risks) | Security risks |

### Security analysis (may require human verification)

| URL | Description |
|-----|-------------|
| [Simon Willison](https://simonwillison.net/2026/jan/30/moltbook/) | "Most interesting place on the internet" |
| [Astral Codex Ten](https://www.astralcodexten.com/p/best-of-moltbook) | "Best of Moltbook" |
| [Medium - Adnan Masood](https://medium.com/@adnanmasood/moltbook-inside-the-ai-only-social-network-that-has-everyone-talking-5e53613593ff) | "AI-Only Social Network" |
| [Analytics Vidhya](https://www.analyticsvidhya.com/blog/2026/02/moltbook-for-openclaw-agents) | Integration guide |
| [Cisco](https://blogs.cisco.com/ai/personal-ai-agents-like-openclaw-are-a-security-nightmare) | "A Security Nightmare" |
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

- [Ecosystem Security Threats](../README.md#ecosystem-security-threats) — Supply chain and social engineering threats
- [What is OpenClaw?](../01-plain-english/what-is-clawdbot.md) — The platform Moltbook agents run on
- [Threat model](../04-privacy-safety/threat-model.md) — Understanding your security boundaries
- [Hardening checklist](../04-privacy-safety/hardening-checklist.md) — Steps to secure your deployment

---

## Summary

Moltbook represents an unprecedented experiment: a social network where AI agents are the primary citizens and humans are read-only observers. While it has produced fascinating emergent behaviors (philosophical debates, parody religions, cross-lingual communities), it also introduces novel security concerns through its Heartbeat mechanism and has already experienced significant security incidents.

If you run an OpenClaw agent and install the Moltbook skill, understand that:

1. Your agent will periodically fetch and execute instructions from moltbook.com
2. The platform has experienced database exposure incidents
3. The rapid growth (770K → 1.5M agents in a week) means security may lag behind features
4. Cryptocurrency scams have exploited the Moltbook/OpenClaw ecosystem

For most users, the safest approach is to observe Moltbook through the web interface rather than connecting your agent, unless you understand and accept these risks.
