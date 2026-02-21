# Skills.sh: Unverified Third-Party Skill Distribution

> **Time:** 5 minutes
> **Difficulty:** Beginner-friendly with practical examples
> **Severity:** HIGH (unverified supply chain risk when third-party skills are manually imported or auto-imported by external tooling)

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

> **The Analogy:** If ClawHub is like an unofficial app store, skills.sh is like a bulletin board at a coffee shop where anyone can pin a flyer advertising software. There's no built-in front door in OpenClaw for skills.sh, but users or wrappers can still pull those flyers into their local skills folders.

Skills.sh is a third-party website that aggregates and distributes OpenClaw skills. In the current OpenClaw core codebase, skills are loaded from local directories and ClawHub workflows; there is no built-in skills.sh discovery pipeline. The risk remains high when operators (or external automation around OpenClaw) import unvetted skills from skills.sh and then run install steps.

### Platform Comparison

| Feature | ClawHub | skills.sh | OpenClaw core (current repo) |
|---------|---------|-----------|------------------------------|
| **Publisher verification** | GitHub account >= 1 week | None | N/A (local loading from filesystem/workspace) |
| **Automated scanning** | VirusTotal + Gemini LLM (Feb 2026) | None | N/A for remote marketplaces |
| **Local scanning** | Local scanner + `openclaw security audit` | Local scanner + `openclaw security audit` | Local scanner warnings + `openclaw security audit` |
| **Code review** | None (human) | None | Operator-dependent |
| **Sandboxing** | Default: none; opt-in via sandbox config | Default: none; opt-in via sandbox config | Depends on your `tools.*`/sandbox config |
| **Auto-discovery** | Manual (`npx clawhub ...`) | External/third-party tooling only | No built-in skills.sh auto-discovery |
| **Takedown process** | Report-based (3+ reports) | Unknown | N/A |

**Key difference:** ClawHub at least runs marketplace-side scanning before publish. Skills.sh has no equivalent guardrail, and imported skills bypass any ClawHub-specific checks.

**Scanner limitations:** The local scanner (`openclaw security audit`) only inspects JS/TS file types (`.js`, `.ts`, `.mjs`, `.cjs`, `.mts`, `.cts`, `.jsx`, `.tsx`) — it does **not** scan `.md`, `.sh`, `.py`, or `.mmd` files. It also caps at 500 files and 1 MB per file (silently skipping anything over-limit), and warns on suspicious patterns but does **not** block installation. See `src/security/skill-scanner.ts` and `src/agents/skills-install.ts`.

Source: [YouTube video](https://www.youtube.com/watch?v=_CzEmKTk5Rs) [[24:42](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=1482)]

---

## B. External "Find Skills" Automation Attack

### The Pipeline

> **The Analogy:** Imagine your assistant has a custom "find me apps" wrapper that scrapes the coffee shop bulletin board, downloads whatever looks useful, and installs dependencies with minimal review. The bulletin board has no security guards.

This is a **third-party automation pattern**, not a built-in OpenClaw core workflow (as of Feb 21, 2026):

```
User asks agent to perform a task
  ↓
Agent doesn't have a matching skill
  ↓
External wrapper/tool searches skills.sh for matches
  ↓
Wrapper downloads skill files into local workspace
  ↓
Operator or workflow runs install steps for that skill
  ↓
If malicious: credential theft, data exfiltration, persistence
```

### Why This Is Dangerous

| Risk | Description |
|------|-------------|
| **No human review** | Third-party wrappers can automate discovery/import with little or no review |
| **No scanning** | Skills.sh has no VirusTotal or equivalent scanning |
| **Trust by proximity** | Users may assume "found by automation" means vetted |
| **Operational impact** | Untrusted skill metadata/install steps can trigger risky dependency installs |
| **Persistence** | Once imported and enabled, malicious prompts/dispatch rules persist across sessions |

### Attack Scenario

```
Attacker publishes "data-analysis-pro" on skills.sh
  ↓
User tells agent: "Analyze this spreadsheet"
  ↓
External automation imports "data-analysis-pro" into workspace
  ↓
Operator/workflow runs install steps with minimal review
  ↓
Malicious dependency/script/tool-dispatch reads ~/.openclaw/credentials/* and exfiltrates to attacker
  ↓
User sees spreadsheet analysis output (skill works as advertised)
  ↓
Credentials already stolen — user has no indication of compromise
```

Source: [YouTube video](https://www.youtube.com/watch?v=_CzEmKTk5Rs) [[25:07](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=1507)], [[25:35](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=1535)]

---

## C. Practical Defenses

### Use Current Hardening Controls (Not Deprecated Keys)

```bash
# Restrict exec behavior (current keys)
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.ask on-miss

# Avoid loading extra skill directories unless explicitly trusted
openclaw config set skills.load.extraDirs []

# Manage executable allowlist entries used by exec approvals
openclaw approvals get
openclaw approvals allowlist add "/usr/bin/uname"
```

`skills.autoDiscover`, `skills.requireApproval`, and `skills.sources` are not in the current OpenClaw config schema.

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
| `tools.exec.security` | `allowlist` | Limits shell execution capability |
| `tools.exec.ask` | `on-miss` or `always` | Requires approval flow when not allowlisted |
| `skills.load.extraDirs` | `[]` (or trusted dirs only) | Reduces untrusted skill ingestion paths |
| `agents.list[].skills` | explicit allowlist | Restricts which skills a given agent can load |

### Prefer ClawHub Over Skills.sh

While neither platform is perfect, ClawHub is the source currently surfaced in OpenClaw docs/system prompt and at least provides:
- VirusTotal scanning (70+ AV engines + Gemini LLM review)
- Daily rescans of previously approved skills
- Publisher identity tied to GitHub accounts
- Community reporting and takedown process

If a skill is available on both platforms, prefer the ClawHub version and still review the skill locally before enabling/installing dependencies.

---

## Cited Sources

- [YouTube: OpenClaw Security Deep Dive](https://www.youtube.com/watch?v=_CzEmKTk5Rs) — skills.sh coverage at [[24:42](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=1482)], [[25:07](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=1507)], [[25:35](https://www.youtube.com/watch?v=_CzEmKTk5Rs&t=1535)]

---

> **See also:** [Cline CLI Supply Chain Attack](../08-security-analysis/cline-supply-chain-attack.md) — real-world example of npm `postinstall` hook abuse (Feb 2026)

## Related Documentation

- [ClawHub Marketplace Risks](./clawhub-marketplace-risks.md) - ClawHub-specific supply chain analysis
- [Cross-Cutting Vulnerabilities](./cross-cutting.md) - Supply chain risks section
- [Threat Model](../04-privacy-safety/threat-model.md) - Plugin/extension risks
- [Hardening Checklist](../04-privacy-safety/hardening-checklist.md) - Plugin safety guidance
- [Prompt Injection Attacks](./prompt-injection-attacks.md) - 27 attack examples; unverified skills are a delivery vector for injection payloads
