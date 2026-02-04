# Moltbot (formerly Clawdbot) Beginner's Guide

A comprehensive guide for understanding, installing, and securely deploying Clawdbot - the self-hosted AI assistant platform.

## What is Clawdbot?

**Clawdbot** is a self-hosted AI assistant that connects to your favorite messaging apps. Think of it as a bridge between powerful AI models (like Claude, GPT, Gemini) and your everyday chat platforms (WhatsApp, Telegram, Discord, Slack, Signal, iMessage, and more).

**Key Benefits:**
- **Privacy First** - Your data stays on your hardware, not in someone else's cloud
- **Universal Access** - Talk to AI from any messaging app you already use
- **Full Control** - Choose your AI provider, configure who can access it, run it anywhere

---

## Documentation Index

| Document | Description |
|----------|-------------|
| [01 - What is Clawdbot?](./01-what-is-clawdbot.md) | Plain English explanation for complete beginners |
| [02 - Architecture Overview](./02-architecture-overview.md) | Technical architecture with diagrams |
| [03 - Messaging Channels](./03-messaging-channels.md) | Supported chat platforms and how they connect |
| [04 - AI Providers](./04-ai-providers.md) | AI models and provider integrations |
| [05 - Installation Guide](./05-installation-guide.md) | Step-by-step installation instructions |
| [06 - Configuration](./06-configuration.md) | Configuration system explained |
| [07 - Security & Privacy](./07-security-privacy.md) | Security features and privacy controls |
| [08 - Mac Mini Deployment](./08-mac-mini-deployment.md) | Standalone Mac Mini setup guide |
| [09 - VPS Deployment](./09-vps-deployment.md) | Isolated VPS server deployment |
| [10 - Commands Reference](./10-commands-reference.md) | Essential CLI commands |
| [11 - Security Audit Analysis](./11-security-audit-analysis.md) | Code-verified analysis of security audit claims (Issue #1796 + Medium article) |

---

## Quick Start

```bash
# Install Clawdbot
npm install -g clawdbot@latest

# Run the interactive setup wizard (recommended)
clawdbot onboard --install-daemon

# Or manual setup:
clawdbot doctor     # Check system health
clawdbot login      # Authenticate AI providers
clawdbot gateway run --port 18789
```

---

## Requirements

- **Node.js 22+** (required)
- Package manager: npm, pnpm, or bun
- Operating System: macOS, Linux, or Windows

---

## Official Resources

- **Documentation**: https://docs.clawd.bot
- **GitHub Repository**: https://github.com/clawdbot/clawdbot
- **Default Port**: 18789
- **Config File**: `~/.clawdbot/clawdbot.json`

---

## Use Cases Covered

This guide focuses on two privacy-conscious deployment scenarios:

1. **Mac Mini Standalone** - Personal AI assistant on dedicated home hardware with local-only access and optional Ollama for fully local AI processing

2. **Isolated VPS** - Cloud server deployment with strict isolation, firewall rules, and Tailscale for secure remote access

---

## Security Audit Analysis (Issue [#1796](https://github.com/clawdbot/clawdbot/issues/1796))

In January 2026, the Argus Security Platform (v1.0.15) ran an automated 6-phase scan combining Semgrep, Trivy, Gitleaks, TruffleHog, and Claude Sonnet 4.5 AI analysis against the Clawdbot repository. The report claimed **512 total findings including 8 CRITICAL**. This section provides a code-verified analysis of those claims.

### Critical Claims Assessment

All 8 CRITICAL findings were manually verified against the source code:

| # | Claim | Verdict | Explanation |
|---|-------|---------|-------------|
| 1 | Plaintext OAuth token storage | **True, by design** | Tokens stored as JSON files with `0o600` permissions, set on every write (`src/infra/json-file.ts:20`). Standard practice for CLI tools (cf. `gh`, `gcloud`). No keychain integration, but filesystem permissions enforced. |
| 2 | Missing CSRF in OAuth state validation | **False** | The `?? expectedState` is a parsing fallback, not a bypass. Actual CSRF validation performs strict `state !== verifier` comparison before token exchange (`extensions/google-gemini-cli-auth/oauth.ts:618-619`). |
| 3 | Hardcoded OAuth client secret | **True, standard practice** | Per [RFC 8252 Sections 8.4-8.5](https://datatracker.ietf.org/doc/html/rfc8252#section-8.4), desktop/CLI apps are "public clients" that cannot maintain secret confidentiality. Google's own CLI tools follow the same pattern. Base64 encoding is cosmetic only. |
| 4 | Token refresh race condition | **False** | Uses `proper-lockfile` with exponential backoff (config: `src/agents/auth-profiles/constants.ts:12-21`). Lock held throughout the entire refresh-and-save cycle (`src/agents/auth-profiles/oauth.ts:43-105`). Errors propagate to callers. |
| 5 | Insufficient file permission checks | **True, by design** | Permissions set to `0o600` on every write (secure default). Audit/fix tooling exists via `clawdbot security audit` and `clawdbot security fix`. No pre-load validation, but files stay correct unless manually changed externally. |
| 6 | Path traversal in agent directories | **False** | Paths go through `resolveUserPath()` (`src/agents/agent-paths.ts:10,12`) which calls `path.resolve()` (`src/utils.ts:209`), normalizing traversal. Agent IDs come from environment/config, not user input. |
| 7 | Webhook signature bypass | **True, properly gated** | `skipVerification` parameter exists in `extensions/voice-call/src/webhook-security.ts` but requires explicit parameter passing. Dev-only flag, not enabled by default, no evidence of production exposure. |
| 8 | Insufficient token expiry validation | **False** | Every token use path checks `Date.now() < cred.expires` before returning credentials. On refresh failure, re-reads the store and re-checks expiry. No stale token fallback (`src/agents/auth-profiles/oauth.ts:176-197`). |

**Result: 0 of 8 CRITICAL claims are actual security vulnerabilities.**

- 3 are true observations about intentional design decisions (not vulnerabilities)
- 1 is true but properly gated behind a dev-only flag (low risk)
- 4 are factually incorrect (the code already handles these correctly)

### Bulk Scanner Context

The remaining 504 findings break down as:

| Scanner | Count | Assessment |
|---------|-------|------------|
| Gitleaks | 255 | "Generic API key" pattern matches on test fixtures, UUIDs, and base64 strings. Overwhelmingly false positives. |
| Semgrep | 190 | Flagged `ws://` localhost connections (safe for local gateway), CHANGELOG text matches, and standard patterns without context. |
| Trivy | 20 | Dependency CVEs. Valid to track as maintenance items but standard for any Node.js project, not code vulnerabilities. |
| TruffleHog | 8 | Unverified secret patterns. No confirmed credential leaks found. |

Automated scanners without codebase context produce high false-positive rates. The 512-finding headline reflects scanner noise, not 512 security problems.

### Maintainer Response

The maintainer ([steipete](https://github.com/steipete)) reviewed and [confirmed](https://github.com/clawdbot/clawdbot/issues/1796):

> Some items are accurate but by design (public OAuth client secret; plaintext credential stores with 0600 perms). Other items are incorrect or overstated (OAuth state; token-refresh lock "race"). Webhook signatures are verified by default and only bypassed via an explicit dev-only config flag.

### Existing Security Controls

For the actual security architecture, access controls, credential handling, and privacy model, see [07 - Security & Privacy](./07-security-privacy.md). Key controls include allowlist-based access, local-only data storage, `0o600` file permissions, and configurable security audit/fix tooling.

### Recommended Hardening

For production deployments, implement defense in depth:
- Enable Docker sandbox with `network: none` for execution isolation
- Wrap untrusted content in `<untrusted>` tags with a system prompt rule to ignore instructions inside
- Block dangerous command patterns (`rm -rf`, `curl | bash`, `git push --force`)
- Protect shell history: `export HISTCONTROL=ignoreboth`
- Set a gateway auth token and bind to localhost only

See [11 - Security Audit Analysis](./11-security-audit-analysis.md#recommended-hardening-measures) and [VibeProof Security Guide](https://vibeproof.dev/blog/moltbot-security-setup-guide) for details.

---

## Second Security Audit (Medium Article)

In January 2026, a Medium article by Saad Khalid titled *"Why Clawdbot is a Bad Idea: Critical Zero-days Found in My Audit"* claimed **8 critical zero-day vulnerabilities** (CVSS 7.5-10.0) based on a self-described "Complete White Box Penetration Test." All 8 claims were verified against the source code.

| # | Claim | Verdict | Explanation |
|---|-------|---------|-------------|
| 1 | Config injection RCE via `setupCommand` | **Partially true, overstated** | `setupCommand` executes inside Docker container (not host) (`src/agents/sandbox/docker.ts:242-243`). Config changes require gateway auth. Container has `no-new-privileges`. Real risk: Medium. |
| 2 | Arbitrary write via `nodes:screen_record` outPath | **True but overstated** | `outPath` lacks validation (`src/agents/tools/nodes-tool.ts:344-347`), but writes to paired node device, not gateway host. Requires node pairing approval. Real risk: Low-Medium. |
| 3 | Log traversal via `logs.tail` | **False** | `LogsTailParamsSchema` has `additionalProperties: false` with only `cursor`, `limit`, `maxBytes`. File path from `getResolvedLoggerSettings().file` (config), not user input. |
| 4 | DNS rebinding SSRF via web-fetch | **False** | `resolvePinnedHostname()` + `createPinnedDispatcher()` (`src/infra/net/ssrf.ts:209-247`) pin DNS resolution. Redirect-to-private-IP explicitly tested and blocked (`web-fetch.ssrf.test.ts:120-142`). |
| 5 | Self-approving agent (no RBAC) | **False** | `authorizeGatewayMethod()` (`src/gateway/server-methods.ts:93-160`) enforces role checks. Agents connect as `role: "node"`, blocked from all non-node methods. Approval requires `operator.approvals` scope. |
| 6 | Token field shifting via pipe injection | **Misleading** | Pipe-delimited format (`src/gateway/device-auth.ts:13-31`) lacks input sanitization (true), but tokens are RSA-signed. Modified payload fails signature verification. |
| 7 | Shell injection via incomplete regex | **False** | `isSafeExecutableValue()` (`src/infra/exec-safety.ts:16-44`) validates executable *names* (not commands). `BARE_NAME_PATTERN = /^[A-Za-z0-9._+-]+$/` is strict. Article conflates config validation with shell injection. |
| 8 | Environment variable injection (LD_PRELOAD) | **Partially true, MITIGATED in PR #12** | Gateway validates `params.env` via blocklist (`src/agents/bash-tools.exec.ts:61-78,971-973`). Node-host has blocklist (`src/node-host/runner.ts:165-174`). Requires human approval + localhost + no sandbox. |

**Result: 0 of 8 claims are exploitable as described.**

- 5 are factually incorrect (claims 3, 4, 5, 6, 7)
- 2 are partially true but heavily overstated (claims 1, 8)
- 1 is a true observation with misleading risk framing (claim 2)

### Legitimate Gaps Noted

Three defense-in-depth items were identified (not exploitable as described, but worth hardening):

1. ~~**Gateway-side env var blocklist:**~~ **CLOSED in PR #12.** Gateway now validates env vars via `DANGEROUS_HOST_ENV_VARS` blocklist and `validateHostEnv()` (`src/agents/bash-tools.exec.ts:59-107`).
2. **Pipe-delimited token format:** RSA signing prevents exploitation, but a structured format (JSON) would be more robust.
3. **outPath validation:** `screen_record` outPath accepts arbitrary paths. Writes are confined to the paired node device, but path validation would add depth.

### Post-Merge Hardening (PR #1)

Three upstream commits strengthened controls analyzed above: Docker PATH injection fix (`771f23d` — Claim 1), per-sender group tool policies (`3b0c80c` — Claim 5), and webhook timing-safe comparison (`3b8792e` — Audit 1 Claim 7). Additional hardening: `fs-safe.ts` file serving (`5eee991`) and browser evaluate gating (`78f0bc3`).

### Post-Merge Hardening (PR #2)

Five security-relevant changes: transient network error handling prevents gateway crashes (`3b879fe`), per-account session isolation prevents cross-account leakage (`d499b14`), Discord username resolution gating (`7958ead`), Telegram session fragmentation fix (`9154971`), and formal TLA+ security models (`3bf768a`).

All three legitimate gaps remain open (gateway env blocklist, pipe-delimited token format, outPath validation).

### Post-Merge Hardening (PR #3)

One security-relevant commit: XML attribute injection prevention in media text attachments (`b71772427` — #3700): escapes `<`, `>`, `"`, `'`, `&` in file names and MIME types, adds UTF-16/BOM detection, MIME override logging for auditability.

All three legitimate gaps remain open.

### Post-Merge Hardening (PR #5)

One security-relevant commit: Telegram skill command scoping (`c6ddc95fc` — #4360): scopes skill commands to bound agent per bot, preventing cross-agent command registration (thanks @robhparker).

All three legitimate gaps remain open.

### Post-Merge Hardening (PR #6)

One security-relevant commit: Gateway token undefined fix (`201d7fa95` — #4873): prevents `String(undefined)` from producing the literal `"undefined"` string as gateway token, ensuring fallback to random token generation (thanks @Hisleren).

Additionally, `SECURITY.md` was updated (`2cdfecdde`) to clarify: no bug bounty program, and public internet exposure is out of scope.

All three legitimate gaps remain open.

### Post-Merge Hardening (PR #7)

One security-relevant commit:
- **`c67df653b`** — Restricts local path extraction in media parser to prevent LFI (#4880): hardens `src/media/parse.ts` against local file inclusion attacks, adds test coverage in `src/media/parse.test.ts`

This is new security hardening unrelated to existing audit claims. **All three legitimate gaps remain open.**

### Post-Merge Hardening (PR #9 — 50 commits)

Two critical security fixes:

- **`1295b6705`** — GHSA-4mhr-g7xj-cg8j: Block arbitrary exec via lobsterPath/cwd (#5335): Lobster extension now validates `lobsterPath` against plugin config, blocking arbitrary command execution via tool-provided paths.

- **`34e2425b4`** — LFI prevention: Restrict MEDIA path extraction (#4930): `src/auto-reply/reply/stage-sandbox-media.ts` restricts inbound media staging to media directory only, preventing path traversal attacks.

Additional hardening:
- **`7a6c40872`** — System prompt safety guardrails (#5445)
- **`baf9505bf`** — Formal models conformance check (CI)

**All three legitimate gaps remain open** (gateway env blocklist, pipe-delimited token format, outPath validation).

### Post-Merge Hardening (PR #11 — 21 commits)

One security-relevant commit:

- **`a1e89afcc`** — Secure Chrome extension relay CDP: Adds token-based authentication (`x-openclaw-relay-token` header) and loopback address validation (`src/browser/extension-relay.ts:79,104-134,177-178`) to the Chrome DevTools Protocol relay. Prevents unauthorized CDP access from non-localhost sources.

This is new security hardening unrelated to existing audit claims. **All three legitimate gaps remain open** (gateway env blocklist, pipe-delimited token format, outPath validation).

### Post-Merge Hardening (PR #12 — 64 commits)

**Critical: Gateway env var blocklist gap closed.**

Seven security-relevant commits:

- **`0a5821a81`** + **`a87a07ec8`** — Strict environment variable validation (#4896) (thanks @HassanFleyah): `DANGEROUS_HOST_ENV_VARS` blocklist and `validateHostEnv()` now block `LD_PRELOAD`, `DYLD_*`, `NODE_OPTIONS`, `PATH`, etc. on gateway host execution (`src/agents/bash-tools.exec.ts:59-107,971-973`). **Closes Legitimate Gap #1.**

- **`b796f6ec0`** — Web tools and file parsing hardening (#4058) (thanks @VACInc)
- **`a2b00495c`** — TLS 1.3 minimum requirement (thanks @loganaden)
- **`1bdd9e313`** — WhatsApp accountId path traversal prevention (#4610)
- **`9b6fffd00`** — Message tool sandbox path validation (#6398)
- **`7aeabbabd`** — OAuth provider guard refinement

**Gap status: 1 closed, 2 remain open** (pipe-delimited token format, outPath validation).

### Post-Merge Hardening (PR #13)

Two security-relevant commits:

- **`4e4ed2ea1`** — Slack media security (#6639): Caps media download sizes and validates Slack file URLs to prevent DoS and path traversal attacks.

- **`d46b489e2`** — Telegram download timeout (CWE-400): Adds timeout to Telegram file downloads to prevent resource exhaustion from slow/hanging connections.

- **`01449a2f4`** — Telegram download timeouts (#6914) (thanks @hclsys).

**Gap status: 1 closed, 2 remain open** (pipe-delimited token format, outPath validation).

### Post-Merge Hardening (Feb 3 sync 2)

Four security-relevant commits:

- **`d1ecb4607`** — Harden exec allowlist parsing: Rejects `$()` command substitution and backticks inside double-quoted strings (`src/infra/exec-approvals.ts:664,719`). Addresses Audit 2 "shell injection regex" claim.

- **`fe81b1d71`** — Require shared auth before device bypass: Validates shared secret auth before allowing Tailscale device bypass (`src/gateway/server/ws-connection/message-handler.ts:398-458`).

- **`fff59da96`** — Slack fail closed on channel type lookup: Fails closed when channel type lookup fails, infers type from ID prefix (`src/slack/monitor/slash.ts:181-182`).

- **`578bde1e0`** — Security: healthcheck skill (#7641): Bootstrap audit guidance tooling (`skills/healthcheck/SKILL.md`) (thanks @Takhoffman).

- **`cfd6b21d0`** — Repair malformed tool calls (#7473): Session transcript integrity fix (thanks @justinhuangcode).

**Gap status: 1 closed, 2 remain open** (pipe-delimited token format, outPath validation).

### Post-Merge Hardening (Feb 3 sync 3)

Three security-relevant commits:

- **`afbb1af6c`** — Restore safety + session_status hints: Re-adds safety guidelines to agent system prompts (`src/agents/system-prompt.ts`). Regression fix for accidentally removed safety section.

- **`c248da031`** — Memory: harden QMD memory_get path checks: Validates `.md` extension and rejects symlinks (`src/memory/qmd-manager.ts`). Mitigates path traversal attacks.

- **`1861e7636`** — Memory: clamp QMD citations to injected budget: Budget clamping for memory citations (`src/agents/tools/memory-tool.ts`). Defense-in-depth against prompt injection.

**Gap status: 1 closed, 2 remain open** (pipe-delimited token format, outPath validation).

### Post-Merge Hardening (Feb 4 sync 1)

Three security-relevant commits:

- **`a7f4a53ce`** — Harden Windows exec allowlist: Blocks cmd.exe bypass via `&` metacharacter. New `WINDOWS_UNSUPPORTED_TOKENS` set rejects `& | < > ^ ( ) % !` in Windows shell commands. Prevents allowlist circumvention on Windows platforms (`src/infra/exec-approvals.ts`, `src/node-host/runner.ts`). Thanks @simecek.

- **`8f3bfbd1c`** — Matrix allowlist hardening: Requires full MXIDs (`@user:server`) for Matrix allowlists. Display name resolution only accepts single exact matches from directory search. Closes ambiguous name resolution vulnerability (`extensions/matrix/src/matrix/monitor/allowlist.ts`). Thanks @MegaManSec.

- **`f8dfd034f`** — Voice-call inbound policy hardening: Requires exact phone number matching (no suffix), rejects anonymous callers, requires Telnyx `publicKey` for allowlist/pairing, token-gates Twilio media streams, caps webhook body to 1MB (`extensions/voice-call/src/`). Thanks @simecek.

**Gap status: 1 closed, 2 remain open** (pipe-delimited token format, outPath validation).

### Post-Merge Hardening (Feb 4 sync 2)

Two security-relevant commits:

- **`66d8117d4`** — Control UI origin hardening: New `checkBrowserOrigin()` (`src/gateway/origin-check.ts:57-85`) validates WebSocket Origin headers for Control UI and Webchat connections. Accepts only: configured `allowedOrigins`, same-host requests, or loopback addresses. Prevents clickjacking and cross-origin WebSocket hijacking. New config: `gateway.controlUi.allowedOrigins`.

- **`efe2a464a`** — Approval scope gating (#1) (thanks @mitsuhiko): `/approve` command now requires `operator.approvals` or `operator.admin` scope for gateway clients (`src/auto-reply/reply/commands-approve.ts:89-101`). Defense-in-depth layer atop existing `authorizeGatewayMethod()` RBAC (`src/gateway/server-methods.ts:93`). Strengthens protection against Audit 2 Claim 5 (agent self-approval).

**Gap status: 1 closed, 2 remain open** (pipe-delimited token format, outPath validation).

For the full detailed analysis with code references, see [11 - Security Audit Analysis](./11-security-audit-analysis.md#second-security-audit-medium-article-january-2026).

---

## Navigation

Start with [01 - What is Clawdbot?](./01-what-is-clawdbot.md) if you're completely new, or jump to [05 - Installation Guide](./05-installation-guide.md) if you want to get started immediately.

---

*Generated with Claude Opus 4.5 for the Clawdbot codebase*
