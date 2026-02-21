> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Open PRs](./open-upstream-prs.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Hudson Rock](./hudson-rock-infostealer-analysis.md) | [Cline Supply Chain](./cline-supply-chain-attack.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## Cisco AI Defense: Blog Post Claims + Skill Scanner Evaluation (Feb 2026)

> **Source (blog):** "Personal AI Agents Like OpenClaw Are a Security Nightmare" — Cisco AI Defense team blog post (Feb 2026)
> **Source (tool):** `cisco-ai-skill-scanner` — open-source multi-engine skill scanner ([GitHub](https://github.com/cisco-ai-defense/skill-scanner))
> **Audited against:** OpenClaw v2026.2.9 (`package.json:3`)

---

### Part A: Blog Post Claim Verification

The Cisco blog post identifies four risk categories for OpenClaw deployments and reports 9 findings (2 critical, 5 high) from scanning a deliberately malicious test skill ("What Would Elon Do?"). Below we verify each claim against source code.

#### Summary of claims

| # | Risk claim | Cisco severity | Our verdict | Key evidence |
|---|-----------|---------------|-------------|-------------|
| 1 | Privileged execution — skills can run arbitrary commands | High | TRUE but OVERSTATED | Gated by exec approval system + env var blocklist + sandbox |
| 2 | Credential exposure — plaintext tokens on disk | High | PARTIALLY TRUE | 0o600 permissions + symlink blocking; no encryption-at-rest (known gap) |
| 3 | Extended attack surface via messaging integrations | Medium | TRUE but OVERSTATED | Pairing system + allowFrom lists + group policy defaults restrict access |
| 4 | "Inadequate built-in security" | Critical | FALSE | 65+ audit checks, install-time scanning, mandatory auth, SSRF protection, RBAC |

---

#### Claim 1: Privileged execution

**Cisco says:** Skills can run arbitrary shell commands with the operator's full system privileges, creating a path to host compromise.

**What the code actually does:**

1. **Exec approval system** (`src/infra/exec-approvals.ts:1-50`) — every shell command goes through `requiresExecApproval()` / `evaluateShellAllowlist()` with configurable policies (`deny` / `allowlist` / `full`) and per-agent scoping
2. **Environment variable blocklist** (`src/agents/bash-tools.exec-runtime.ts:34-51`) — 17 dangerous env vars (`LD_PRELOAD`, `DYLD_INSERT_LIBRARIES`, `NODE_OPTIONS`, `BASH_ENV`, etc.) are blocked, plus prefix-based blocking for `DYLD_*` and `LD_*` (line 52)
3. **Sanitization enforcement** (`src/agents/bash-tools.exec-runtime.ts:56-80`) — `validateHostEnv()` throws `Security Violation` errors if blocked vars or PATH modifications are detected
4. **Sandbox support** — Docker exec args (`buildDockerExecArgs`) provide container-level isolation

**Verdict:** TRUE that skills can invoke shell commands — this is by design for an AI agent framework. OVERSTATED because execution is gated by a multi-layer approval system, not arbitrary.

---

#### Claim 2: Credential exposure

**Cisco says:** Authentication tokens and credentials are stored as plaintext JSON on disk, vulnerable to exfiltration.

**What the code actually does:**

1. **File permissions** (`src/infra/fs-safe.ts:57-58`) — files opened with `O_NOFOLLOW` on non-Windows platforms, blocking symlink-based exfiltration
2. **Symlink protection** (`src/infra/fs-safe.ts:67-68, 74-77`) — double-checked via `isSymlinkOpenError()` catch and explicit `lstat().isSymbolicLink()` check
3. **Path traversal prevention** (`src/pairing/pairing-store.ts:53-62`) — `safeChannelKey()` strips `../`, `/`, `\`, and other path traversal characters
4. **Root escape prevention** (`src/infra/fs-safe.ts:53-54`) — `openFileWithinRoot()` verifies resolved path stays within root directory
5. **Known gap:** credentials are stored as JSON without encryption at rest (acknowledged; mitigated by 0o600 and OS-level controls)

**Verdict:** PARTIALLY TRUE — plaintext storage is a real defense-in-depth gap (also flagged by CVE-2026-25253, now patched for the path traversal vector). The file permission enforcement and symlink blocking are robust, but encryption at rest would be stronger.

---

#### Claim 3: Extended attack surface via messaging

**Cisco says:** OpenClaw's messaging integrations (Discord, Telegram, etc.) extend the attack surface beyond localhost to remote users who can trigger agent actions.

**What the code actually does:**

1. **Pairing system** (`src/pairing/pairing-store.ts:12-25`) — 8-character codes with 60-minute TTL, max 3 pending, with lockfile-based concurrency control
2. **AllowFrom lists** (`src/pairing/pairing-store.ts:42-45`) — per-channel allowlists restrict which users can interact with the agent
3. **Gateway bind default** (`src/gateway/server.impl.ts:112-118`) — defaults to `loopback` (127.0.0.1), not LAN/public
4. **Mandatory auth for non-loopback** (`src/gateway/server-runtime-config.ts:99-103`) — server refuses to start on non-loopback without a configured auth token/password

**Verdict:** TRUE that messaging integrations extend the attack surface — that is inherent to any messaging-capable application. OVERSTATED because pairing + allowlists + bind defaults significantly restrict who can reach the agent.

---

#### Claim 4: "Inadequate built-in security"

**Cisco says:** OpenClaw lacks adequate built-in security controls, relying primarily on the user to secure their deployment.

**What the code actually does:**

1. **Audit framework** (`src/security/audit.ts:1-80`, `src/security/audit-extra.ts`) — 65+ security checks across 12 categories (filesystem permissions, secrets in config, attack surface, exposure matrix, hooks hardening, model hygiene, plugin trust, etc.)
2. **Install-time skill scanning** (`src/security/skill-scanner.ts:38-47`) — regex-based scanning of `.js`/`.ts` files at skill install time
3. **Loopback default** (`src/gateway/server.impl.ts:112-118`) — gateway binds to 127.0.0.1 by default
4. **Mandatory auth enforcement** (`src/gateway/server-runtime-config.ts:99-103`) — throws error if binding to non-loopback without auth credentials
5. **SSRF protection** (`src/infra/net/ssrf.ts:167-272`) — DNS pinning with private IP blocking (10.x, 127.x, 169.254.x, 172.16-31.x, 192.168.x, 100.64-127.x, IPv6 link-local/ULA including full-form IPv4-mapped IPv6, plus metadata hostnames)
6. **SSRF policy enforcement** (`src/infra/net/ssrf.ts:337-389`) — `resolvePinnedHostnameWithPolicy()` blocks both hostname-based and resolved-IP-based private network access
7. **RBAC on every call** (`src/gateway/server-methods.ts:105-175`) — `authorizeGatewayMethod()` enforces role + scope checks (operator/node roles, admin/approvals/pairing/read/write scopes) on every gateway method

**Verdict:** FALSE — OpenClaw has extensive built-in security. The blog post appears to have evaluated an older version or did not examine the codebase beyond surface-level claims.

---

#### "What Would Elon Do?" test skill

The blog post demonstrates scanning a deliberately malicious skill that exfiltrates environment variables and installs a reverse shell. The scanner correctly flagged this with 2 critical and 5 high findings.

**Assessment:** This demonstrates a real supply chain risk — malicious skills can harm users who install them without review. However, this is a **supply chain risk**, not a codebase vulnerability. OpenClaw already documents this threat:

- **ClawHub/VirusTotal scanning** — 70+ AV engines + Gemini LLM analysis at publish time with daily rescans
- **Install-time scanning** — built-in skill scanner runs at install (`src/security/skill-scanner.ts`)
- **Exec approval system** — blocks unauthorized command execution even from installed skills

Cross-reference: [ClawHub marketplace risks](../05-worst-case-security/clawhub-marketplace-risks.md)

---

#### Blog post verdict

| # | Risk claim | Verdict |
|---|-----------|---------|
| 1 | Privileged execution | TRUE but OVERSTATED — gated by approval + blocklist + sandbox |
| 2 | Credential exposure | PARTIALLY TRUE — plaintext with robust file protections; known gap |
| 3 | Extended attack surface | TRUE but OVERSTATED — pairing + allowlists restrict access |
| 4 | "Inadequate built-in security" | FALSE — 65+ audit checks, mandatory auth, SSRF protection, RBAC |

**Result: 0 of 4 risk claims accurately characterize the current codebase without significant qualification.** Claims 1-3 identify real capabilities but overstate risk by ignoring the mitigations. Claim 4 is factually incorrect.

---

### Part B: Skill Scanner Tool Evaluation

#### Tool overview

| Field | Detail |
|-------|--------|
| **Repository** | `cisco-ai-defense/skill-scanner` (GitHub) |
| **License** | Apache 2.0 |
| **Language** | Python 3.10+ |
| **Engines** | Static (regex/YARA), Behavioral (Python AST), LLM (Cisco AI Defense / OpenAI), VirusTotal |
| **Output** | JSON, SARIF (CI/CD integration) |
| **Supported formats** | `.py`, `.js`, `.ts`, `.md` (SKILL.md), `.yaml` |

#### Detection engines vs OpenClaw applicability

| Engine | What it detects | OpenClaw relevance |
|--------|----------------|-------------------|
| **Static (regex)** | Hardcoded secrets, suspicious URLs, known malicious patterns | High — complements built-in scanner |
| **Static (YARA)** | Malware signatures, obfuscated code patterns | Medium — adds signature-based detection OpenClaw lacks |
| **Behavioral (AST)** | Dynamic imports, eval(), exec(), subprocess calls in Python | Low — most OpenClaw skills are markdown + JS/TS, not Python |
| **LLM analysis** | Prompt injection, social engineering, hidden instructions in .md files | **Critical** — fills the key gap (see below) |
| **VirusTotal** | Known malware hashes across 70+ engines | Low — ClawHub already integrates VirusTotal |

#### Gap analysis: What Cisco catches that existing tools don't

The key finding from evaluating this tool is a gap in OpenClaw's built-in scanner:

```typescript
// src/security/skill-scanner.ts:38-47
const SCANNABLE_EXTENSIONS = new Set([
  ".js",  ".ts",  ".mjs", ".cjs",
  ".mts", ".cts", ".jsx", ".tsx",
]);
```

**The built-in scanner does not scan `.md` files.** Since OpenClaw skills are defined primarily by `SKILL.md` files (which contain the system prompt, tool definitions, and behavioral instructions), prompt injection attacks embedded in SKILL.md files are invisible to the built-in scanner.

Cisco's LLM engine can analyze SKILL.md content for:
- Hidden instructions ("ignore previous instructions and...")
- Encoded/obfuscated prompt injection
- Social engineering patterns ("you are now in developer mode...")
- Data exfiltration instructions disguised as normal behavior

This is a meaningful detection gap that existing tools don't address:
- **OpenClaw built-in** — scans JS/TS only, not .md
- **ClawHub/VirusTotal** — focuses on malware binaries, not prompt injection in text
- **Koi Security Scanner** — third-party, targets code patterns not prompt content

> **See:** [Prompt Injection Attacks](../05-worst-case-security/prompt-injection-attacks.md) for 27 documented examples of the injection techniques these scanners aim to detect.

#### Beyond SKILL.md: all persistent .md files are unscanned

The SKILL.md gap extends to **all persistent `.md` files** in the workspace. Two separate injection paths exist, each with different trust levels and attack surface:

**Path 1 — Bootstrap files (system prompt injection, high trust):**

Nine named `.md` files are loaded by `loadWorkspaceBootstrapFiles()` (`src/agents/workspace.ts:441-495`) and injected directly into the system prompt via `buildBootstrapContextFiles()` (`src/agents/pi-embedded-helpers/bootstrap.ts:187-239`). They appear as fully trusted context with **no content validation** — only truncation at 20,000 characters per file (`src/agents/pi-embedded-helpers/bootstrap.ts:85`).

| Bootstrap file | Purpose | Max chars | Injection path |
|----------------|---------|-----------|----------------|
| `AGENTS.md` | Multi-agent configuration | 20,000 | System prompt |
| `SOUL.md` | Agent personality / behavioral instructions | 20,000 | System prompt |
| `TOOLS.md` | Tool usage guidance | 20,000 | System prompt |
| `IDENTITY.md` | Agent identity definition | 20,000 | System prompt |
| `USER.md` | User-specific context | 20,000 | System prompt |
| `HEARTBEAT.md` | Heartbeat/health configuration | 20,000 | System prompt |
| `BOOTSTRAP.md` | General bootstrap instructions | 20,000 | System prompt |
| `MEMORY.md` | Persistent memory context | 20,000 | System prompt |
| `memory.md` | Persistent memory context (lowercase variant) | 20,000 | System prompt |

Source: `src/agents/workspace.ts:30-31` (file list), `src/agents/pi-embedded-helpers/bootstrap.ts:85` (truncation limit)

**Total unscanned system prompt attack surface: 9 x 20,000 = 180,000 characters.**

**Path 2 — Memory directory files (tool-call injection, lower trust):**

Files in `memory/*.md` are **not** loaded by `loadWorkspaceBootstrapFiles()`. They go through a separate pipeline: `listMemoryFiles()` (`src/memory/internal.ts:78-107`) and `resolveDefaultCollections()` (`src/memory/backend-config.ts:235-254`), accessed via `memory_search`/`memory_get` tool calls with a 4,000-character injection budget — not as system prompt context. The QMD backend validates `.md` extension and rejects symlinks (`src/memory/qmd-manager.ts:521-525`) but does **not** scan content.

**Neither path is scanned by the built-in skill scanner** (`src/security/skill-scanner.ts:38-47`), which only processes JS/TS files.

**Subagent mitigation:** `filterBootstrapFilesForSession()` at `src/agents/workspace.ts:499-507` limits subagents to only `AGENTS.md` + `TOOLS.md`, reducing the bootstrap attack surface from 9 files to 2 in multi-agent setups.

**Risk scenario:** An attacker with workspace write access (via compromised skill, plugin, shared git repo, or social engineering) plants persistent prompt injection in any of these files. The injection persists across sessions and appears as trusted system context, making it significantly harder for the model to reject than runtime injection from user messages or fetched content.

Cross-references: [Attack #27 (Persistent Memory Injection)](../05-worst-case-security/prompt-injection-attacks.md#-attack-27-persistent-memory-injection), [Threat model #7](../04-privacy-safety/threat-model.md#7-persistent-memory-files), [Post-merge hardening Gap #4](./post-merge-hardening.md#legitimate-gaps-status)

#### Security tool comparison matrix

| Capability | OpenClaw built-in | ClawHub / VirusTotal | Koi Scanner | Cisco skill-scanner |
|-----------|-------------------|---------------------|-------------|-------------------|
| **When it runs** | Skill install time | Publish + daily rescan | Manual CLI | Manual CLI |
| **JS/TS code scanning** | Yes (regex) | Yes (70+ AV engines) | Yes | Yes (regex + YARA) |
| **Python code scanning** | No | Yes (AV engines) | Partial | Yes (AST + YARA) |
| **SKILL.md scanning** | **No** | No | No | **Yes (LLM engine)** |
| **Bootstrap .md scanning** | No | No | No | Yes (LLM engine) |
| **Prompt injection detection** | No | No | No | **Yes (LLM engine)** |
| **SARIF CI/CD output** | No | No | Unknown | Yes |
| **Offline capable** | Yes | No (needs ClawHub API) | Yes | Partial (LLM/VT need API keys) |
| **Cost** | Free (bundled) | Free (ClawHub) | Free | Free (LLM engine needs API key) |

#### Practical usage for OpenClaw operators

**Install:**

```bash
pip install cisco-ai-skill-scanner
```

**Scan installed skills:**

```bash
# Scan all installed skills
skill-scanner scan ~/.openclaw/skills/

# Scan a specific skill
skill-scanner scan ~/.openclaw/skills/my-skill/

# Scan with SARIF output for CI/CD
skill-scanner scan ~/.openclaw/skills/ --format sarif -o results.sarif
```

**Scan workspace bootstrap files for prompt injection:**

```bash
# Point the Cisco scanner at your workspace directory containing .md files
# The LLM engine should analyze .md files by default when present in the scan directory
skill-scanner scan ~/your-workspace-dir/
```

> **Note:** Do not rely on `--include "*.md"` — this flag is unverified for the Cisco tool. Instead, point the scanner at the directory containing your workspace `.md` files directly. See [Beyond SKILL.md](#beyond-skillmd-all-persistent-md-files-are-unscanned) for the full list of files to scan.

**Recommended workflow — layered scanning:**

1. **At install time** (automatic): OpenClaw built-in scanner checks JS/TS
2. **After install** (manual): Run `skill-scanner scan` to check SKILL.md files for prompt injection
3. **At publish time** (automatic): ClawHub/VirusTotal scans uploaded skills
4. **Periodic** (manual/CI): Run full scans with all engines enabled

#### Limitations

| Limitation | Impact |
|-----------|--------|
| Requires Python 3.10+ | Additional runtime dependency for Node.js-based OpenClaw |
| LLM engine needs API keys (Cisco AI Defense or OpenAI) | Core prompt injection detection unavailable without paid API access |
| Behavioral analysis is Python-only (AST-based) | Most OpenClaw skills use markdown + JS/TS, not Python |
| No install-time integration | Cannot run automatically during `openclaw skills install` — must be invoked manually |
| No incremental scanning | Rescans everything on each run; no caching of previous results |

---

### Running total across all external audits

| Audit | Claims tested | Exploitable |
|-------|--------------|-------------|
| [Issue #1796 (Argus)](./issue-1796-argus-audit.md) | 8 | 0 |
| [Medium article (Saad Khalid)](./medium-article-audit.md) | 8 | 0 |
| [SecurityScorecard STRIKE + declawed.io](./securityscorecard-strike-report.md) | 5 | 0 |
| [ZeroLeeks AI Red Team](./zeroleeks-audit.md) | 34 | 0 |
| [Cisco AI Defense blog post](./cisco-ai-defense-skill-scanner.md) | 4 | 0 |
| **Total** | **59** | **0** |

**0/59 third-party vulnerability claims confirmed exploitable on the current codebase.**

The Cisco blog post also contributes a genuinely useful open-source tool (`skill-scanner`) that fills a detection gap in SKILL.md prompt injection scanning — a constructive outcome even though the risk claims overstate the threat.
