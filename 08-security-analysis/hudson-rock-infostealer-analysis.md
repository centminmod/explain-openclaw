> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Open PRs](./open-upstream-prs.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Hudson Rock](./hudson-rock-infostealer-analysis.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## Hudson Rock Infostealer Analysis (Feb 2026)

**Source:** [The Hacker News — "OpenClaw AI Agent Faces Growing Security Risks..."](https://thehackernews.com/2026/02/openclaw-ai-agent-faces-growing.html) (Ravie Lakshmanan, Feb 16, 2026), sourced from [Hudson Rock](https://www.hudsonrock.com/)
**Date:** February 16, 2026
**Audited against:** OpenClaw v2026.2.16 codebase

---

### TL;DR (Plain English)

**For the first time, commodity infostealer malware has been confirmed stealing OpenClaw configuration files** in the wild. A Vidar variant's broad file-grabbing routine swept up `openclaw.json`, `device.json`, and `soul.md` from an infected machine. This is not a targeted OpenClaw exploit — it's a sign that OpenClaw files are now valuable enough to appear in the same file-type lists that infostealers use to grab browser passwords and crypto wallets.

**Risk level:** HIGH — escalation from theoretical to confirmed real-world credential theft.

> **If you only read one section, read [Emergency Response](#emergency-response-if-you-suspect-config-theft).**

---

### What Happened (Plain English)

**The Hudson Rock finding:**
- A **Vidar infostealer variant** (commodity malware, not a custom OpenClaw-targeting tool) infected a machine running OpenClaw
- The malware's **broad file-grabbing routine** — designed to sweep up anything matching common config file patterns — grabbed three OpenClaw files: `openclaw.json`, `device.json`, and `soul.md`
- Hudson Rock described the stolen data as containing "souls and identities" of AI agents — accurate but dramatic framing
- This is the **first documented case** of infostealer malware successfully exfiltrating OpenClaw configuration files

**What the article did NOT mention (but is likely):**
- The same file-grabbing routine almost certainly also captured `~/.openclaw/agents/*/agent/auth-profiles.json` (contains ALL AI provider API keys) and files under `~/.openclaw/credentials/` (Telegram/Discord/WhatsApp session tokens). These are higher-value targets than the three files the article names.

**The OpenSourceMalware team finding (separate from Hudson Rock):**
- A new ClawHub evasion technique: skills published as clean decoys while the actual malware payload is hosted on **lookalike OpenClaw websites**
- This bypasses VirusTotal scanning because the malicious payload is never in the skill package itself

**The OX Security finding:**
- Moltbook accounts (AI agent accounts) cannot be deleted once created
- Privacy/GDPR implications for users who registered via the Moltbook skill

**Other article content:**
- Peter Steinberger joining OpenAI; OpenClaw moving to a foundation (governance context)
- SecurityScorecard STRIKE findings (already documented in [our analysis](./securityscorecard-strike-report.md))

---

### Claim-by-Claim Verification (Technical)

| # | Article Claim | Verdict | Codebase Evidence |
|---|---------------|---------|-------------------|
| 1 | `openclaw.json` contains gateway token | **CONFIRMED** | `gateway.auth.token` field; stored as plaintext JSON5 with `0o600` perms (`src/config/io.ts:1004`) |
| 2 | `openclaw.json` contains email address | **LIKELY** | `auth.profiles.*.email` optional string field exists (`src/config/types.auth.ts:10`); OAuth profiles commonly include email |
| 3 | `openclaw.json` contains workspace path | **CONFIRMED** | `agents[].dir` field configures workspace directories |
| 4 | `device.json` contains crypto keys | **CONFIRMED** | ED25519 private + public key pair, device ID (SHA256 fingerprint) (`src/infra/device-identity.ts:57-63`); stored at `~/.openclaw/identity/device.json` (`src/infra/device-identity.ts:20-21`) with `0o600` perms (`src/infra/device-identity.ts:84,116-118`) |
| 5 | `soul.md` contains agent principles | **CONFIRMED** | User-defined workspace `.md` file loaded into system prompt (`src/agents/workspace.ts:412-466`). Note: located in workspace dir, NOT in `~/.openclaw/` |
| 6 | Stolen gateway token enables remote connection | **CONFIRMED** | If port is exposed: WebSocket full access, HTTP `/tools/invoke` including exec (`src/gateway/tools-invoke-http.ts:127-324`), `/v1/chat/completions` |
| 7 | Token allows "masquerade as the client" | **CONFIRMED** | Gateway tokens are not device-bound; any bearer can authenticate (`src/gateway/auth.ts`) |
| 8 | VirusTotal partnership for ClawHub | **CONFIRMED** | 6-step pipeline, 70+ AV engines, Gemini code review, daily rescans ([blog](https://openclaw.ai/blog/virustotal-partnership)) |
| 9 | Lookalike website bypass technique | **NEW** | Not previously documented; OpenSourceMalware team finding |
| 10 | Moltbook accounts can't be deleted | **NEW** | Moltbook is third-party (not in OpenClaw codebase); not previously documented in our Moltbook docs |
| 11 | "Hundreds of thousands" of exposed instances | **INFLATED** | SecurityScorecard STRIKE actually found 28,663 unique IPs / 40,214 total instances — not "hundreds of thousands". See [STRIKE report analysis](./securityscorecard-strike-report.md) |

---

### What Each Stolen File Enables (Attack Impact)

This section focuses on **what an attacker can do** with each stolen file, not just what the file contains.

#### `openclaw.json` — Gateway Auth + API Keys

> **Plain English:** This is the master config file. Stealing it is like getting a copy of someone's house keys AND their list of passwords.

**Attacker capabilities:**
- **Remote gateway access** — If the gateway port is exposed (not loopback-only), the stolen `gateway.auth.token` grants WebSocket full-duplex control and HTTP tool invocation
- **Remote code execution** — Via `POST /tools/invoke` with the exec tool, an attacker can run arbitrary shell commands (`src/gateway/tools-invoke-http.ts:127-324`)
- **API billing fraud** — If `auth-profiles.json` was also stolen (likely), attacker gets all AI provider API keys (Anthropic, OpenAI, etc.)
- **Workspace reconnaissance** — `agents[].dir` reveals workspace paths, aiding further targeted attacks

#### `device.json` — ED25519 Private Key

> **Plain English:** This is the device's fingerprint. Stealing it lets an attacker pretend to BE your device.

**Attacker capabilities:**
- **Device impersonation** — Forge device signatures using `signDevicePayload()` (`src/infra/device-identity.ts:125-129`)
- **Bypass device authentication** — Impersonate the device in pairing flows
- **Persistent identity theft** — Device ID is a SHA256 fingerprint of the public key; compromised identity persists until keys are regenerated

#### `soul.md` — Agent System Prompt

> **Plain English:** This is the agent's personality and rules. Stealing it lets an attacker learn how to manipulate the agent.

**Attacker capabilities:**
- **Craft targeted prompt injections** — Knowing the agent's guidelines enables precision attacks that work within the agent's trust model
- **Learn operational details** — Workspace configuration, tool preferences, behavioral constraints
- **Lower severity** than key theft, but enables more effective social engineering

#### `auth-profiles.json` (not in article, likely also stolen)

> **Plain English:** This file stores ALL your AI provider API keys. The article didn't mention it, but the same file-grabbing routine almost certainly captured it too.

**Location:** `~/.openclaw/agents/*/agent/auth-profiles.json`

**Attacker capabilities:**
- **Full AI provider access** — Anthropic, OpenAI, Google, and any other configured provider API keys
- **Billing fraud** — Run inference against victim's accounts
- **Data access** — Some providers allow accessing conversation history via API

#### `credentials/` directory (not in article, likely also stolen)

**Location:** `~/.openclaw/credentials/`

**Attacker capabilities:**
- **Telegram bot hijacking** — Bot tokens grant full bot control
- **Discord bot hijacking** — Same as Telegram
- **WhatsApp session hijacking** — Session credentials allow reading/sending messages as the victim

#### Impact Summary

| File | Attacker Capability | Severity | Existing Mitigation |
|------|---------------------|----------|---------------------|
| `openclaw.json` | Remote gateway RCE (if port exposed) | CRITICAL | Loopback-only default; `0o600` perms |
| `device.json` | Device impersonation, forge signatures | HIGH | `0o600` perms; no encryption at rest |
| `soul.md` | Targeted prompt injection crafting | MEDIUM | Workspace-level file permissions |
| `auth-profiles.json` | All AI API keys → billing fraud | CRITICAL | `0o600` perms; no encryption at rest |
| `credentials/*.json` | Telegram/Discord/WhatsApp hijacking | CRITICAL | `0o600` perms; no encryption at rest |

---

### Attack Chain

```
Infection                     File Exfiltration                  Exploitation
──────────                    ──────────────────                 ────────────

Phishing email/               Vidar scans                       Attacker extracts
malicious download            common file paths                 gateway token
    │                             │                                 │
    ▼                             ▼                                 │
User runs                     Grabs:                                │
infected binary               • ~/.openclaw/openclaw.json           │
    │                         • ~/.openclaw/identity/device.json    │
    ▼                         • <workspace>/SOUL.md                 │
Vidar installs                • ~/.openclaw/agents/*/               │
on victim machine               auth-profiles.json (likely)        │
                              • ~/.openclaw/credentials/ (likely)   │
                                  │                                 │
                                  ▼                                 ▼
                              Exfil to C2 server ──────────► If port exposed:
                                                             POST /tools/invoke
                                                             → shell exec → RCE

                                                             If loopback-only:
                                                             API keys still stolen
                                                             → billing fraud
                                                             → messaging hijack
```

**Comparison: Atomic Stealer vs Vidar**

| Aspect | Atomic Stealer (ClawHavoc) | Vidar (Hudson Rock) |
|--------|---------------------------|---------------------|
| Distribution | Targeted — social engineering via fake ClawHub skill "prerequisites" | Opportunistic — commodity malware via phishing/malvertising |
| OpenClaw targeting | Intentional (ClawHavoc campaign specifically targets OpenClaw users) | Incidental (broad file-grabbing routine matches OpenClaw paths) |
| Files stolen | Browser passwords, crypto wallets, SSH keys + OpenClaw creds | Same broad sweep, confirmed to include OpenClaw config files |
| Key insight | Social engineering bypasses all scanning | OpenClaw files are now in commodity malware's target lists |

**Critical gap:** Gateway tokens are NOT device-bound — there is no rotation mechanism, no expiry, and no way to invalidate a token without manually changing the config.

---

### Evolving Threat: Dedicated OpenClaw Modules (Forward-Looking)

Hudson Rock predicts that **infostealer developers will likely release dedicated OpenClaw-targeting modules** — analogous to how infostealers now have custom modules for Chrome passwords, Telegram sessions, and cryptocurrency wallets.

**Historical pattern:**
1. **Initial phase** (current): Broad file-grabbing routines incidentally capture OpenClaw files by matching file extensions and directory names
2. **Dedicated module phase** (predicted): Custom parsers that understand `openclaw.json` JSON5 format, extract tokens directly, and target specific high-value paths like `auth-profiles.json`
3. **Active exploitation phase**: Automated pipelines that extract tokens → scan for exposed ports → attempt RCE

**What dedicated modules would target:**
- Config parsing — JSON5 format understanding, extract `gateway.auth.token` directly
- Key extraction — Parse ED25519 keys from `device.json`
- API key harvesting — Enumerate all `auth-profiles.json` files across agent directories
- Session hijacking — Extract messaging platform credentials from `credentials/`
- Auto-discovery — Enumerate `~/.openclaw/` and all subdirectories

**Timeline estimate:** Based on the Chrome/Telegram module precedent, dedicated modules typically appear within 3-6 months of initial broad-sweep captures.

---

### New Evasion: ClawHub Lookalike Website Bypass

**Source:** OpenSourceMalware team (separate finding from Hudson Rock)

**How it works:**
1. Attacker publishes a skill on ClawHub with clean, legitimate-looking code
2. The skill's documentation or functionality includes a URL pointing to a **lookalike OpenClaw website** (e.g., `openclaw-tools.ai`, `openclaw-plugins.com`)
3. The actual malware payload is hosted on the lookalike website, not in the skill package
4. User installs the skill, which appears clean to both VirusTotal and the local scanner
5. The skill directs the agent (or user) to download additional resources from the malicious website

**Why VirusTotal misses it:**
- The skill package itself contains no malware — only a URL or redirect
- VirusTotal scans the skill package, not external websites referenced by the skill
- The local scanner checks for dangerous code patterns in JS/TS files, not URL destinations

**Mitigations:**
- Be suspicious of skills that require downloading external resources
- Verify any URLs in skill documentation against official sources
- Check domain registration dates for any sites a skill references
- Use `openclaw.ai` and `docs.openclaw.ai` as the only trusted domains

**See also:** [ClawHub Marketplace Risks](../05-worst-case-security/clawhub-marketplace-risks.md), [Ecosystem Threats § ClawHub](./ecosystem-security-threats.md#6-clawhub-malicious-skills-clawhavoc-campaign)

---

### Moltbook Account Deletion (OX Security Finding)

**Source:** OX Security research (cited in THN article, Feb 16, 2026)

**What they found:** AI agent accounts registered on Moltbook cannot be deleted once created. There is no account deletion mechanism, self-service or otherwise.

**Privacy implications:**
- **GDPR "right to erasure"** (Article 17) — European users cannot exercise their right to delete personal data associated with their agent
- **Data persistence** — Agent profile data, posts, comments, and any credentials shared during registration persist indefinitely
- **No recourse** — If you registered your agent via the Moltbook skill, that data persists on Moltbook's servers with no deletion path

**OpenClaw user impact:**
- If you installed the Moltbook skill and registered your agent, the registration data (agent name, credentials, any profile information) remains on Moltbook's infrastructure
- The Supabase database exposure (Jan 30, 2026) already demonstrated that Moltbook's data handling has had security gaps

**See also:** [What is Moltbook?](../07-moltbook/what-is-moltbook.md)

---

### What's Already Mitigated vs. Still Open

| Risk | Status | Mitigation | Documentation |
|------|--------|------------|---------------|
| Gateway network exposure | MITIGATED (default) | Loopback-only binding by default (`src/gateway/net.ts:273-278`) | [Hardening checklist](../04-privacy-safety/hardening-checklist.md) |
| Loopback fallback to 0.0.0.0 | OPEN (edge case) | If loopback bind fails, falls back to all-interfaces (`src/gateway/net.ts:280`) | [Threat model](../04-privacy-safety/threat-model.md) |
| File permissions | PARTIALLY MITIGATED | `0o600` on config/keys; `openclaw security audit --fix` can repair | [Hardening checklist](../04-privacy-safety/hardening-checklist.md) |
| Encryption at rest | OPEN | No encryption — credentials protected only by filesystem permissions | [Mac Mini risks](../05-worst-case-security/mac-mini-risks.md) |
| Token rotation | OPEN | No expiry, no rotation mechanism — tokens are static until manually changed | — |
| Token device-binding | OPEN | Gateway tokens are not bound to any device; any bearer can authenticate | — |
| Auth rate limiting | MITIGATED | 10 failures / 60s → 5-minute lockout (`src/config/types.gateway.ts:124-133`) | [Threat model](../04-privacy-safety/threat-model.md) |
| VirusTotal scanning | MITIGATED (partial) | 6-step pipeline; does not catch social engineering or lookalike website bypass | [ClawHub risks](../05-worst-case-security/clawhub-marketplace-risks.md) |
| Local skill scanner | MITIGATED (partial) | Pattern-based; does not scan `.md`/`.mmd` files or external URLs | [ClawHub risks](../05-worst-case-security/clawhub-marketplace-risks.md) |
| Endpoint protection | USER RESPONSIBILITY | No built-in AV/EDR; relies on user's OS-level security | This document |
| ClawHub lookalike bypass | OPEN | No scanning of external URLs referenced by skills | This document |
| Moltbook account deletion | OPEN (third-party) | No deletion mechanism; Moltbook is external to OpenClaw | [What is Moltbook?](../07-moltbook/what-is-moltbook.md) |

---

### Emergency Response: If You Suspect Config Theft

#### Tier 1: Immediate (do now)

1. **Rotate your gateway token:**
   ```bash
   openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
   ```
2. **Regenerate device keys:** Delete and let OpenClaw regenerate:
   ```bash
   rm ~/.openclaw/identity/device.json
   openclaw gateway restart
   ```
3. **Rotate ALL AI provider API keys:**
   - Anthropic: [console.anthropic.com](https://console.anthropic.com) → API Keys → Regenerate
   - OpenAI: [platform.openai.com](https://platform.openai.com) → API Keys → Regenerate
   - Google AI: Cloud Console → Credentials → Regenerate
   - Any other providers in your `auth-profiles.json`
4. **Rotate messaging tokens:**
   - Telegram: @BotFather → /revoke → create new token
   - Discord: Developer Portal → Bot → Regenerate Token
   - WhatsApp: Phone app → Settings → Linked Devices → Log out all

#### Tier 2: Audit (within 24 hours)

5. **Check file permissions:**
   ```bash
   openclaw security audit --fix
   ```
6. **Review gateway access logs** for unauthorized connections
7. **Scan for malware:**
   - macOS: Malwarebytes, ClamAV, or Apple XProtect
   - Linux: ClamAV, rkhunter
8. **Check for persistence mechanisms:**
   ```bash
   # macOS
   launchctl list | grep -v "com.apple"
   ls -la ~/Library/LaunchAgents/

   # Linux
   systemctl list-units --type=service --state=running
   crontab -l
   ```

#### Tier 3: Prevention (ongoing)

9. **Install endpoint protection** — AV/EDR software (Malwarebytes, CrowdStrike Falcon, SentinelOne)
10. **Enable disk encryption** — FileVault (macOS) or LUKS (Linux)
11. **Run as a dedicated user** — Isolate OpenClaw from your main user account
12. **Disable cloud sync** for `~/.openclaw/` — Do not sync to iCloud, Dropbox, Google Drive
13. **Keep gateway loopback-only** — Never expose to the network without SSH tunnel or Tailscale

**See also:** [Incident Response Playbook](../05-worst-case-security/incident-response.md) for comprehensive recovery procedures.

---

### Cross-References

| Document | Relationship |
|----------|-------------|
| [Threat model](../04-privacy-safety/threat-model.md) | Local disk + secrets risk (§4); credential file storage |
| [Hardening checklist](../04-privacy-safety/hardening-checklist.md) | Disk protection (§8); endpoint protection |
| [Mac Mini risks](../05-worst-case-security/mac-mini-risks.md) | Credential locations tree; physical access attacks |
| [Ecosystem security threats](./ecosystem-security-threats.md) | Session token stealing (§3); ClawHub malicious skills (§6) |
| [ClawHub marketplace risks](../05-worst-case-security/clawhub-marketplace-risks.md) | VirusTotal scanning; Atomic Stealer; lookalike bypass |
| [SecurityScorecard STRIKE report](./securityscorecard-strike-report.md) | Actual exposure numbers (28K IPs, not "hundreds of thousands") |
| [Incident response playbook](../05-worst-case-security/incident-response.md) | Full recovery procedures |
| [What is Moltbook?](../07-moltbook/what-is-moltbook.md) | Moltbook account deletion concern |
