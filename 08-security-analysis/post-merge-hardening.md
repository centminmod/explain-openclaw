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
- [Feb 2 sync 4](./post-merge-hardening/2026-02-02-sync-4.md)
- [Feb 2 sync 10](./post-merge-hardening/2026-02-02-sync-10.md)
- [Feb 3 sync 2](./post-merge-hardening/2026-02-03-sync-2.md)
- [Feb 3 sync 3](./post-merge-hardening/2026-02-03-sync-3.md)
- [Feb 4 sync 1](./post-merge-hardening/2026-02-04-sync-1.md)
- [Feb 4 sync 2](./post-merge-hardening/2026-02-04-sync-2.md)
- [Feb 4 sync 3](./post-merge-hardening/2026-02-04-sync-3.md)
- [Feb 5 sync 1 (Notes)](./post-merge-hardening/2026-02-05-sync-1.md)
- [Feb 5 sync 2](./post-merge-hardening/2026-02-05-sync-2.md)
- [Feb 5 sync 3](./post-merge-hardening/2026-02-05-sync-3.md)
- [Feb 6 sync 2](./post-merge-hardening/2026-02-06-sync-2.md)
- [Feb 6 sync 3](./post-merge-hardening/2026-02-06-sync-3.md)
- [Feb 6 sync 4](./post-merge-hardening/2026-02-06-sync-4.md)
- [Feb 7 sync 1](./post-merge-hardening/2026-02-07-sync-1.md)
- [Feb 7 sync 2](./post-merge-hardening/2026-02-07-sync-2.md)
- [Feb 7 sync 3](./post-merge-hardening/2026-02-07-sync-3.md)
- [Feb 9 sync 1](./post-merge-hardening/2026-02-09-sync-1.md)
- [Feb 9 sync 3](./post-merge-hardening/2026-02-09-sync-3.md)
- [Feb 10 sync 3 (26 commits)](./post-merge-hardening/2026-02-10-sync-3.md)
- [Feb 10 sync 5 (51 commits)](./post-merge-hardening/2026-02-10-sync-5.md)
- [Feb 10 sync 7 (6 commits)](./post-merge-hardening/2026-02-10-sync-7.md)
- [Feb 11 sync 1 (22 commits)](./post-merge-hardening/2026-02-11-sync-1.md)
- [Feb 11 sync 2 (17 commits)](./post-merge-hardening/2026-02-11-sync-2.md)
- [Feb 12 sync 1 (32 commits)](./post-merge-hardening/2026-02-12-sync-1.md)
- [Feb 12 sync 2 (Notes)](./post-merge-hardening/2026-02-12-sync-2.md)
- [Feb 12 sync 3 (8 commits)](./post-merge-hardening/2026-02-12-sync-3.md)
- [Feb 12 sync 5 (33 commits)](./post-merge-hardening/2026-02-12-sync-5.md)
- [Feb 13 sync 1 (35 commits)](./post-merge-hardening/2026-02-13-sync-1.md)
- [Feb 13 sync 2 (17 commits)](./post-merge-hardening/2026-02-13-sync-2.md)
- [Feb 13 sync 3 (26 commits)](./post-merge-hardening/2026-02-13-sync-3.md)
- [Feb 13 sync 4 (20 commits)](./post-merge-hardening/2026-02-13-sync-4.md)
- [Feb 13 sync 5 (35 commits)](./post-merge-hardening/2026-02-13-sync-5.md)
- [Feb 13 sync 6 (35 commits)](./post-merge-hardening/2026-02-13-sync-6.md)
- [Feb 14 sync 1 (16 commits)](./post-merge-hardening/2026-02-14-sync-1.md)
- [Feb 14 sync 2 (16 commits)](./post-merge-hardening/2026-02-14-sync-2.md)
- [Feb 14 sync 3 (35 commits)](./post-merge-hardening/2026-02-14-sync-3.md)
- [Feb 14 sync 4 (21 commits)](./post-merge-hardening/2026-02-14-sync-4.md)
- [Feb 14 sync 6 (33 commits)](./post-merge-hardening/2026-02-14-sync-6.md)
- [Feb 14 sync 7 (63 commits)](./post-merge-hardening/2026-02-14-sync-7.md)
- [Feb 14 sync 8 (5 commits)](./post-merge-hardening/2026-02-14-sync-8.md)
- [Feb 14 sync 9 (71 commits)](./post-merge-hardening/2026-02-14-sync-9.md)
- [Feb 14 sync 10 (48 commits)](./post-merge-hardening/2026-02-14-sync-10.md)
- [Feb 14 sync 11 (18 commits)](./post-merge-hardening/2026-02-14-sync-11.md)
- [Feb 15 sync 1 (38 commits)](./post-merge-hardening/2026-02-15-sync-1.md)
- [Feb 15 sync 2 (30 commits)](./post-merge-hardening/2026-02-15-sync-2.md)
- [Feb 15 sync 3 (30 commits)](./post-merge-hardening/2026-02-15-sync-3.md)
- [Feb 15 sync 4 (30 commits)](./post-merge-hardening/2026-02-15-sync-4.md)
- [Feb 15 sync 5 (30 commits)](./post-merge-hardening/2026-02-15-sync-5.md)
- [Feb 15 sync 6 (30 commits)](./post-merge-hardening/2026-02-15-sync-6.md)
- [Feb 15 sync 7 (30 commits)](./post-merge-hardening/2026-02-15-sync-7.md)
- [Feb 15 sync 8 (30 commits)](./post-merge-hardening/2026-02-15-sync-8.md)
- [Feb 15 sync 9 (30 commits)](./post-merge-hardening/2026-02-15-sync-9.md)
- [Feb 15 sync 10 (60 commits)](./post-merge-hardening/2026-02-15-sync-10.md)
- [Feb 15 sync 11 (80 commits)](./post-merge-hardening/2026-02-15-sync-11.md)
- [Feb 15 sync 12 (80 commits)](./post-merge-hardening/2026-02-15-sync-12.md)
- [Feb 15 sync 13 (94 commits)](./post-merge-hardening/2026-02-15-sync-13.md)
- [Feb 15 sync 14 (37 commits)](./post-merge-hardening/2026-02-15-sync-14.md)
- [Feb 15 sync 15 (101 commits)](./post-merge-hardening/2026-02-15-sync-15.md)
- [Feb 16 sync 1 (172 commits)](./post-merge-hardening/2026-02-16-sync-1.md)

## Post-Merge Security Hardening

> This section tracks security-relevant commits merged from upstream. Entries are added by the sync-explain-docs-with-upstream skill.

### Legitimate Gaps Status

Four defense-in-depth items were identified across audits:

1. ~~**Gateway-side env var blocklist:**~~ **CLOSED in PR #12.** Gateway now validates env vars via `DANGEROUS_HOST_ENV_VARS` blocklist (`src/agents/bash-tools.exec-runtime.ts:32-50`) and `validateHostEnv()` (`src/agents/bash-tools.exec-runtime.ts:54`, enforced at `src/agents/bash-tools.exec.ts:300`).
2. **Pipe-delimited token format:** RSA signing prevents exploitation, but a structured format (JSON) would be more robust against future changes.
3. **outPath validation in screen_record:** Accepts arbitrary paths without validation. Writes are confined to the paired node device, but path validation would add depth.
4. **Bootstrap/memory `.md` content scanning:** The built-in scanner (`src/security/skill-scanner.ts:37-46`) only scans JS/TS. Nine workspace bootstrap files are injected into the system prompt (20,000 chars each) via `loadWorkspaceBootstrapFiles()` (`src/agents/workspace.ts:400-454`) with no content validation. `memory/*.md` files are accessed via tool calls (4,000-char budget) through a separate pipeline (`src/memory/internal.ts:78-107`) also without content scanning. QMD memory path hardening validates `.md` extension and rejects symlinks (`src/memory/qmd-manager.ts:418-424`) but does not scan content. Subagent exposure is limited — `filterBootstrapFilesForSession()` (`src/agents/workspace.ts:458-466`) restricts subagents to `AGENTS.md` + `TOOLS.md` only. See [Cisco AI Defense gap analysis](./cisco-ai-defense-skill-scanner.md#beyond-skillmd-all-persistent-md-files-are-unscanned).

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

- **`0a5821a81`** + **`a87a07ec8`** — Strict environment variable validation (#4896) (thanks @HassanFleyah): `DANGEROUS_HOST_ENV_VARS` blocklist (`src/agents/bash-tools.exec-runtime.ts:32-50`) and `validateHostEnv()` (`src/agents/bash-tools.exec-runtime.ts:54`, enforced at `src/agents/bash-tools.exec.ts:300`) now block `LD_PRELOAD`, `DYLD_*`, `NODE_OPTIONS`, `PATH`, etc. on gateway host execution. **Closes Legitimate Gap #1.**

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
