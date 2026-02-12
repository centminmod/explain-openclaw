# Skills.sh: Unverified Third-Party Skill Distribution

> **Time:** 5 minutes
> **Difficulty:** Beginner-friendly with practical examples
> **Severity:** HIGH (unverified supply chain with auto-install capabilities)

---

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
- Worst-case security
  - [Overview](./README.md)
  - [Mac Mini risks](./mac-mini-risks.md)
  - [VPS risks](./vps-risks.md)
  - [Moltworker risks](./moltworker-risks.md)
  - [Cross-cutting vulnerabilities](./cross-cutting.md)
  - [ClawHub marketplace risks](./clawhub-marketplace-risks.md)
  - [Skills.sh risks](./skills-sh-risks.md) (this page)
  - [Prompt injection attacks](./prompt-injection-attacks.md)
  - [Misconfiguration examples](./misconfiguration-examples.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

## A. What Is Skills.sh?

### Overview

> **The Analogy:** If ClawHub is like an unofficial app store, skills.sh is like a bulletin board at a coffee shop where anyone can pin a flyer advertising software. There's no store, no review process, no front door — just URLs pointing to skill files that your agent can discover and install automatically.

Skills.sh is a third-party website that aggregates and distributes OpenClaw skills. Unlike ClawHub (which at least has VirusTotal scanning since Feb 2026), skills.sh has no automated security scanning, no vetting process, and no publisher accountability.

### Platform Comparison

| Feature | ClawHub | skills.sh | npm |
|---------|---------|-----------|-----|
| **Publisher verification** | GitHub account >= 1 week | None | npm account |
| **Automated scanning** | VirusTotal + Gemini LLM (Feb 2026) | None | None |
| **Local scanning** | OpenClaw skill scanner at install | OpenClaw skill scanner at install | `npm audit` |
| **Code review** | None (human) | None | None |
| **Sandboxing** | None (runs in-process) | None (runs in-process) | None |
| **Auto-discovery** | Manual search on clawhub.ai | "Find Skills" automation pipeline | Manual `npm search` |
| **Takedown process** | Report-based (3+ reports) | Unknown | Report-based |

**Key difference:** ClawHub at least runs VirusTotal scanning before publishing. Skills.sh skills bypass this entirely — they go straight from an anonymous publisher to your Gateway process.

Source: [YouTube video](https://www.youtube.com/watch?v=_CzEmKTk5Rs) [[24:42](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=1482)]

---

## B. "Find Skills" Automation Attack

### The Pipeline

> **The Analogy:** Imagine your assistant has a "find me apps" feature that automatically searches the coffee shop bulletin board, downloads whatever looks useful, and installs it — all without asking you. The bulletin board has no security guards.

The "Find Skills" feature in some OpenClaw configurations creates an automated pipeline:

```
User asks agent to perform a task
  ↓
Agent doesn't have a matching skill
  ↓
"Find Skills" searches skills.sh for matches
  ↓
Agent auto-discovers a skill that claims to do the job
  ↓
Skill is installed and runs in-process with full Gateway access
  ↓
If malicious: credential theft, data exfiltration, persistence
```

### Why This Is Dangerous

| Risk | Description |
|------|-------------|
| **No human review** | The discovery-to-install pipeline can execute without user approval |
| **No scanning** | Skills.sh has no VirusTotal or equivalent scanning |
| **Trust by proximity** | Users assume skills found by their agent are vetted |
| **Full access** | Installed skills run in-process with access to credentials, sessions, and network |
| **Persistence** | Once installed, a malicious skill continues running across sessions |

### Attack Scenario

```
Attacker publishes "data-analysis-pro" on skills.sh
  ↓
User tells agent: "Analyze this spreadsheet"
  ↓
Agent searches skills.sh, finds "data-analysis-pro"
  ↓
Skill installs automatically (or with minimal prompt)
  ↓
Skill reads ~/.openclaw/credentials/* and exfiltrates to attacker
  ↓
User sees spreadsheet analysis output (skill works as advertised)
  ↓
Credentials already stolen — user has no indication of compromise
```

Source: [YouTube video](https://www.youtube.com/watch?v=_CzEmKTk5Rs) [[25:07](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=1507)], [[25:35](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=1535)]

---

## C. Practical Defenses

### Disable Auto-Discovery

```bash
# Check if "Find Skills" or auto-install is enabled
openclaw config get skills.autoDiscover

# Disable automatic skill discovery
openclaw config set skills.autoDiscover false

# Require manual approval for all skill installations
openclaw config set skills.requireApproval true
```

### Manual Skill Vetting

Before installing any skill from skills.sh:

```bash
# 1. Download but don't install — inspect first
# Check the skill's source code manually

# 2. Look for red flags
grep -r "eval\|Function\|fetch\|axios\|http\|child_process" <skill-directory>/

# 3. Check for hidden files (especially .mmd files)
find <skill-directory>/ -name ".*" -type f
ls -la <skill-directory>/

# 4. Review network calls
grep -r "https\?://" <skill-directory>/ | grep -v "node_modules"

# 5. Run OpenClaw's local scanner manually
openclaw security audit --deep
```

### Configuration Hardening

| Setting | Recommended Value | Why |
|---------|-------------------|-----|
| `skills.autoDiscover` | `false` | Prevents automatic skill search |
| `skills.requireApproval` | `true` | Forces human approval before install |
| `tools.shell.security` | `allowlist` | Limits what installed skills can execute |
| `skills.sources` | `["clawhub"]` only | Excludes skills.sh from search sources |

### Prefer ClawHub Over Skills.sh

While neither platform is perfect, ClawHub at least provides:
- VirusTotal scanning (70+ AV engines + Gemini LLM review)
- Daily rescans of previously approved skills
- Publisher identity tied to GitHub accounts
- Community reporting and takedown process

If a skill is available on both platforms, always prefer the ClawHub version.

---

## Cited Sources

- [YouTube: OpenClaw Security Deep Dive](https://www.youtube.com/watch?v=_CzEmKTk5Rs) — skills.sh coverage at [[24:42](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=1482)], [[25:07](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=1507)], [[25:35](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=1535)]

---

## Related Documentation

- [ClawHub Marketplace Risks](./clawhub-marketplace-risks.md) - ClawHub-specific supply chain analysis
- [Cross-Cutting Vulnerabilities](./cross-cutting.md) - Supply chain risks section
- [Threat Model](../04-privacy-safety/threat-model.md) - Plugin/extension risks
- [Hardening Checklist](../04-privacy-safety/hardening-checklist.md) - Plugin safety guidance
- [Prompt Injection Attacks](./prompt-injection-attacks.md) - 27 attack examples; unverified skills are a delivery vector for injection payloads
