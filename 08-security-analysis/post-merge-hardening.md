> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Open PRs](./open-upstream-prs.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Model Comparison](./ai-model-analysis-comparison.md)

### Table of Contents

- [Legitimate Gaps Status](#legitimate-gaps-status)
- [PR #1 (129 upstream commits)](#post-merge-hardening-pr-1-129-upstream-commits)
- [PR #2 (40 upstream commits)](#post-merge-hardening-pr-2-40-upstream-commits)
- [PR #3 (4 commits)](#post-merge-hardening-pr-3--4-commits)
- [PR #5 (25 commits)](#post-merge-hardening-pr-5--25-commits)
- [PR #6 (34 commits)](#post-merge-hardening-pr-6--34-commits)
- [PR #7](#post-merge-hardening-pr-7)
- [PR #9 (50 commits)](#post-merge-hardening-pr-9--50-commits)
- [PR #11 (21 commits)](#post-merge-hardening-pr-11--21-commits)
- [PR #12 (64 commits)](#post-merge-hardening-pr-12--64-commits)
- [PR #13](#post-merge-hardening-pr-13)
- [Feb 2 sync 4](#post-merge-hardening-feb-2-sync-4)
- [Feb 2 sync 10](#post-merge-hardening-feb-2-sync-10)
- [Feb 3 sync 2](#post-merge-hardening-feb-3-sync-2)
- [Feb 3 sync 3](#post-merge-hardening-feb-3-sync-3)
- [Feb 4 sync 1](#post-merge-hardening-feb-4-sync-1)
- [Feb 4 sync 2](#post-merge-hardening-feb-4-sync-2)
- [Feb 4 sync 3](#post-merge-hardening-feb-4-sync-3)
- [Feb 5 sync 1 (Notes)](#post-merge-notes-feb-5-sync-1)
- [Feb 5 sync 2](#post-merge-hardening-feb-5-sync-2)
- [Feb 5 sync 3](#post-merge-hardening-feb-5-sync-3)
- [Feb 6 sync 2](#post-merge-hardening-feb-6-sync-2)
- [Feb 6 sync 3](#post-merge-hardening-feb-6-sync-3)
- [Feb 6 sync 4](#post-merge-hardening-feb-6-sync-4)
- [Feb 7 sync 1](#post-merge-hardening-feb-7-sync-1)
- [Feb 7 sync 2](#post-merge-hardening-feb-7-sync-2)
- [Feb 7 sync 3](#post-merge-hardening-feb-7-sync-3)
- [Feb 9 sync 1](#post-merge-hardening-feb-9-sync-1)
- [Feb 9 sync 3](#post-merge-hardening-feb-9-sync-3)
- [Feb 10 sync 3 (26 commits)](#post-merge-hardening-feb-10-sync-3-26-upstream-commits)
- [Feb 10 sync 5 (51 commits)](#post-merge-hardening-feb-10-sync-5-51-upstream-commits)
- [Feb 10 sync 7 (6 commits)](#post-merge-hardening-feb-10-sync-7-6-upstream-commits)
- [Feb 11 sync 1 (22 commits)](#post-merge-notes-feb-11-sync-1-22-upstream-commits)
- [Feb 11 sync 2 (17 commits)](#post-merge-hardening-feb-11-sync-2-17-upstream-commits)
- [Feb 12 sync 1 (32 commits)](#post-merge-hardening-feb-12-sync-1-32-upstream-commits)
- [Feb 12 sync 2 (Notes)](#post-merge-notes-feb-12-sync-2-6-upstream-commits)
- [Feb 12 sync 3 (8 commits)](#post-merge-hardening-feb-12-sync-3-8-upstream-commits)

## Post-Merge Security Hardening

> This section tracks security-relevant commits merged from upstream. Entries are added by the sync-explain-docs-with-upstream skill.

### Legitimate Gaps Status

Four defense-in-depth items were identified across audits:

1. ~~**Gateway-side env var blocklist:**~~ **CLOSED in PR #12.** Gateway now validates env vars via `DANGEROUS_HOST_ENV_VARS` blocklist and `validateHostEnv()` (`src/agents/bash-tools.exec.ts:59-107,976-977`).
2. **Pipe-delimited token format:** RSA signing prevents exploitation, but a structured format (JSON) would be more robust against future changes.
3. **outPath validation in screen_record:** Accepts arbitrary paths without validation. Writes are confined to the paired node device, but path validation would add depth.
4. **Bootstrap/memory `.md` content scanning:** The built-in scanner (`src/security/skill-scanner.ts:37-46`) only scans JS/TS. Nine workspace bootstrap files are injected into the system prompt (20,000 chars each) via `loadWorkspaceBootstrapFiles()` (`src/agents/workspace.ts:239-293`) with no content validation. `memory/*.md` files are accessed via tool calls (4,000-char budget) through a separate pipeline (`src/memory/internal.ts:78-107`) also without content scanning. QMD memory path hardening validates `.md` extension and rejects symlinks (`src/memory/qmd-manager.ts:346-352`) but does not scan content. Subagent exposure is limited — `filterBootstrapFilesForSession()` (`src/agents/workspace.ts:295-305`) restricts subagents to `AGENTS.md` + `TOOLS.md` only. See [Cisco AI Defense gap analysis](./cisco-ai-defense-skill-scanner.md#beyond-skillmd-all-persistent-md-files-are-unscanned).

**Gap status: 1 closed, 3 remain open.**

### Post-merge hardening (PR #1, 129 upstream commits)

Three commits directly strengthened controls referenced by both audits:

- **Docker PATH injection fix** (`771f23d`): `setupCommand` PATH handling moved from shell string interpolation to a container env var, closing a command injection vector inside the sandbox (Audit 2 Claim 1). Also addresses [CVE-2026-24763](./official-security-advisories.md#cve-2026-24763-docker-path-command-injection).
- **Per-sender tool policies** (`3b0c80c`): RBAC now extends to per-user tool policies in group chats, deepening the access control that already prevented agent self-approval (Audit 2 Claim 5).
- **Webhook timing-safe comparison** (`3b8792e`): LINE webhook signature validation switched from `===` to `crypto.timingSafeEqual()`, eliminating a theoretical timing side-channel (Audit 1 Claim 7).

Additional security improvements: hardened file serving via `O_NOFOLLOW` + inode verification (`5eee991`), and browser JS execution gated behind `evaluateEnabled` config flag (`78f0bc3`).

### Post-merge hardening (PR #2, 40 upstream commits)

Five security-relevant changes were introduced:

- **Transient network error handling** (`3b879fe`, `3a25a4f`, `0770194`): New `TRANSIENT_NETWORK_CODES` set (`src/infra/unhandled-rejections.ts:20-37`) prevents gateway crashes on network instability. Non-fatal errors like `ECONNRESET`, `ETIMEDOUT`, and undici timeouts are logged and suppressed.

- **Per-account session isolation** (`d499b14`): New `"per-account-channel-peer"` DM scope (`src/routing/session-key.ts:148,166-170`) isolates sessions per account, channel, and peer, preventing cross-account session leakage in multi-account channel setups.

- **Discord username resolution gating** (`7958ead`, `b01612c`): Username-to-user-ID lookups for outbound DMs are now gated through the directory config (`src/discord/targets.ts:77`), preventing unauthorized directory queries.

- **Telegram session fragmentation fix** (`9154971`): `resolveTelegramForumThreadId()` (`src/telegram/bot/helpers.ts:18-31`) now ignores `message_thread_id` for non-forum groups. Reply threads in regular groups no longer create separate sessions.

- **Formal security models** (`3bf768a`): New TLA+ machine-checked models document security invariants for pairing, ingress gating, and routing/session-key isolation (`docs/security/formal-verification.md`).

### Post-Merge Hardening (PR #3 — 4 commits)

One security-relevant commit:

- **`b71772427`** — XML attribute injection prevention in media text attachments (#3700): escapes special characters (`<`, `>`, `"`, `'`, `&`) in file names and MIME types, adds UTF-16/BOM detection, MIME override logging for auditability

### Post-Merge Hardening (PR #5 — 25 commits)

One security-relevant commit:

- **`c6ddc95fc`** — Telegram skill command scoping (#4360): scopes skill commands to bound agent per bot, preventing cross-agent command registration (thanks @robhparker)

### Post-Merge Hardening (PR #6 — 34 commits)

One security-relevant commit:

- **`201d7fa95`** — Gateway token undefined fix (#4873): prevents `String(undefined)` from producing the literal `"undefined"` string as a gateway token. Ensures empty/undefined input falls through to `randomToken()` (thanks @Hisleren)

Additionally, `SECURITY.md` was updated (`2cdfecdde`) to clarify: no bug bounty program, and public internet exposure is out of scope—reinforcing the existing threat model.

### Post-Merge Hardening (PR #7)

One security-relevant commit:

- **`c67df653b`** — Restricts local path extraction in media parser to prevent LFI (#4880): hardens `src/media/parse.ts` against local file inclusion attacks via path extraction, adds test coverage in `src/media/parse.test.ts`

### Post-Merge Hardening (PR #9 — 50 commits)

Two critical security fixes:

- **`1295b6705`** — GHSA-4mhr-g7xj-cg8j: Block arbitrary exec via lobsterPath/cwd (#5335): The Lobster extension now validates and restricts `lobsterPath` to plugin config, blocking tool-provided paths that could enable arbitrary command execution. Comprehensive tests added.

- **`34e2425b4`** — LFI prevention: Restrict MEDIA path extraction (#4930): The `src/auto-reply/reply/stage-sandbox-media.ts` now restricts inbound media staging to the media directory only, preventing local file inclusion attacks via path traversal.

Additional security hardening:

- **`7a6c40872`** — System prompt safety guardrails (#5445): Adds runtime guardrails to agent system prompts.
- **`baf9505bf`** — Formal models conformance check (CI): Adds informational TLA+ conformance verification to CI.

### Post-Merge Hardening (PR #11 — 21 commits)

One security-relevant commit:

- **`a1e89afcc`** — Secure Chrome extension relay CDP: Adds token-based authentication (`x-openclaw-relay-token` header) and loopback address validation (`src/browser/extension-relay.ts:80,105-134,181-182`) to the Chrome DevTools Protocol relay. Prevents unauthorized CDP access from non-localhost sources.

### Post-Merge Hardening (PR #12 — 64 commits)

**Critical: Gateway env var blocklist gap closed.**

Seven security-relevant commits:

- **`0a5821a81`** + **`a87a07ec8`** — Strict environment variable validation (#4896) (thanks @HassanFleyah): `DANGEROUS_HOST_ENV_VARS` blocklist and `validateHostEnv()` now block `LD_PRELOAD`, `DYLD_*`, `NODE_OPTIONS`, `PATH`, etc. on gateway host execution (`src/agents/bash-tools.exec.ts:59-107,976-977`). **Closes Legitimate Gap #1.**

- **`b796f6ec0`** — Web tools and file parsing hardening (#4058) (thanks @VACInc)
- **`a2b00495c`** — TLS 1.3 minimum requirement (thanks @loganaden)
- **`1bdd9e313`** — WhatsApp accountId path traversal prevention (#4610)
- **`9b6fffd00`** — Message tool sandbox path validation (#6398)
- **`7aeabbabd`** — OAuth provider guard refinement

### Post-Merge Hardening (PR #13)

Two security-relevant commits:

- **`4e4ed2ea1`** — Slack media security (#6639): Caps media download sizes and validates Slack file URLs to prevent DoS and path traversal attacks in the Slack channel adapter.

- **`d46b489e2`** — Telegram download timeout (CWE-400): Adds timeout to Telegram file downloads to prevent resource exhaustion from slow/hanging connections. Defense-in-depth against denial-of-service via malicious media attachments.

Additional commits:

- **`01449a2f4`** — Telegram download timeouts (#6914): Complementary timeout handling for Telegram downloads (thanks @hclsys).

### Post-Merge Hardening (Feb 2 sync 4)

One security-relevant commit:

- **`d03eca845`** — Harden plugin and hook install paths: Adds path traversal detection to plugin and hook installation. `validateHookId()` + `resolveSafeInstallDir()` (`src/hooks/install.ts:55-97`) and `validatePluginId()` + `resolveSafeInstallDir()` (`src/plugins/install.ts:59-115`) now reject hook/plugin names containing `..`, `/`, `\`, or reserved segments. Prevents directory traversal attacks during extension installation.

### Post-Merge Hardening (Feb 2 sync 10)

Three security-relevant commits:

- **`81c68f582`** — Guard remote media fetches with SSRF checks: New `fetch-guard.ts` centralized SSRF protection for all remote media fetches (`src/infra/net/fetch-guard.ts:1-170`). Media downloads now validate against private IP ranges before fetching.

- **`9bd64c8a1`** — Expand SSRF guard coverage: Extended SSRF protection to media understanding providers (Deepgram, Google, OpenAI audio/video transcription) and skills installation. Shared utilities in `src/media-understanding/providers/shared.ts`.

- **`57d008a33`** — Harden global updates: New `update-global.ts` validates update sources before executing global npm installs (`src/infra/update-global.ts`).

### Post-Merge Hardening (Feb 3 sync 2)

Four security-relevant commits:

- **`d1ecb4607`** — Harden exec allowlist parsing: Rejects `$()` command substitution and backticks inside double-quoted strings in allowlist pattern matching (`src/infra/exec-approvals.ts:696,699`). Addresses Audit 2 "shell injection regex" claim by preventing shell expansion within quoted arguments.

- **`fe81b1d71`** — Require shared auth before device bypass: Gateway now validates shared secret (token/password) authentication before allowing Tailscale device bypass (`src/gateway/server/ws-connection/message-handler.ts:398-458`). Prevents auth bypass when only Tailscale identity is available.

- **`fff59da96`** — Slack fail closed on slash command channel type lookup: Slash command handler now fails closed when Slack API channel type lookup fails (`src/slack/monitor/slash.ts:181-182`). Infers channel type from ID prefix (D*/C*/G*) as fallback. Addresses potential authorization bypass in Slack slash commands.

- **`578bde1e0`** — Security: healthcheck skill (#7641): Adds bootstrap audit guidance tooling for deployment validation against security claims (`skills/healthcheck/SKILL.md`). New skill for systematic security healthchecks (thanks @Takhoffman).

Additional integrity fix:

- **`cfd6b21d0`** — Repair malformed tool calls and session transcripts (#7473): Session file repair utilities to recover from corrupted tool call structures (`src/agents/session-file-repair.ts`, `src/agents/session-transcript-repair.ts`). Defense-in-depth for transcript integrity (thanks @justinhuangcode).

### Post-Merge Hardening (Feb 3 sync 3)

Three security-relevant commits:

- **`afbb1af6c`** — Restore safety + session_status hints: Re-adds safety guidelines to agent system prompts (`src/agents/system-prompt.ts`), including self-preservation prevention, compliance with stop/pause/audit requests, and safeguard manipulation prevention. Regression fix for accidentally removed safety section.

- **`c248da031`** — Memory: harden QMD memory_get path checks: Validates accessed files end with `.md` extension and checks file type via `fs.lstat()` to reject symbolic links and non-regular files (`src/memory/qmd-manager.ts`). Mitigates path traversal and symlink attacks in QMD memory backend.

- **`1861e7636`** — Memory: clamp QMD citations to injected budget: Implements budget clamping for memory citation snippets respecting `maxInjectedChars` limit (`src/agents/tools/memory-tool.ts`). Defense-in-depth against prompt injection via unbounded memory content.

### Post-Merge Hardening (Feb 4 sync 1)

Three security-relevant commits:

- **`a7f4a53ce`** — Harden Windows exec allowlist: Blocks cmd.exe bypass via `&` metacharacter. New `WINDOWS_UNSUPPORTED_TOKENS` set rejects `& | < > ^ ( ) % !` in Windows shell commands. Prevents allowlist circumvention on Windows platforms (`src/infra/exec-approvals.ts`, `src/node-host/runner.ts`). Thanks @simecek.

- **`8f3bfbd1c`** — Matrix allowlist hardening: Requires full MXIDs (`@user:server`) for Matrix allowlists. Display name resolution only accepts single exact matches from directory search. Closes ambiguous name resolution vulnerability (`extensions/matrix/src/matrix/monitor/allowlist.ts`). Thanks @MegaManSec.

- **`f8dfd034f`** — Voice-call inbound policy hardening: Requires exact phone number matching (no suffix), rejects anonymous callers, requires Telnyx `publicKey` for allowlist/pairing, token-gates Twilio media streams, caps webhook body to 1MB (`extensions/voice-call/src/`). Thanks @simecek.

### Post-Merge Hardening (Feb 4 sync 2)

Two security-relevant commits:

- **`66d8117d4`** — Control UI origin hardening: New `checkBrowserOrigin()` (`src/gateway/origin-check.ts:43-71`) validates WebSocket Origin headers for Control UI and Webchat connections. Accepts only: configured `allowedOrigins`, same-host requests, or loopback addresses. Prevents clickjacking and cross-origin WebSocket hijacking. New config: `gateway.controlUi.allowedOrigins`.

- **`efe2a464a`** — Approval scope gating (#1) (thanks @mitsuhiko): `/approve` command now requires `operator.approvals` or `operator.admin` scope for gateway clients (`src/auto-reply/reply/commands-approve.ts:89-101`). Defense-in-depth layer atop existing `authorizeGatewayMethod()` RBAC (`src/gateway/server-methods.ts:93`). Strengthens protection against Audit 2 Claim 5 (agent self-approval).

### Post-Merge Hardening (Feb 4 sync 3)

Three security-relevant commits:

- **`35eb40a70`** — fix(security): separate untrusted channel metadata from system prompt (#7872) (thanks @KonstantinMirin): New `src/security/channel-metadata.ts` isolates untrusted Discord/Slack channel topics from system prompt injection. Channel topic/purpose metadata moved from `GroupSystemPrompt` (instruction-bearing) to `UntrustedContext` (display-only with security warnings). Prevents channel admins from injecting instructions via topic/purpose fields.

- **`a749db982`** — fix: harden voice-call webhook verification: Significantly enhanced webhook verification in `extensions/voice-call/src/webhook-security.ts` (+270 lines). Added provider-specific validation for Twilio and Plivo with comprehensive test coverage (134 new tests). Reinforces Audit 1 Claim 7 (webhook signature bypass) controls.

- **`6fdb13668`** — docs: document secure DM mode preset (#7872): Formalizes `session.dmScope: "per-channel-peer"` configuration for DM context isolation. Documents mitigation for cross-user context leakage in multi-sender DMs.

### Post-Merge Notes (Feb 5 sync 1)

**New feature: Cloudflare AI Gateway provider** (`5b0851ebd`)

Commit `5b0851ebd` adds Cloudflare AI Gateway as a new provider option. This is not a security fix but provides security-adjacent benefits:

- **Cloudflare AI Gateway** is a forward proxy between OpenClaw and AI inference providers (Anthropic, OpenAI, etc.)
- Routes requests through: `https://gateway.ai.cloudflare.com/v1/<account_id>/<gateway_id>/anthropic`
- Provides: analytics, logging, caching, rate limiting, request retry/fallback
- Security benefits: Guardrails (content moderation), DLP (Data Loss Prevention), request auditing
- Configuration via `openclaw onboard --auth-choice cloudflare-ai-gateway-api-key`

Implementation details (verified via LSP/code review):
- **Provider ID:** `cloudflare-ai-gateway`
- **Default model:** `claude-sonnet-4-5` (200K context, 64K max tokens)
- **API format:** `anthropic-messages`
- **Base URL pattern:** `https://gateway.ai.cloudflare.com/v1/${accountId}/${gatewayId}/anthropic`
- **Env var:** `CLOUDFLARE_AI_GATEWAY_API_KEY` (your Anthropic API key)

Files added/modified (29 files, +663 lines):
- `src/agents/cloudflare-ai-gateway.ts` — Provider definition with `resolveCloudflareAiGatewayBaseUrl()`
- `src/agents/models-config.providers.ts` — Provider config builder integration
- `src/commands/onboard-auth.config-core.ts` — `applyCloudflareAiGatewayConfig()` and `applyCloudflareAiGatewayProviderConfig()`
- `src/commands/onboard-auth.credentials.ts` — `setCloudflareAiGatewayConfig()` credential storage
- `src/commands/auth-choice.apply.api-providers.ts` — Auth choice routing for `cloudflare-ai-gateway-api-key`
- `src/cli/program/register.onboard.ts` — CLI flags for `--cloudflare-ai-gateway-*`
- `src/commands/onboard-non-interactive.cloudflare-ai-gateway.test.ts` — 99 lines of test coverage
- `docs/providers/cloudflare-ai-gateway.md` — Setup documentation

See: https://developers.cloudflare.com/ai-gateway/

### Post-Merge Hardening (Feb 5 sync 2)

Four security-relevant commits:

- **`392bbddf2`** — Owner-only tools + command auth hardening (#9202): New `applyOwnerOnlyToolPolicy()` (`src/agents/tool-policy.ts:91-110`) gates sensitive tools (currently `whatsapp_login`) to owner senders only. Treats undefined `senderIsOwner` as unauthorized (default-deny). New `commands.ownerAllowFrom` config parameter for explicit owner identification. Defense-in-depth for tool access control (thanks @victormier).

- **`4434cae56`** — Harden sandboxed media handling (#9182): New `assertMediaNotDataUrl()` and `resolveSandboxedMediaSource()` (`src/agents/sandbox-paths.ts:55-82`) block data-URL payloads and validate media paths within sandbox boundaries. Enforcement moved to `message-action-runner.ts` for delivery-point validation. Prevents path traversal and sandbox escape via media parameters (thanks @victormier).

- **`a13ff55bd`** — Gateway credential exfiltration prevention (#9179): New `resolveExplicitGatewayAuth()` and `ensureExplicitGatewayAuth()` (`src/gateway/call.ts:59-89`) require explicit credentials when `--url` is overridden to non-local addresses. Prevents credential leakage to attacker-controlled URLs (CWE-522). Local addresses (127.0.0.1, private IPs, tailnet 100.x.x.x) retain credential fallback (thanks @victormier).

- **`385a7eba3`** — Enforce owner allowlist for commands: Hardens `commands.ownerAllowFrom` enforcement (`src/auto-reply/command-auth.ts:203-328`)—when explicit owners are configured, non-matching senders cannot execute commands even if `allowFrom` is wildcard.

### Post-Merge Hardening (Feb 5 sync 3)

Two security-relevant commits strengthening Windows ACL test coverage:

- **`f26cc6087`** — Tests: add test coverage for security/windows-acl.ts: Adds 26 comprehensive unit tests for Windows ACL inspection utilities including `resolveWindowsUserPrincipal`, `parseIcaclsOutput`, `summarizeWindowsAcl`, `inspectWindowsAcl`, `formatWindowsAclSummary`, and `createIcaclsResetCommand`. Strengthens Windows file permission security testing (thanks @M00N7682).

- **`d6cde28c8`** — fix: stabilize windows acl tests and command auth registry (#9335): Stabilizes Windows ACL tests and fixes command auth registry behavior (thanks @M00N7682).

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation, bootstrap/memory .md scanning).

### Post-Merge Hardening (Feb 6 sync 2)

Six security-relevant commits:

- **`8fdc0a284` + `873182ec2` + `b8004a28c`** — Secure DM guidance improvements: Enhanced documentation for `session.dmScope: "per-account-channel-peer"` configuration in multi-user DM setups. Adds prominent security warning about context leakage risk and concrete example. Complements PR #2 per-account session isolation hardening (`d499b14`). Thanks @Shrinija17.

- **`d6c088910`** — Credential protection via .gitignore: Adds `memory/` and `.agent/*.json` (excluding `workflows/`) to gitignore, preventing accidental commit of agent credentials and session data. Defense-in-depth for credential hygiene.

- **`ea237115a`** — CLI flag handling refinement: Passes `--disable-warning=ExperimentalWarning` as Node CLI argument instead of via NODE_OPTIONS environment variable (fixes npm pack compatibility). Defense-in-depth for env var handling—NOT directly related to audit claim #8 (LD_PRELOAD/NODE_OPTIONS injection), which is already mitigated via blocklists in `src/node-host/runner.ts:166-175` and `src/agents/bash-tools.exec.ts:61-78` (PR #12). Thanks @18-RAJAT.

- **`93b450349`** — Session state hygiene: Clears stale token metrics (totalTokens, inputTokens, outputTokens, contextTokens) when starting new sessions via /new or /reset. Prevents misleading context usage display from previous sessions.

- **`f32eeae3b`** — Compaction transcript repair: Removes orphaned tool_result messages when assistant messages with tool_use blocks are dropped during compaction pruning. Prevents API rejections due to unexpected tool_use_id. Defense-in-depth against prompt injection via malformed session transcripts.

- **`7c951b01a`** — Feishu mention gating: Requires bot open_id match for group mention detection when bot ID is available. Prevents agent replies when other users (not the bot) are mentioned in Feishu groups. Access control hardening.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation, bootstrap/memory .md scanning).

### Post-Merge Hardening (Feb 6 sync 3)

Fourteen security-relevant commits:

**CRITICAL (3):**

- **`47538bca4` + `a459e237e`** (PR [#9518](https://github.com/openclaw/openclaw/pull/9518)) — **Canvas auth bypass fix:** FIXES tracked issue [#9517](https://github.com/openclaw/openclaw/issues/9517). New `authorizeCanvasRequest()` function in `src/gateway/server-http.ts:96-130` wraps all canvas/A2UI HTTP and WebSocket requests with bearer-token + authorized-WebSocket-client authentication. Canvas host paths now require either a valid gateway auth token or an already-authenticated WebSocket connection from the same IP. E2E tests: `src/gateway/server.canvas-auth.e2e.test.ts` (212 lines). Thanks @coygeek.

- **`0c7fa2b0d`** (PR [#9858](https://github.com/openclaw/openclaw/pull/9858)) — **Credential leakage in config APIs:** `config.get` previously exposed all secrets (tokens, API keys) to any connected gateway client. New `redactConfigSnapshot()` function in `src/config/redact-snapshot.ts:117-126` strips sensitive values from config snapshots before returning them over the gateway wire protocol. Partially addresses tracked issues [#5995](https://github.com/openclaw/openclaw/issues/5995), [#9627](https://github.com/openclaw/openclaw/issues/9627), [#9813](https://github.com/openclaw/openclaw/issues/9813). Tests: `src/config/redact-snapshot.test.ts` (335 lines).

- **`bc88e58fc`** (PR [#9806](https://github.com/openclaw/openclaw/pull/9806)) — **Skill/plugin code safety scanner:** New `src/security/skill-scanner.ts` (441 lines) provides static analysis of skill/plugin source code for dangerous patterns (eval, child_process, network listeners, credential access). `scanDirectoryWithSummary()` at `:415` returns severity-bucketed findings. Integrated into skill installation flow via `collectSkillInstallScanWarnings()` in `src/agents/skills-install.ts:104-131` and into `openclaw security audit --deep` via `collectPluginsCodeSafetyFindings()` and `collectInstalledSkillsCodeSafetyFindings()` in `src/security/audit-extra.ts:1131,1235`. Tests: `src/security/skill-scanner.test.ts` (345 lines).

**HIGH (4):**

- **`141f551a4` + `6ff209e93`** (PRs [#9903](https://github.com/openclaw/openclaw/pull/9903)/[#9790](https://github.com/openclaw/openclaw/pull/9790)) — **Exec-approvals allowlist coercion:** Bare string entries in exec-approvals allowlists bypassed validation because they lacked the required `{ pattern: "..." }` wrapper. New `coerceAllowlistEntries()` in `src/infra/exec-approvals.ts:137-162` normalizes bare strings into proper allowlist objects during config load. Tests: `src/infra/exec-approvals.test.ts` (+130 lines). Thanks @mcaxtr.

- **`57326f72e`** (PR [#2092](https://github.com/openclaw/openclaw/pull/2092)) — **Nextcloud-Talk HMAC signing:** Signs outbound message text with HMAC instead of the full JSON body, preventing message forgery if the transport is intercepted. Affects `extensions/nextcloud-talk/src/send.ts`.

- **`34a58b839`** (PR [#9870](https://github.com/openclaw/openclaw/pull/9870)) — **Ollama API key env var:** Adds `OLLAMA_API_KEY` support to the Ollama provider configuration, enabling token-based authentication for remote Ollama deployments. Affects `src/agents/models-config.providers.ts` and `src/agents/model-auth.ts`.

- **`02842bef9`** (PR [#9971](https://github.com/openclaw/openclaw/pull/9971)) — **Slack mention stripping:** Strips `@mentions` from Slack messages before processing `/new` and `/reset` commands to prevent mention-injection where crafted messages could trigger unintended bot responses.

**MEDIUM (3 groups):**

- **Cron race conditions** (3 commits — PRs [#9823](https://github.com/openclaw/openclaw/pull/9823)/[#9948](https://github.com/openclaw/openclaw/pull/9948)/[#9932](https://github.com/openclaw/openclaw/pull/9932)) — Fixes race conditions in cron job scheduling where concurrent timer fires could duplicate job executions. Adds locking in `src/cron/service/timer.ts`, `src/cron/service/jobs.ts`, and `src/cron/service/store.ts`. Tests: `src/cron/service.every-jobs-fire.test.ts` (127 lines), `src/cli/cron-cli/shared.test.ts` (63 lines).

- **`ec0728b35`** (PR [#1962](https://github.com/openclaw/openclaw/pull/1962)) — **Session lock release:** Ensures session write locks are properly released on abnormal process exit, preventing deadlocked sessions. Affects `src/agents/session-write-lock.ts`.

- **`861725fba`** (PR [#4598](https://github.com/openclaw/openclaw/pull/4598)) — **Aborted message tool extraction:** Fixes tool_use extraction from aborted/errored assistant messages during session transcript repair. Prevents orphaned tool_result blocks from causing API rejection. Affects `src/agents/session-transcript-repair.ts`.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation, bootstrap/memory .md scanning).

### Post-Merge Hardening (Feb 6 sync 4)

Four security-relevant commits:

**HIGH (2):**

- **`717129f7f`** (PR [#9436](https://github.com/openclaw/openclaw/pull/9436)) — **Remove auth tokens from URL query parameters:** Complete removal of query-parameter token acceptance. `extractHookToken()` in `src/gateway/hooks.ts:92-109` no longer accepts `url.searchParams.get("token")`. New explicit HTTP 400 rejection in `src/gateway/server-http.ts:154-161` when `?token=` is present. Dashboard URL no longer appends `?token=`. **FIXES** tracked issues #5120 and #9435 (CWE-598). Thanks @coygeek.

- **`bccdc95a9`** (PR [#10000](https://github.com/openclaw/openclaw/pull/10000)) — **Cap sessions_history payloads:** New `SESSIONS_HISTORY_MAX_BYTES` (80KB) and `SESSIONS_HISTORY_TEXT_MAX_CHARS` (4000) in `src/agents/tools/sessions-history-tool.ts:24-25`. Sanitization strips thinking signatures, image data, usage/cost metadata. Prevents DoS via unbounded session history injection. Thanks @gut-puncture.

**MEDIUM (2):**

- **`c75275f10`** (PR [#10146](https://github.com/openclaw/openclaw/pull/10146)) — **Harden control UI asset handling in update flow:** New `resolveControlUiDistIndexHealth()` in `src/infra/control-ui-assets.ts:19-32`. Update runner uses explicit entry point and adds post-doctor UI repair. Defense-in-depth for update flow integrity. Thanks @gumadeiras.

- **`4a59b7786`** — **Harden CLI update restart imports and version resolution:** Version resolution in `src/version.ts` uses structured candidate search with package name validation (`PACKAGE_JSON_CANDIDATES` at line 6, `BUILD_INFO_CANDIDATES` at line 13). Defense-in-depth for self-update integrity.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation, bootstrap/memory .md scanning).

### Post-Merge Hardening (Feb 7 sync 1)

One security-relevant commit:

**MEDIUM (1):**

- **`421644940`** (PR [#10176](https://github.com/openclaw/openclaw/pull/10176)) — **Guard resolveUserPath against undefined input:** New `resolveRunWorkspaceDir()` in `src/agents/workspace-run.ts:72` validates workspace dir type/value before resolution, falls back to per-agent defaults (not CWD). New `classifySessionKeyShape()` in `src/routing/session-key.ts:62` rejects malformed `agent:` session keys. New SHA256-based identifier redaction in `src/logging/redact-identifier.ts` for safe audit logging. Addresses **Audit 1 Claim #6** (path traversal in agent dirs) — adds defense-in-depth upstream of `resolveUserPath()` (`src/agents/agent-paths.ts:10,13` → `src/utils.ts:243,245`). 139 new test lines covering edge cases. Thanks @Yida-Dev.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation, bootstrap/memory .md scanning).

### Post-Merge Hardening (Feb 7 sync 2)

One security-adjacent commit (reliability/hardening focus, continues cron race condition work from Feb 6 sync 3):

**LOW (1):**

- **`d90cac990`** (PR [#10776](https://github.com/openclaw/openclaw/pull/10776)) — **Cron scheduler reliability, store hardening, and UX improvements:** Adds `isJobDue()` guard in `src/cron/service/timer.ts` to prevent stale timer firings. Reduces `MAX_TIMER_DELAY_MS` for tighter scheduling. Input normalization hardened in `src/cron/normalize.ts`. Store state initialization improved in `src/cron/service/store.ts` with migration support. 2,952 lines added across 58 files (mostly tests + UI). Continues cron race condition hardening from Feb 6 sync 3 (`1ecae8098`, `8e74fbb41`).

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation, bootstrap/memory .md scanning).

### Post-Merge Hardening (Feb 7 sync 3)

35 upstream commits, primarily Baidu Qianfan provider support (PR [#8868](https://github.com/openclaw/openclaw/pull/8868)), CI pipeline optimization, and release version bumps (2026.2.6-1 through 2026.2.6-3).

**LOW (1):**

- **`c5194d814`** — **Dashboard token delivery via URL fragment:** Restores token-authenticated dashboard URLs using URL fragments (`#token=`) instead of the previously removed query parameters (`?token=`). Fragments are not sent to servers, not logged in access logs, and not included in Referer headers (CWE-598 mitigation preserved). Follows PR [#9436](https://github.com/openclaw/openclaw/pull/9436) which removed query-param tokens entirely. Affects `src/commands/dashboard.ts`, `src/commands/onboard-helpers.ts`, `src/wizard/onboarding.finalize.ts`, `ui/src/ui/app-settings.ts`.

**Notable non-security changes:**
- **Baidu Qianfan provider** (`88ffad1c4`, PR [#8868](https://github.com/openclaw/openclaw/pull/8868)): New `QIANFAN_API_KEY` env var in `src/agents/model-auth.ts:305`, provider config in `src/agents/models-config.providers.ts`, onboarding flow in `src/commands/onboard-auth.config-core.ts`. Thanks @ide-rea.
- **Voyage AI embeddings fix** (`e78ae48e6`, PR [#10818](https://github.com/openclaw/openclaw/pull/10818)): Adds `input_type` parameter to Voyage AI embedding requests for improved retrieval accuracy.
- **CI pipeline optimization** (`47596257e`, `2d7428a7f`): Concurrency controls, consolidated macOS jobs, re-enabled parallel vitest on Windows.

**Line number verification:** All 14 key security function references verified via LSP — no line shifts in this sync (0 security-critical source files changed).

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation, bootstrap/memory .md scanning).

### Post-Merge Hardening (Feb 9 sync 1)

43 upstream commits. Key changes: new agent CRUD API (RBAC-gated), gateway LAN bind fix, sandbox USER directive fix, STATE_DIR credential path fix, cron isolation hardening, context overflow recovery, new MITRE ATLAS threat model documentation.

**HIGH (3):**

- **`980f78873`** (PR [#11045](https://github.com/openclaw/openclaw/pull/11045)) — **Agent CRUD RBAC gating:** New `agents.create`, `agents.update`, `agents.delete` gateway methods gated behind `operator.admin` scope in `authorizeGatewayMethod()` (`src/gateway/server-methods.ts:146-148`). Prevents non-admin clients from creating/modifying/deleting agents. Strengthens Audit 2 Claim 5 (agent self-approval) controls — agents cannot call these methods.

- **`b8c8130ef`** (PR [#11448](https://github.com/openclaw/openclaw/pull/11448)) — **Gateway LAN IP bind fix:** New `pickPrimaryLanIPv4()` in `src/gateway/net.ts:9-25` resolves the primary non-internal IPv4 address for `bind=lan` mode. WebSocket and probe URLs now use the actual LAN IP instead of `0.0.0.0`, preventing unintended exposure of internal URLs to external networks.

- **`28e1a65eb`** (PR [#11289](https://github.com/openclaw/openclaw/pull/11289)) — **Sandbox USER directive fix:** Adds `USER root` directive to `Dockerfile.sandbox` and `Dockerfile.sandbox-browser`, fixing sandbox container user context. Also fixes `workspace:*` protocol references in extension package.json files and removes dead config entries.

**MEDIUM (4):**

- **`ebe573040`** (PR [#4824](https://github.com/openclaw/openclaw/pull/4824)) — **STATE_DIR credential path fix:** Device identity and canvas host now use `STATE_DIR` environment variable instead of hardcoded `~/.openclaw`, preventing credential path misalignment in non-standard installations (e.g., Docker, Nix). Affects `src/infra/device-identity.ts`, `src/canvas-host/server.ts`. New test: `src/infra/device-identity.state-dir.test.ts`.

- **`8fae55e8e`** (PR [#11641](https://github.com/openclaw/openclaw/pull/11641)) — **Cron isolated announce flow hardening:** Shared isolated announce flow between cron jobs and main agent. Hardened cron scheduling and delivery with improved timer management and job state tracking. Continues cron race condition work from Feb 6 sync 3.

- **`ea423bbbf`** — **Context overflow sanitization:** Enhanced error handling in `sanitizeUserFacingText()` for context overflow scenarios. Improves user-facing error messages when agent encounters context limits, reducing information leakage in error paths.

- **`0deb8b0da`** (PR [#11579](https://github.com/openclaw/openclaw/pull/11579)) — **Tool result overflow recovery:** New `src/agents/pi-embedded-runner/tool-result-truncation.ts` (328 lines) detects and truncates oversized tool results that cause context overflow. Prevents DoS via tools returning unbounded output. Tests: `src/agents/pi-embedded-runner/tool-result-truncation.test.ts` (215 lines).

**Notable non-security changes:**
- **MITRE ATLAS threat model** (`74fbbda28`): New `docs/security/THREAT-MODEL-ATLAS.md` (603 lines) documents 20+ threats across 8 MITRE ATLAS tactics. Identifies P0 risks: direct prompt injection, malicious skill installation, credential harvesting via skills. References key security files including `src/infra/exec-approvals.ts`, `src/gateway/auth.ts`, `src/infra/net/ssrf.ts`.
- **Memory hardening** (5 commits): QMD startup resilience, SQLITE_BUSY fallback, cache eviction idempotency, forced sync queuing.
- **CI pipeline overhaul** (`multiple`): Docs-only detection, pnpm store caching, consolidated test sharding.

**Line number shifts in this sync:** `src/gateway/server-methods.ts` +3 lines (93-160 → 93-163), `src/gateway/net.ts` +24 lines (all functions shifted: `isTrustedProxyAddress` 74→98, `resolveGatewayClientIp` 82→106, `resolveGatewayBindHost` 129→153). All references updated and LSP-verified.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation, bootstrap/memory .md scanning).

### Post-Merge Hardening (Feb 9 sync 3)

45 upstream commits. Key changes: device pairing + phone control plugins (new attack surface), iOS alpha node app, centralized home-dir resolution, transcript integrity fix, routing/thread guards tightened, exec approval UI clarity.

**HIGH — New attack surface (1):**

- **`730f86dd5`** (PR [#11755](https://github.com/openclaw/openclaw/pull/11755)) — **Gateway/Plugins: device pairing + phone control:**
  - New `/pair` command generates base64-encoded setup codes containing gateway URL + auth credentials. Credentials transmitted in a base64 blob (not encrypted) over the chat channel.
  - New `/phone` command temporarily arms/disarms dangerous node commands with auto-expiry timer (default 10m), modifying the live config file.
  - **Positive:** New default-deny policy for dangerous commands — `camera.snap`, `camera.clip`, `screen.record`, `contacts.add`, `calendar.add`, `reminders.add`, `sms.send` are now in `DEFAULT_DANGEROUS_NODE_COMMANDS` (`src/gateway/node-command-policy.ts`) and must be explicitly enabled via `gateway.nodes.allowCommands`. **Partially mitigates Gap #3** (outPath in screen_record) since `screen.record` is now default-denied.
  - Session key canonicalization in `chat.ts` distinguishes `rawSessionKey` from `canonicalKey`, preventing session confusion attacks.

**MODERATE — Path hardening (2):**

- **`db137dd65`** (PR [#12091](https://github.com/openclaw/openclaw/pull/12091)) — **fix(paths): respect OPENCLAW_HOME for all internal paths:** New centralized `src/infra/home-dir.ts` with `resolveEffectiveHomeDir()` using precedence: `OPENCLAW_HOME` > `HOME` > `USERPROFILE` > `os.homedir()`. Eliminates scattered `os.homedir()` calls. All path-sensitive callsites migrated (config IO, agent dirs, session transcripts, pairing store, cron store, CLI profiles, OAuth dir). Strengthens Audit 1 Claim 6 (path traversal) defense-in-depth.

- **`456bd5874`** (PR [#12125](https://github.com/openclaw/openclaw/pull/12125)) — **fix(paths): structurally resolve home dir:** Single `path.resolve()` exit point in `resolveEffectiveHomeDir()` prevents unresolved paths from escaping. Removes dangerous colon-split in `tool-meta.ts` `shortenMeta()` that broke on Windows drive letters.

**MODERATE — Transcript integrity (1):**

- **`0cf93b8fa`** (PR [#12283](https://github.com/openclaw/openclaw/pull/12283)) — **Gateway: fix post-compaction amnesia for injected messages:** Eliminates raw JSONL appends for injected assistant messages. Now uses `SessionManager.appendMessage()` with proper `parentId` chain. Fixes `stopReason` from invalid `"injected"` to `"stop"` with explicit metadata. Prevents context amnesia where compaction summaries and security-relevant prior instructions could be lost.

**LOW-MODERATE (3):**

- **`d85f0566a`** — **fix: tighten thread-clear and telegram retry guards:** Thread-clear in `updateLastRoute` now only clears when explicit route is provided. `hasMessageThreadIdParam` validates value is finite number or non-empty string, preventing invalid thread IDs causing cross-thread message delivery.

- **`8d96955e1`** (PR [#11372](https://github.com/openclaw/openclaw/pull/11372)) — **fix(routing): make bindings dynamic per-message:** Calls `loadConfig()` fresh per incoming message instead of using stale startup config. Security-relevant: routing restriction changes (e.g., restricting agent access) take effect immediately without restart.

- **`ad8b839aa`** (PR [#11937](https://github.com/openclaw/openclaw/pull/11937)) — **Exec approvals: render forwarded commands in monospace:** Exec-approval requests render commands in backtick-fenced code blocks with escalating fence lengths. Helps operators accurately review commands before approving, reducing risk of approving visually misleading commands.

**Line number shifts in this sync:** `src/config/io.ts` +2-3 lines (writeConfigFile 480→482, 0o700 at 495→497, 0o600 at 489→509). `src/routing/session-key.ts` +30 lines (per-account-channel-peer 119,135→148,166-170). `src/agents/bash-tools.exec.ts` +5 lines (env merge 967→972, validateHostEnv 975→976-977, bypassApprovals 1273→1278). `src/config/paths.ts` reorganized (resolveUserPath at 87-105, resolveCanonicalConfigPath at 114-123). `src/plugins/config-state.ts` +4 (loadPaths 69→73, enable 190→194). All references updated and LSP-verified.

**CVE status:** 5 published advisories (CVE-2026-25593, CVE-2026-25475, GHSA-g8p2-7wf7-98mq, CVE-2026-25157, CVE-2026-24763) — all pre-existing, none patched in this merge.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation — Gap #3 partially mitigated by default-deny on `screen.record`, bootstrap/memory .md scanning).

### Post-Merge Hardening (Feb 10 sync 3, 26 upstream commits)

Primarily CI infrastructure (code-size gates, tiered lint/format), extension TypeScript fixes, and test coverage expansion. One security-relevant fix:

**LOW — Access control correctness (1):**

- **`29425e27e`** (PR [#12779](https://github.com/openclaw/openclaw/issues/12779), thanks @liuxiaopai-ai) — **fix(telegram): match DM allowFrom against sender user id:** DM access control previously used `chatId` as the `allowFrom` match key. For Telegram DMs, `allowFrom` entries are typically user IDs (`msg.from.id`) or `@usernames`. Using `chatId` caused legitimate DMs to be rejected. Now matches against `msg.from.id` with `chatId` fallback (`src/telegram/bot-message-context.ts:231-233`). Regression tests added in `a77afe618`.

**Other notable changes:**

- **`2e4334c32`** — **test(auth): cover key normalization:** 222 new test lines covering API key normalization across MiniMax VLM, Firecrawl web-fetch, skills.update, and provider-usage auth paths. Increases confidence in credential handling correctness.

- **`512b2053c`** (PR [#12795](https://github.com/openclaw/openclaw/pull/12795)) — **fix(web_search): Fix invalid model name sent to Perplexity:** Strips `perplexity/` prefix from model names when sending directly to `api.perplexity.ai`. Adds `resolvePerplexityRequestModel()` and `isDirectPerplexityBaseUrl()` functions, shifting all line numbers in `src/agents/tools/web-search.ts` by +21 below line 283.

**Line number shifts in this sync:** `src/agents/tools/web-search.ts` +21 lines at line 283 (Perplexity model resolution functions inserted). References updated: `wrapWebContent` 509,580,582→530,601,603; X-Title 390→411; Brave headers 558-563→579-584; Grok fetch 442-448→463-469.

**CVE status:** 5 published advisories — all pre-existing, none patched in this merge.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation — Gap #3 partially mitigated by default-deny on `screen.record`, bootstrap/memory .md scanning).

### Post-Merge Hardening (Feb 10 sync 5, 51 upstream commits)

**CRITICAL — Anti-spoofing & access control (4):**

1. **`53273b490`** — **fix(auto-reply): prevent sender spoofing in group prompts:** Separates user-controlled content from trusted metadata in system prompts. Introduces `BodyForAgent` with clean text, moves metadata to JSON blocks with "untrusted" labels. Removes attacker-controlled group subject/member lists from system prompts. Prevents bracket injection in envelope headers. Affects all channels (42 files changed). New `src/auto-reply/reply/inbound-meta.ts` replaces vulnerable `inbound-sender-meta.ts`.

2. **`4537ebc43`** — **fix: enforce Discord agent component DM auth:** Prevents unauthorized users from injecting system events via Discord buttons/select menus. New `src/discord/monitor/agent-components.ts` (525 lines) with `ensureDmComponentAuthorized()`. Uses `rawData.channel_id` as source of truth to prevent channel spoofing. Validates against DM allowlists before processing component interactions.

3. **`47f6bb414`** — **Commands: add commands.allowFrom config:** New `commands.allowFrom` configuration for per-provider command authorization separate from channel access. Supports global (`"*"`) and provider-specific lists. Expands `resolveCommandAuthorization()` in `src/auto-reply/command-auth.ts:203-328`. **Strengthens Audit 2 "self-approving agent" (Claim 5)** by enabling granular command authorization.

4. **`1d46ca3a9`** — **fix(signal): enforce mention gating for group messages:** Signal group messages bypassed `requireMention` configuration. New `src/signal/monitor/event-handler.ts:87` enforcement with 206-line test suite. Aligns Signal with Slack/Discord/Telegram mention gating.

**MODERATE — Code health / auditability (1):**

5. **`f17c978f5`** — **refactor(security,config): split oversized files:** Splits `src/security/audit-extra.ts` (1,199 LOC barrel) into `audit-extra.sync.ts` (618 LOC, config-based checks) + `audit-extra.async.ts` (720 LOC, I/O-based checks). Also splits `src/config/schema.ts`. Improves code auditability.

**LOW — Defense in depth (5):**

6. **`e19a23520`** — **fix: unify session maintenance and cron run pruning:** New `src/cron/session-reaper.ts` implements session pruning with entry capping and file rotation. Prevents session.json DoS from unbounded growth.

7. **`8ff1618bf`** — **Discord: add exec approval cleanup option:** `execApprovals.cleanup` removes approval UI after handling. Reduces message clutter in approval channels.

8. **`54315aeac`** — **Agents: scope sanitizeUserFacingText rewrites to errorContext:** `sanitizeUserFacingText()` now only applies error keyword rewrites when `errorContext: true`. Prevents false positive error detection in normal assistant replies.

9. **`d3c71875e`** — **fix: cap Discord gateway reconnect at 50 attempts:** Prevents infinite reconnect loop DoS.

10. **`8d75a496b`** — **refactor: centralize isPlainObject, isRecord, isErrno, isLoopbackHost utilities:** Consolidates into `src/utils.ts` and `src/infra/errors.ts`. Moves `isLoopbackHost()` to `src/gateway/net.ts:263` and `isLoopbackAddress()` consolidation.

**Line number shifts in this sync:** `web-search.ts` +17 (410→427, 579→596, 463→480), `origin-check.ts` -14 (57-85→43-71, `isLoopbackHost` moved to `net.ts`), `auth.ts` -14 (107-128→93-114, `isLoopbackAddress` consolidated), `oauth.ts` -23 (618→595, WSL dedup), `command-auth.ts` expanded (216-259→203-328, `commands.allowFrom` added), `audit-extra.ts` split to barrel (sync: `167-172`, async: `547-720`), `sessions/store.ts` major refactor (364→667, 422→729, pruning overhaul).

**CVE status:** 5 published advisories — all pre-existing, none patched in this merge.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation — Gap #3 partially mitigated, bootstrap/memory .md scanning).

### Post-Merge Hardening (Feb 10 sync 7, 6 upstream commits)

Primarily QMD memory query scoping, legacy config migration for `memorySearch`, and gateway eager-init for QMD backend. One security-relevant fix:

**LOW — Data isolation (1):**

- **`ef4a0e92b`** (PR [#11645](https://github.com/openclaw/openclaw/pull/11645)) — **fix(memory/qmd): scope query to managed collections:** `QmdMemoryManager.query()` now passes `-c <collection>` flags via new `buildCollectionFilterArgs()` (`src/memory/qmd-manager.ts:1006-1012`), restricting QMD searches to explicitly configured collections only. If no managed collections are configured, queries return empty results with a warning log. Defense-in-depth for **Gap #4** (bootstrap/memory `.md` scanning) — ensures QMD queries cannot inadvertently access unmanaged collection data.

**Other notable changes:**

- **`efc79f69a`** — **Gateway: eager-init QMD backend on startup:** New `startGatewayMemoryBackend()` in `src/gateway/server-startup-memory.ts` initializes QMD backend during gateway startup rather than lazily on first query. Improves reliability; non-blocking (failure logged as warning).

- **`868873016` + `a76dea0d2` + `8d80212f9`** — **Config: migrate legacy top-level memorySearch:** Adds migration rule for `memorySearch` → `agents.defaults.memorySearch` in `src/config/legacy.rules.ts:88-92` and `src/config/legacy.migrations.part-3.ts`. Changelog credited.

- **`40919b1fc`** — **fix(test): add StringSelectMenu to @buape/carbon mock:** Non-security test fix.

**Line number shifts in this sync:** `src/memory/qmd-manager.ts` +5 lines at line 265 (collection filter args), +8 lines at end (new `buildCollectionFilterArgs()` method). References updated: `qmd-manager.ts:331-337` → `336-342` (4 files).

**CVE status:** 5 published advisories — all pre-existing, none patched in this merge.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation — Gap #3 partially mitigated, bootstrap/memory .md scanning — Gap #4 strengthened by collection scoping).

### Post-Merge Notes (Feb 11 sync 1, 22 upstream commits)

**Merge commit:** `05baacb2a` | **Range:** `2a5799afb..05baacb2a`

**Security relevance: LOW** — zero overlap with 25 key security-referenced source files. No line number shifts in any documented files.

**Breakdown:** ~13 docs/CI/maintenance, 5 bug fixes, 3 features, 1 revert.

**Notable commits:**

- **`67d25c653` + `22458f57f` + `c4d3800c2`** — **fix: strip `<think>` reasoning tags and `[Historical context: ...]` from messaging tool send path** (`src/agents/tools/message-tool.ts`). Sanitization improvement — prevents model internals (reasoning traces) from leaking to end users via the messaging tool. LOW security relevance (defense-in-depth for information disclosure).

- **`31f616d45`** — **feat: ClawDock — shell docker helpers for dev** (`scripts/shell-helpers/clawdock-helpers.sh`, #12817). Developer convenience scripts for Docker-based development. NONE security relevance.

- **`c0befdee0`** — **feat(onboard): custom/local API configuration flow** (`src/commands/onboard-custom.ts`, #11106). New onboarding path for custom API providers. LOW security relevance (new provider config path, no auth bypass).

- **`5fab11198`** — **fix: matrix media attachments** (`extensions/matrix/src/matrix/monitor/media.ts`, #12967). Bug fix for media handling. NONE security relevance.

- **`6731c6a1c`** — **fix(docker): support Bash 3.2 in docker-setup.sh** (#9441). Compatibility fix. NONE security relevance.

**Other changes:** Maintainer workflow docs (AGENTS.md, CONTRIBUTING.md, PR_WORKFLOW.md), CI paths-ignore tweaks, credits/token-use doc updates, onboarding overview docs, env example expansion, stale bot config, embedded subscribe/utils refactoring, revert of docker release workflow change.

**Line number shifts in this sync:** NONE. No documented security source files were modified.

**CVE status:** 5 published advisories — all pre-existing, none patched in this merge.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation — Gap #3 partially mitigated, bootstrap/memory .md scanning — Gap #4 strengthened by collection scoping).

### Post-Merge Hardening (Feb 11 sync 2, 17 upstream commits)

**Merge commit:** `86aca131b` | **Range:** `05baacb2a..86aca131b`

**Security relevance: LOW** — no direct overlap with the 16 audit claims or 3 legitimate gaps. Five commits are security-adjacent (defense-in-depth improvements, new attack surface from IRC).

**Security-relevant commits (5):**

**MEDIUM (3):**

1. **`7f1712c1b`** (PR [#13455](https://github.com/openclaw/openclaw/pull/13455)) — **Enforce embedding model token limit to prevent overflow:** New `enforceEmbeddingMaxInputTokens()` in `src/memory/embedding-chunk-limits.ts` with per-model limits (`src/memory/embedding-model-limits.ts`). Uses UTF-8 byte length as conservative upper bound. Splits oversized chunks via binary search. Prevents DoS from oversized embedding inputs.

2. **`424d2dddf`** (PR [#13498](https://github.com/openclaw/openclaw/pull/13498)) — **Prevent act:evaluate hangs from blocking browser tool:** Clamps `evaluate()` timeout to 120s max in `src/browser/pw-tools-core.interactions.ts:235-246`. Injects `Promise.race` timeout into browser context. Adds abort signal handling for force-disconnect. Prevents DoS from runaway browser evaluations.

3. **`fa906b26a`** — **IRC first-class channel support:** New `extensions/irc/` with TLS client, NickServ auth, channel allowlisting (`extensions/irc/src/policy.ts`), case-insensitive matching, and control character sanitization. **New attack surface** — external network protocol exposure. TLS uses default Node.js cert validation.

**LOW (2):**

4. **`841dbeee0`** (PR [#13468](https://github.com/openclaw/openclaw/pull/13468)) — **Coerce form values to schema types before config.set:** New `ui/src/ui/controllers/config/form-coerce.ts` converts HTML form strings to proper types (number, boolean) before Zod validation. Prevents type confusion at Web UI → Gateway boundary.

5. **`ca629296c`** (PR [#13672](https://github.com/openclaw/openclaw/pull/13672)) — **Add agentId support to webhook mappings:** Extends `src/gateway/hooks.ts` and `src/gateway/server-http.ts` with per-mapping `agentId` fields and `hooks.allowedAgentIds` allowlist enforcement. New `isHookAgentAllowed()`, `resolveHookTargetAgentId()`. Positive: follows least privilege for webhook → agent routing.

**Other notable (non-security):**

- `2b02e8a7a` (PR [#10568](https://github.com/openclaw/openclaw/pull/10568)) — Gateway: stream thinking events and decouple tool events from verbose level
- `d2c2f4185` (PR [#13733](https://github.com/openclaw/openclaw/pull/13733)) — Heartbeat: inject current time into agent prompts
- `45488e4ec` (PR [#12102](https://github.com/openclaw/openclaw/pull/12102)) — Fix: remap session JSONL chunk line numbers to original source positions
- `74273d62d` (PR [#13723](https://github.com/openclaw/openclaw/pull/13723)) — Fix(pairing): show actual code in approval command instead of placeholder

**Line number shifts in this sync:** `runner.ts` +1 (165-174→166-175, new `withTimeout` import), `hooks.ts` +46 (46-63→92-109, 65-111→111-157, new types/agent policy functions/imports), `server-http.ts` +4/+18 (92-126→96-130, 150-157→154-161, 332→350, 356-376→374-394, 418-440→436-458), `manager.ts` refactored (2257-2354→2198-2301, session helpers extracted). All references updated in documentation.

**CVE status:** 5 published advisories — all pre-existing, none patched in this merge.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation — Gap #3 partially mitigated, bootstrap/memory .md scanning — Gap #4 strengthened by collection scoping).

### Post-Merge Hardening (Feb 12 sync 1, 32 upstream commits)

**Merge commit:** `3ed7dc83a` | **Range:** `86aca131b..3ed7dc83a`

**CRITICAL — Scope hardening & supply chain (2):**

1. **`cfd112952`** — **fix(gateway): default-deny missing connect scopes:** Removed implicit `operator.admin` scope grant when scopes are empty or omitted in WebSocket connect params. Changed `message-handler.ts` from `requestedScopes.length > 0 ? requestedScopes : role === "operator" ? ["operator.admin"] : []` to `Array.isArray(connectParams.scopes) ? connectParams.scopes : []`. Prevents privilege escalation via omitted scope arrays. **Strengthens Audit 2 Claim 5** (self-approving agent / no RBAC). 170+ lines of new test coverage.

2. **`92702af7a`** — **fix(plugins): ignore install scripts during plugin/hook install:** Added `--ignore-scripts` flag to `npm install` in both `src/plugins/install.ts` and `src/hooks/install.ts`. Prevents RCE via malicious npm postinstall/preinstall lifecycle scripts in plugin/hook dependencies. New supply chain defense-in-depth.

**MEDIUM — Config correctness (1):**

3. **`66ca5746c`** — **fix(config): avoid redacting maxTokens-like fields:** Anchored sensitive-key regex from `/token/i` to `/token$/i` in `src/config/redact-snapshot.ts:15`, `schema.field-metadata.ts`, `schema.hints.ts`. Prevents false-positive redaction of legitimate numeric fields (`maxTokens`, `contextTokens`). Avoids config round-trip validation failures.

**Other notable changes:**

4. **`27453f5a3`** — **fix(web-search): handle xAI Responses API format in Grok provider:** Expanded `extractGrokContent()` to iterate message outputs, extract `url_citation` annotations. Line number shifts documented below.

5. **`e85bbe01f`** — **fix: report subagent timeout as 'timed out' instead of 'completed successfully':** Added `"timeout"` status to `AgentRunSnapshot` in `src/agents/subagent-registry.ts`. Prevents false success reporting for timed-out subagents.

6. **`93411b74a`** — **fix(cli): exit with non-zero code when configure/agents-add wizards are cancelled:** Exit code 0→1 on cancellation for automation reliability.

**Line number shifts in this sync:** `web-search.ts` +21-24 below line 109 (Grok type/function expansion + `extractGrokContent()` rewrite). References updated: X-Title 427-428→448-449, Grok fetch 480-486→502-508, wrapWebContent 547,618,620→571,592,642,644, Brave headers 596-601→620-625.

**CVE status:** 5 published advisories — all pre-existing, none patched in this merge.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation — Gap #3 partially mitigated, bootstrap/memory .md scanning — Gap #4 strengthened by collection scoping).

### Post-Merge Notes (Feb 12 sync 2, 6 upstream commits)

**Merge commit:** `d0b825593` | **Range:** `3ed7dc83a..d0b825593`

**Security relevance: LOW** — no overlap with the 16 audit claims or 3 legitimate gaps.

**Security-adjacent (1):**

- **`029b77c85`** (PR [#14223](https://github.com/openclaw/openclaw/pull/14223)) — **Custom provider non-interactive onboarding:** Enables fully unattended `openclaw onboard` with new `--custom-*` flags: `--custom-base-url`, `--custom-model-id`, `--custom-api-key`, `--custom-provider-id`, `--custom-compatibility`. Auth resolution chain: CLI flag > `CUSTOM_API_KEY` env var > auth profile fallback. Auto-inference: providing `--custom-base-url` auto-sets `--auth-choice custom-api-key`. Files: `src/commands/onboard-custom.ts` (+350/-), `src/cli/program/register.onboard.ts`, `src/commands/onboard-non-interactive/local/auth-choice-inference.ts`, `src/commands/onboard-non-interactive/local/auth-choice.ts`, `src/commands/onboard-non-interactive/api-keys.ts`, `src/commands/onboard-types.ts`. Tests: `src/commands/onboard-custom.test.ts`, `src/commands/onboard-non-interactive.provider-auth.test.ts` (240 lines). **Credential hygiene note:** `--custom-api-key` flag exposes the API key in process lists (`ps aux`); prefer `CUSTOM_API_KEY` env var in shared/CI environments.

**Non-security (5):**

- `c8d9733e4` — Changelog: add #13262 entry
- `a67752e6b` — fix(discord): use partial mock for @buape/carbon in slash test
- `940ce424c` — chore: make review mode switching idempotent
- `4200782a5` — fix(heartbeat): honor `heartbeat.model` config for heartbeat turns (#14103). New `src/infra/heartbeat-active-hours.ts` (99 lines) + `src/infra/heartbeat-runner.model-override.test.ts` (246 lines).
- `72fbfaa75` — chore: making PR review chores deterministic + less token hungry

**Line number shifts in this sync:** NONE. No documented security source files were modified.

**CVE status:** 5 published advisories — all pre-existing, none patched in this merge.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation — Gap #3 partially mitigated, bootstrap/memory .md scanning — Gap #4 strengthened by collection scoping).

### Post-Merge Hardening (Feb 12 sync 3, 8 upstream commits)

**Merge commit:** `8518d876a` | **Range:** `d0b825593..8518d876a`

**MEDIUM — Injection prevention & LFI defense (3):**

1. **`1d2c5783f`** (PR [#13830](https://github.com/openclaw/openclaw/pull/13830)) — **fix(agents): enable tool call ID sanitization for Anthropic provider:** `resolveTranscriptPolicy()` in `src/agents/transcript-policy.ts:98` now includes `isAnthropic` in the `sanitizeToolCallIds` condition (was only Google + Mistral). Prevents injection via malformed tool call IDs in Anthropic API sessions. Tests added for all 4 provider policies.

2. **`bebba124e`** (PR [#13952](https://github.com/openclaw/openclaw/pull/13952)) — **fix(ui): escape raw HTML in chat messages:** Added `htmlEscapeRenderer` to `marked.parse()` in `ui/src/ui/markdown.ts:132-133`. Raw HTML in messages is now displayed as escaped text instead of rendered output. DOMPurify already handled XSS, but this prevents confusing UX from pasted HTML (e.g., error pages appearing as formatted content). References #13937.

3. **`4baa43384`** — **fix(media): guard local media reads + accept all path types:** Major refactor of LFI defense for CVE-2026-25475 (GHSA-r8g4-86fx-92mq). `isValidMedia()` (`src/media/parse.ts:36-64`) now accepts all local path types (absolute, tilde, traversal, Windows, UNC, bare filenames). Security validation moved from parse layer to load layer: new `assertLocalMediaAllowed()` (`src/web/media.ts:42-69`) enforces directory root guards (default: `/tmp`, `~/.openclaw/media`, `~/.openclaw/agents`), resolves symlinks before checking. Pre-validated callers pass `localRoots: "any"` to skip the check. Helper `isLikelyLocalPath()` (`src/media/parse.ts:24-33`) extracted. 40+ test lines added.

**LOW — Robustness (3):**

4. **`729181bd0`** (PR [#13747](https://github.com/openclaw/openclaw/pull/13747)) — **fix(agents): exclude rate limit errors from context overflow classification:** Added `isRateLimitErrorMessage()` check before context overflow heuristics in `src/agents/pi-embedded-helpers/errors.ts`. Prevents incorrect truncation/compaction when rate limited (429, "exceeded quota", "rate limit exceeded"). Tests added for 5 rate limit patterns.

5. **`43818e158`** (PR [#13926](https://github.com/openclaw/openclaw/pull/13926)) — **fix(agents): re-run tool_use pairing repair after history truncation:** `limitHistoryTurns()` can orphan `tool_result` blocks by removing the assistant message that contained the matching `tool_use`. Now calls `sanitizeToolUseResultPairing()` after truncation in both `src/agents/pi-embedded-runner/compact.ts:433-438` and `src/agents/pi-embedded-runner/run/attempt.ts:562-567`. Defense-in-depth against malformed conversation state.

6. **`2f1f82674`** + **`3d343932c`** — **Memory/QMD: harden no-results parsing:** QMD query JSON parsing extracted to `src/memory/qmd-query-parser.ts` (47 lines). `parseQmdQueryJson()` is now a standalone function (was `QmdMemoryManager.parseQmdQueryJson()`). Same logic, better isolation. `QmdQueryResult` type also exported. `qmd-manager.ts` reduced by ~36 lines.

**Non-security (1):**

- **`9df89ceda`** — chore: Update deps (pnpm-lock.yaml, package.json version bumps).

**Line number shifts in this sync:** `qmd-manager.ts` -12 lines at 336 (old 336-342→324-329 for `.md` validation), -15 lines at 987 (old 987-993→972-978 for `buildCollectionFilterArgs`). `media/parse.ts` refactored: old `isValidMedia()` 17-33→36-64, new `isLikelyLocalPath()` at 24-33. `attempt.ts` +1 (211-215→212-216, new import). All references updated in documentation.

**CVE status:** 5 published advisories — all pre-existing. Commit `4baa43384` strengthens the fix for CVE-2026-25475 (LFI via MEDIA tokens) by moving validation to load layer with directory root guards.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation — Gap #3 partially mitigated, bootstrap/memory .md scanning — Gap #4 strengthened by collection scoping).

### Post-Merge Hardening (Feb 12 sync 5, 33 commits)

**Merge commit:** `24224da4c` | **Range:** `dd0a2184c..b094491cf`

**LOW — Session isolation (1):**

1. **`631102e71`** (PR [#4887](https://github.com/openclaw/openclaw/pull/4887)) — **fix(agents): scope process/exec tools to sessionKey for isolation:** `createOpenClawCodingTools()` now prefers `options.sessionKey` as the `scopeKey` for process/exec tool lists, falling back to `agentId` only if no sessionKey is provided (`src/agents/pi-tools.ts:215-217`). This prevents cross-session process visibility and process killing — each agent session gets its own isolated process scope. Defense-in-depth for session isolation.

**LOW — Error resilience (1):**

2. **`b912d3992`** (PR [#13500](https://github.com/openclaw/openclaw/pull/13500)) — **(fix): handle Cloudflare 521 and transient 5xx errors gracefully:** New `isCloudflareOrHtmlErrorPage()` and `isTransientHttpError()` helpers in `src/agents/pi-embedded-helpers/errors.ts`. HTML error pages (5xx + `<!DOCTYPE`) are detected and sanitized to a clean user-facing message instead of leaking raw HTML. Model fallback now triggers on transient 5xx HTML error pages. Agent runner retries transient HTTP errors before failing (`src/auto-reply/reply/agent-runner-execution.ts`). Not a direct security fix, but prevents information leakage from raw HTML error pages.

**LOW — Cron robustness (6):**

3. **`04f695e56`** (PR [#14385](https://github.com/openclaw/openclaw/pull/14385)) — **fix(cron): isolate schedule errors to prevent one bad job from breaking all jobs:** `recomputeNextRuns()` now wraps `computeJobNextRunAtMs()` in try/catch per job. Failed jobs increment `scheduleErrorCount` and auto-disable after 3 consecutive errors. Other jobs continue unaffected.

4. **`ace5e33ce`** (PR [#14233](https://github.com/openclaw/openclaw/pull/14233)) — **fix(cron): re-arm timer when onTimer fires during active job execution:** Timer is now re-armed even when a job is already running, preventing missed scheduled runs.

5. **`39e3d58fe`** (PR [#14068](https://github.com/openclaw/openclaw/pull/14068)) — **fix: prevent cron jobs from skipping execution when nextRunAtMs advances:** New `recomputeNextRunsForMaintenance()` function only computes missing `nextRunAtMs` values, preventing silent advancement past due jobs without execution (see #13992).

6. **`a88ea42ec`** (PR [#13878](https://github.com/openclaw/openclaw/pull/13878)) — **fix(cron): prevent one-shot at jobs from re-firing on restart after skip/error:** One-shot `at` jobs that are skipped or errored now have `nextRunAtMs` cleared, preventing re-execution on service restart.

7. **`b0dfb8395`** (PR [#13983](https://github.com/openclaw/openclaw/pull/13983)) — **fix(cron): use requested agentId for isolated job auth resolution:** `runCronIsolatedAgentTurn()` now uses `normalizedRequested ?? defaultAgentId` regardless of whether an agent config entry exists. Ensures auth-profiles, workspace, and agentDir resolve to correct per-agent paths.

8. **`04e3a66f9`** (PR [#14140](https://github.com/openclaw/openclaw/pull/14140)) — **fix(cron): pass agentId to runHeartbeatOnce for main-session jobs:** Heartbeat runs now receive the correct `agentId` for main-session cron jobs.

**Non-security (18):**

- **`b094491cf`** + **`fa427f63b`** — Config: restore schema.ts helper types after hints refactor.
- **`d8016c30c`** — fix: add types condition to plugin-sdk export for moduleResolution NodeNext.
- **`5c32989f5`** — perf: use JSON.parse instead of JSON5.parse for sessions.json (~35x faster).
- **`4b86c9e55`** + **`bbfaac88f`** — Telegram: surface REACTION_INVALID as non-fatal warning, handle no-text message in model picker.
- **`3696b15ab`** — Slack: change default replyToMode from "off" to "all".
- **`472808a20`** — Tests: update thread ID handling in Slack message collection tests.
- **`7a0591ef8`** + **`186dc0363`** + **`e24d02308`** — WhatsApp: allow media-only sends, default MIME type for voice messages, convert Markdown formatting to WhatsApp formatting. New `src/markdown/whatsapp.ts` module.
- **`cf6e8e18d`** + **`a028c0512`** + **`3d771afe7`** + **`8fdb2e64a`** + **`7ca8d936d`** + **`18610a587`** — Feishu: preserve doc block order, use resolved account in status probe, tighten mention trigger matching, buffer upload path, remove workspace dev dependency, refresh lockfile.
- **`36e27ad56`** + **`6d9d4d04e`** — Memory/QMD: add configurable search mode with query/search/vsearch support and fallback on unsupported flags.
- **`dd6047d99`** — fix(cron): prevent duplicate fires when multiple jobs trigger simultaneously.
- **`c28cbac51`** — CI: add PR size autolabel workflow.

**Line number shifts in this sync:** `qmd-manager.ts` +22 lines at 260-275 (search mode refactor + error fallback), +12 at 975 (new `isUnsupportedQmdOptionError()`), +11 at 1010 (new `buildSearchArgs()`). Old 324-329→346-352 (.md validation), old 972-978→1006-1012 (`buildCollectionFilterArgs`). `backend-config.ts` +10 (new `searchMode` field + resolver). Old 223-242→233-252 (`resolveDefaultCollections`). `cron/isolated-agent/run.ts` +3 at 124 (agentId comments). Old 310-315→303-310 (suspicious patterns, already stale from prior sync), old 321-327→316-322 (`buildSafeExternalPrompt`). `sessions/store.ts` -1 (removed JSON5 import). Old 209-211→208-210. All references updated in documentation.

**CVE status:** 5 published advisories — all pre-existing, none patched in this merge.

**Gap status: 1 closed, 3 remain open** (pipe-delimited token format, outPath validation, bootstrap/memory .md scanning — unchanged).
