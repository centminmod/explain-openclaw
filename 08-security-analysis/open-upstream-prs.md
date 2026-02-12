> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Open PRs](./open-upstream-prs.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## Open Upstream Security Pull Requests

> **Status:** These PRs in upstream openclaw/openclaw fix or harden security-related code. Monitor merge status and sync locally when merged.
>
> **Last checked:** 13-02-2026 (09:16 AEST)

### OPEN/DRAFT PRs (monitor for merge)

| PR | Status | Category | Summary | Related Issue | Local Impact |
|----|--------|----------|---------|---------------|--------------|
| [#8513](https://github.com/openclaw/openclaw/pull/8513) | OPEN | security-fix | Require auth for plugin HTTP routes | [#8512](https://github.com/openclaw/openclaw/issues/8512) | OPEN/PENDING |
| [#11435](https://github.com/openclaw/openclaw/pull/11435) | OPEN | security-fix | Validate `OPENCLAW_BROWSER_CONTROL_MODULE` before dynamic import | [#11437](https://github.com/openclaw/openclaw/issues/11437) | OPEN/PENDING |
| [#11432](https://github.com/openclaw/openclaw/pull/11432) | OPEN | hardening | Add `--ignore-scripts` to npm install in hook/plugin installers | [#11434](https://github.com/openclaw/openclaw/issues/11434) | OPEN/PENDING |
| [#11439](https://github.com/openclaw/openclaw/pull/11439) | OPEN | hardening | Warn on relative `OPENCLAW_CONFIG_PATH` + disable config-origin plugin auto-enable | [#11431](https://github.com/openclaw/openclaw/issues/11431) | OPEN/PENDING |
| [#13275](https://github.com/openclaw/openclaw/pull/13275) | OPEN | security-fix | SSRF guard bypassed by IPv4-compatible IPv6 addresses (`::127.0.0.1`) | — | OPEN/PENDING |
| [#13198](https://github.com/openclaw/openclaw/pull/13198) | OPEN | security-fix | External content marker sanitization bypassed via Unicode homoglyph | — | OPEN/PENDING |
| [#13183](https://github.com/openclaw/openclaw/pull/13183) | OPEN | security-fix | Use `execFileSync` instead of `execSync` to prevent shell injection in daemon | — | OPEN/PENDING |
| [#13170](https://github.com/openclaw/openclaw/pull/13170) | OPEN | hardening | Enforce `webhookSecret` at runtime for Telegram webhook mode | [#6606](https://github.com/openclaw/openclaw/issues/6606) | OPEN/PENDING |
| [#13117](https://github.com/openclaw/openclaw/pull/13117) | OPEN | security-fix | Telegram webhook allows unauthenticated update injection | [#6606](https://github.com/openclaw/openclaw/issues/6606) | OPEN/PENDING |
| [#13012](https://github.com/openclaw/openclaw/pull/13012) | OPEN | hardening | Detect invisible Unicode in skills and plugins (ASCII smuggling, Trojan Source) | — | OPEN/PENDING |
| [#12387](https://github.com/openclaw/openclaw/pull/12387) | OPEN | security-fix | Fix SSRF vulnerability in matrix-bot-sdk | — | OPEN/PENDING |
| [#12351](https://github.com/openclaw/openclaw/pull/12351) | OPEN | hardening | Harden A2A announce target resolution + DNS rebinding defense | — | OPEN/PENDING |
| [#11742](https://github.com/openclaw/openclaw/pull/11742) | OPEN | security-fix | Remove loopback address auth bypass in BlueBubbles webhook handler | — | OPEN/PENDING |
| [#11169](https://github.com/openclaw/openclaw/pull/11169) | OPEN | security-fix | Remove bundled soul-evil hook that enables silent agent hijacking | [#8776](https://github.com/openclaw/openclaw/issues/8776) | OPEN/PENDING |
| [#11152](https://github.com/openclaw/openclaw/pull/11152) | OPEN | security-fix | Add `place_id` validation to prevent path traversal SSRF | — | OPEN/PENDING |
| [#11054](https://github.com/openclaw/openclaw/pull/11054) | OPEN | security-fix | Add auth token to sandbox browser bridge | [#6609](https://github.com/openclaw/openclaw/issues/6609) | OPEN/PENDING |
| [#10257](https://github.com/openclaw/openclaw/pull/10257) | OPEN | security-fix | Anchor MIME sanitization regex and block fullwidth bypass | — | OPEN/PENDING |
| [#10238](https://github.com/openclaw/openclaw/pull/10238) | OPEN | security-fix | Fix TwiML injection via unescaped locale/language/voice parameters | — | OPEN/PENDING |
| [#9529](https://github.com/openclaw/openclaw/pull/9529) | OPEN | security-fix | Validate archive entries against path traversal (Zip Slip) | [#3277](https://github.com/openclaw/openclaw/issues/3277) | OPEN/PENDING |
| [#9513](https://github.com/openclaw/openclaw/pull/9513) | OPEN | security-fix | Skill download archive path traversal checks | [#9512](https://github.com/openclaw/openclaw/issues/9512) | OPEN/PENDING |
| [#8757](https://github.com/openclaw/openclaw/pull/8757) | OPEN | security-fix | Validate redirect destinations to prevent SSRF via open redirect (MS Teams) | — | OPEN/PENDING |
| [#8718](https://github.com/openclaw/openclaw/pull/8718) | OPEN | security-fix | Sanitize download filenames to prevent path traversal (CWE-22) | [#8696](https://github.com/openclaw/openclaw/issues/8696) | OPEN/PENDING |
| [#8604](https://github.com/openclaw/openclaw/pull/8604) | OPEN | security-fix | Unauthenticated Nostr profile endpoints allow remote profile takeover | — | OPEN/PENDING |
| [#8305](https://github.com/openclaw/openclaw/pull/8305) | OPEN | hardening | Add SSRF protection to browser navigation | — | OPEN/PENDING |
| [#8186](https://github.com/openclaw/openclaw/pull/8186) | OPEN | security-fix | Validate sandbox `setupCommand` to prevent shell injection | — | OPEN/PENDING |
| [#7616](https://github.com/openclaw/openclaw/pull/7616) | OPEN | security-fix | Harden zip extraction against path traversal | [#3277](https://github.com/openclaw/openclaw/issues/3277) | OPEN/PENDING |
| [#5401](https://github.com/openclaw/openclaw/pull/5401) | OPEN | security-fix | Detect audio binary by magic bytes to prevent context injection | — | OPEN/PENDING |
| [#2580](https://github.com/openclaw/openclaw/pull/2580) | OPEN | security-fix | SSRF, path traversal, shell injection, and rate limiting protections | — | OPEN/PENDING |
| [#2544](https://github.com/openclaw/openclaw/pull/2544) | OPEN | security-fix | XSS vulnerability in Canvas Host | — | OPEN/PENDING |
| [#14689](https://github.com/openclaw/openclaw/pull/14689) | OPEN | hardening | Auto-set `per-channel-peer` dmScope default during multi-channel onboarding | [#14688](https://github.com/openclaw/openclaw/issues/14688) | OPEN/PENDING |
| [#14222](https://github.com/openclaw/openclaw/pull/14222) | OPEN | hardening | Add `needsApproval` to `before_tool_call` hook; move AgentShield to extension | — | OPEN/PENDING |
| [#14843](https://github.com/openclaw/openclaw/pull/14843) | OPEN | security-fix | Strip `apiKey` from `models.json` cache to prevent credential exposure | [#14808](https://github.com/openclaw/openclaw/issues/14808) | OPEN/PENDING |
| [#14665](https://github.com/openclaw/openclaw/pull/14665) | OPEN | security-fix | Handle additional Unicode angle bracket homoglyphs in content sanitization | — | OPEN/PENDING |
| [#14662](https://github.com/openclaw/openclaw/pull/14662) | OPEN | security-fix | Escape `<` in inline JSON to prevent script tag XSS | — | OPEN/PENDING |
| [#14814](https://github.com/openclaw/openclaw/pull/14814) | OPEN | hardening | Use timing-safe comparison for hook token auth | — | OPEN/PENDING |
| [#14661](https://github.com/openclaw/openclaw/pull/14661) | OPEN | hardening | Restrict canvas IP-based auth to private networks | — | OPEN/PENDING |
| [#14655](https://github.com/openclaw/openclaw/pull/14655) | OPEN | hardening | Allow trusted clients to request unredacted config via `includeSecrets` | [#14586](https://github.com/openclaw/openclaw/issues/14586) | OPEN/PENDING |
| [#14197](https://github.com/openclaw/openclaw/pull/14197) | OPEN | security-fix | Harden browser API auth, token comparisons (7 locations), and hook tokens | — | OPEN/PENDING |
| [#14098](https://github.com/openclaw/openclaw/pull/14098) | OPEN | hardening | Sanitize JSON tool-call payload text to prevent leak via Ollama/local providers | — | OPEN/PENDING |
| [#14061](https://github.com/openclaw/openclaw/pull/14061) | OPEN | security-fix | Docker gateway auth bypass via Host header spoofing — verify client IP matches Docker gateway | — | OPEN/PENDING |
| [#14512](https://github.com/openclaw/openclaw/pull/14512) | CLOSED | hardening | Allow Docker bridge connections to extension relay via `OPENCLAW_RELAY_ALLOW_PRIVATE` (closed: author banned) | [#14433](https://github.com/openclaw/openclaw/issues/14433) | NOT AFFECTED |
| [#14350](https://github.com/openclaw/openclaw/pull/14350) | OPEN | hardening | Add `--harden` CLI flag for security-hardened gateway mode (loopback bind, token auth, no Tailscale) | — | OPEN/PENDING |
| [#14318](https://github.com/openclaw/openclaw/pull/14318) | OPEN | hardening | Enforce outbound allowlist on Discord send functions (blocks agent writes to non-allowed channels) | — | OPEN/PENDING |
| [#14224](https://github.com/openclaw/openclaw/pull/14224) | OPEN | hardening | Telegram member-info action exposes chat administrators (admin enumeration) | — | OPEN/PENDING |
| [#14112](https://github.com/openclaw/openclaw/pull/14112) | OPEN | hardening | Test verifying `--ignore-scripts` blocks postinstall execution in plugin archive install | [#13132](https://github.com/openclaw/openclaw/issues/13132) | OPEN/PENDING |
| [#13894](https://github.com/openclaw/openclaw/pull/13894) | OPEN | hardening | Add manifest scanner for SKILL.md trust analysis (8 threat categories) | — | OPEN/PENDING |
| [#13876](https://github.com/openclaw/openclaw/pull/13876) | OPEN | hardening | Auth & security enhancements — CLI sync, guard models, config protection | [#13196](https://github.com/openclaw/openclaw/issues/13196), [#13236](https://github.com/openclaw/openclaw/issues/13236) | OPEN/PENDING |
| [#13817](https://github.com/openclaw/openclaw/pull/13817) | DRAFT | hardening | Configurable prompt injection monitor for tool results | — | OPEN/PENDING |
| [#13777](https://github.com/openclaw/openclaw/pull/13777) | OPEN | security-fix | Redact sensitive values in CLI `config get` output | [#13683](https://github.com/openclaw/openclaw/issues/13683) | OPEN/PENDING |
| [#13767](https://github.com/openclaw/openclaw/pull/13767) | OPEN | security-fix | Reject literal "undefined" and "null" gateway auth tokens | [#13756](https://github.com/openclaw/openclaw/issues/13756) | OPEN/PENDING |
| [#13737](https://github.com/openclaw/openclaw/pull/13737) | OPEN | hardening | Docker UID/GID remap hardening and docker-setup privilege isolation | — | OPEN/PENDING |
| [#13680](https://github.com/openclaw/openclaw/pull/13680) | OPEN | hardening | Add per-IP rate limiting to gateway authentication (CWE-307) | — | OPEN/PENDING |
| [#13521](https://github.com/openclaw/openclaw/pull/13521) | OPEN | security-fix | Require webhook secret in Telegram runtime webhook mode | [#13116](https://github.com/openclaw/openclaw/issues/13116) | OPEN/PENDING |
| [#13474](https://github.com/openclaw/openclaw/pull/13474) | OPEN | hardening | Distinguish webhooks from internal hooks in audit summary | [#13466](https://github.com/openclaw/openclaw/issues/13466) | OPEN/PENDING |
| [#13321](https://github.com/openclaw/openclaw/pull/13321) | OPEN | hardening | Android gateway device identity hardening and A2UI UX improvements | — | OPEN/PENDING |
| [#13308](https://github.com/openclaw/openclaw/pull/13308) | OPEN | hardening | Address audit findings (gateway, CI, Docker) | — | OPEN/PENDING |
| [#13290](https://github.com/openclaw/openclaw/pull/13290) | OPEN | docs | Warn against storing secrets in injected workspace files (TOOLS.md enters model context) | — | OPEN/PENDING |
| [#13293](https://github.com/openclaw/openclaw/pull/13293) | OPEN | hardening | Block tainted sink calls from untrusted tool outputs | — | OPEN/PENDING |
| [#13254](https://github.com/openclaw/openclaw/pull/13254) | OPEN | hardening | Harden archive extraction and plugin update rollback | — | OPEN/PENDING |
| [#13185](https://github.com/openclaw/openclaw/pull/13185) | OPEN | hardening | Sanitize error responses to prevent information leakage | — | OPEN/PENDING |
| [#13184](https://github.com/openclaw/openclaw/pull/13184) | OPEN | hardening | Default standalone servers to loopback bind | — | OPEN/PENDING |
| [#13169](https://github.com/openclaw/openclaw/pull/13169) | OPEN | hardening | Add `--ignore-scripts` to npm install during plugin/hook installation | — | OPEN/PENDING |
| [#13144](https://github.com/openclaw/openclaw/pull/13144) | OPEN | hardening | Harden archive extraction, auth tokens, hook transforms, and queue limits | — | OPEN/PENDING |
| [#13129](https://github.com/openclaw/openclaw/pull/13129) | OPEN | hardening | Clarify dmScope remediation with explicit `openclaw config set` CLI command | — | OPEN/PENDING |
| [#13090](https://github.com/openclaw/openclaw/pull/13090) | OPEN | security-fix | Device-pair extension leaks gateway credentials in chat messages | — | OPEN/PENDING |
| [#13073](https://github.com/openclaw/openclaw/pull/13073) | OPEN | security-fix | Finish credential redaction that was merged unfinished | — | OPEN/PENDING |
| [#13042](https://github.com/openclaw/openclaw/pull/13042) | OPEN | hardening | Add guard model for prompt injection sanitization | — | OPEN/PENDING |
| [#12958](https://github.com/openclaw/openclaw/pull/12958) | OPEN | hardening | Block agent read access to sensitive config and credential files | — | OPEN/PENDING |
| [#12871](https://github.com/openclaw/openclaw/pull/12871) | OPEN | hardening | Shell injection warning + prefer bash for embedded agents | — | OPEN/PENDING |
| [#12839](https://github.com/openclaw/openclaw/pull/12839) | OPEN | hardening | Add vault proxy mode for credential isolation | — | OPEN/PENDING |
| [#12260](https://github.com/openclaw/openclaw/pull/12260) | OPEN | hardening | Redact secrets in tool results before persisting to session transcript | [#5995](https://github.com/openclaw/openclaw/issues/5995) | OPEN/PENDING |
| [#12174](https://github.com/openclaw/openclaw/pull/12174) | OPEN | hardening | Add path containment check in `apply_patch` for non-sandboxed mode | — | OPEN/PENDING |
| [#11119](https://github.com/openclaw/openclaw/pull/11119) | DRAFT | enhancement | Prompt injection defense: instruction signing and model `verify` gates | — | OPEN/PENDING |

### MERGED PRs (sync status verified)

| PR | Status | Category | Summary | Related Issue | Local Impact |
|----|--------|----------|---------|---------------|--------------|
| [#1795](https://github.com/openclaw/openclaw/pull/1795) | MERGED | security-fix | Prevent auth bypass when behind unconfigured reverse proxy | — | ALREADY SYNCED |
| [#2016](https://github.com/openclaw/openclaw/pull/2016) | MERGED | enhancement | Add gateway network exposure check to `openclaw doctor` | [#2015](https://github.com/openclaw/openclaw/issues/2015) | ALREADY SYNCED |
| [#2568](https://github.com/openclaw/openclaw/pull/2568) | MERGED | docs | Add formal verification page | — | ALREADY SYNCED |
| [#4880](https://github.com/openclaw/openclaw/pull/4880) | MERGED | security-fix | Restrict local path extraction in media parser to prevent LFI | — | ALREADY SYNCED |
| [#7872](https://github.com/openclaw/openclaw/pull/7872) | MERGED | docs | Document secure DM mode preset | — | ALREADY SYNCED |
| [#8241](https://github.com/openclaw/openclaw/pull/8241) | MERGED | hardening | Matrix thread session isolation | — | ALREADY SYNCED |
| [#9377](https://github.com/openclaw/openclaw/pull/9377) | MERGED | docs | Improve DM security guidance with concrete example | — | ALREADY SYNCED |
| [#9436](https://github.com/openclaw/openclaw/pull/9436) | MERGED | security-fix | Remove query token acceptance from gateway hooks | [#9435](https://github.com/openclaw/openclaw/issues/9435), [#5120](https://github.com/openclaw/openclaw/issues/5120) | SYNC NEEDED |
| [#9518](https://github.com/openclaw/openclaw/pull/9518) | MERGED | security-fix | Add `authorizeCanvasRequest()` for canvas/A2UI auth | [#9517](https://github.com/openclaw/openclaw/issues/9517) | SYNC NEEDED |
| [#11093](https://github.com/openclaw/openclaw/pull/11093) | MERGED | security-fix | Add `sanitizeFilename()` to BlueBubbles attachments | [#10333](https://github.com/openclaw/openclaw/issues/10333) | SYNC NEEDED |
| [#13182](https://github.com/openclaw/openclaw/pull/13182) | MERGED | hardening | Split oversized security audit files using dot-naming convention | — | ALREADY SYNCED |
| [#13787](https://github.com/openclaw/openclaw/pull/13787) | MERGED | security-fix | BlueBubbles webhook auth bypass via loopback proxy trust (CVSS 8.6) | [#13786](https://github.com/openclaw/openclaw/issues/13786) | SYNC NEEDED |
| [#14029](https://github.com/openclaw/openclaw/pull/14029) | MERGED | security-fix | Pass Twilio stream auth token via `<Parameter>` instead of query string | — | ALREADY SYNCED |
| [#14218](https://github.com/openclaw/openclaw/pull/14218) | MERGED | security-fix | Antigravity thinking signature sanitization bypass via orphaned user-message repair | [#13765](https://github.com/openclaw/openclaw/issues/13765) | SYNC NEEDED |
| [#14659](https://github.com/openclaw/openclaw/pull/14659) | MERGED | hardening | Add `--ignore-scripts` to skills install commands | — | ALREADY SYNCED |

**Total:** 88 tracked PRs (15 merged, 70 open, 2 draft, 1 closed)

### Cross-Reference: PRs and Tracked Issues

| PR # | Fixes Issue(s) | Issue Severity | PR Status | Notes |
|------|---------------|----------------|-----------|-------|
| [#1795](https://github.com/openclaw/openclaw/pull/1795) | (unconfigured proxy bypass) | HIGH | MERGED | Fail-secure proxy detection in `isLocalDirectRequest()` at `src/gateway/auth.ts:93-113` |
| [#2016](https://github.com/openclaw/openclaw/pull/2016) | [#2015](https://github.com/openclaw/openclaw/issues/2015) | HIGH | MERGED | `noteSecurityWarnings()` at `src/commands/doctor-security.ts:11` checks gateway bind + auth |
| [#4880](https://github.com/openclaw/openclaw/pull/4880) | (LFI via MEDIA tokens) | HIGH | MERGED | `isValidMedia()` at `src/media/parse.ts:36-64` accepts all path types; LFI guard moved to `assertLocalMediaAllowed()` at `src/web/media.ts:42-69` |
| [#8241](https://github.com/openclaw/openclaw/pull/8241) | (Matrix thread isolation) | MEDIUM | MERGED | `:thread:${threadRootId}` suffix at `extensions/matrix/src/matrix/monitor/handler.ts:446-447` |
| [#8513](https://github.com/openclaw/openclaw/pull/8513) | [#8512](https://github.com/openclaw/openclaw/issues/8512) (CRITICAL) | CRITICAL | OPEN | Adds auth requirement for plugin HTTP routes in gateway |
| [#9436](https://github.com/openclaw/openclaw/pull/9436) | [#9435](https://github.com/openclaw/openclaw/issues/9435) (HIGH), [#5120](https://github.com/openclaw/openclaw/issues/5120) (MEDIUM) | HIGH | MERGED | Query token acceptance removed from `extractHookToken()` in `src/gateway/hooks.ts`; server returns HTTP 400 when `?token=` present |
| [#9518](https://github.com/openclaw/openclaw/pull/9518) | [#9517](https://github.com/openclaw/openclaw/issues/9517) (HIGH) | HIGH | MERGED | New `authorizeCanvasRequest()` at `src/gateway/server-http.ts:96-130` wraps canvas/A2UI endpoints |
| [#9529](https://github.com/openclaw/openclaw/pull/9529) | [#3277](https://github.com/openclaw/openclaw/issues/3277) (HIGH) | HIGH | OPEN | Validates archive entries against Zip Slip path traversal |
| [#9513](https://github.com/openclaw/openclaw/pull/9513) | [#9512](https://github.com/openclaw/openclaw/issues/9512) (HIGH) | HIGH | OPEN | Adds path traversal checks to skill download archive extraction |
| [#11054](https://github.com/openclaw/openclaw/pull/11054) | [#6609](https://github.com/openclaw/openclaw/issues/6609) (HIGH) | HIGH | OPEN | Adds auth token to sandbox browser bridge |
| [#11093](https://github.com/openclaw/openclaw/pull/11093) | [#10333](https://github.com/openclaw/openclaw/issues/10333) (MEDIUM) | MEDIUM | MERGED | `sanitizeFilename()` at `extensions/bluebubbles/src/attachments.ts:26-30` strips dangerous chars from Content-Disposition filenames |
| [#11169](https://github.com/openclaw/openclaw/pull/11169) | [#8776](https://github.com/openclaw/openclaw/issues/8776) (HIGH) | HIGH | OPEN | Removes bundled soul-evil hook that enables silent agent hijacking |
| [#11435](https://github.com/openclaw/openclaw/pull/11435) | [#11437](https://github.com/openclaw/openclaw/issues/11437) (CRITICAL) | CRITICAL | OPEN | Validates `OPENCLAW_BROWSER_CONTROL_MODULE` env var before dynamic `import()` |
| [#11432](https://github.com/openclaw/openclaw/pull/11432) | [#11434](https://github.com/openclaw/openclaw/issues/11434) (HIGH) | HIGH | OPEN | Adds `--ignore-scripts` to npm install in hook and plugin installers |
| [#11439](https://github.com/openclaw/openclaw/pull/11439) | [#11431](https://github.com/openclaw/openclaw/issues/11431) (HIGH) | HIGH | OPEN | Warns on relative `OPENCLAW_CONFIG_PATH` and disables config-origin plugin auto-enable |
| [#12260](https://github.com/openclaw/openclaw/pull/12260) | [#5995](https://github.com/openclaw/openclaw/issues/5995) (HIGH) | HIGH | OPEN | Redacts secrets from tool results before persisting to session transcripts |
| [#13117](https://github.com/openclaw/openclaw/pull/13117) | [#6606](https://github.com/openclaw/openclaw/issues/6606) (HIGH) | HIGH | OPEN | Enforces webhook authentication for Telegram webhook mode |
| [#13170](https://github.com/openclaw/openclaw/pull/13170) | [#6606](https://github.com/openclaw/openclaw/issues/6606) (HIGH) | HIGH | OPEN | Enforces `webhookSecret` at runtime for Telegram webhook mode |
| [#13182](https://github.com/openclaw/openclaw/pull/13182) | (refactor) | LOW | MERGED | Barrel re-export at `src/security/audit-extra.ts` → `.sync.ts` + `.async.ts` |
| [#13680](https://github.com/openclaw/openclaw/pull/13680) | (CWE-307 brute force) | MEDIUM | OPEN | Per-IP rate limiting for gateway auth — no limiter exists locally in `src/gateway/auth.ts` |
| [#13521](https://github.com/openclaw/openclaw/pull/13521) | [#13116](https://github.com/openclaw/openclaw/issues/13116) (HIGH) | HIGH | OPEN | Fail-close webhook secret enforcement; related to #13170 and #13117 |
| [#13474](https://github.com/openclaw/openclaw/pull/13474) | [#13466](https://github.com/openclaw/openclaw/issues/13466) (LOW) | LOW | OPEN | Split `hooks:` → `hooks.webhooks:` + `hooks.internal:` at `audit-extra.sync.ts:294-317` |
| [#8718](https://github.com/openclaw/openclaw/pull/8718) | [#8696](https://github.com/openclaw/openclaw/issues/8696) (HIGH) | HIGH | OPEN | Sanitizes download filenames to prevent path traversal (CWE-22) |
| [#13787](https://github.com/openclaw/openclaw/pull/13787) | [#13786](https://github.com/openclaw/openclaw/issues/13786) (HIGH) | HIGH | MERGED | Loopback bypass in BlueBubbles webhook auth (CVSS 8.6); related to #11742; merged 2026-02-12 |
| [#13777](https://github.com/openclaw/openclaw/pull/13777) | [#13683](https://github.com/openclaw/openclaw/issues/13683) (HIGH) | HIGH | OPEN | Sandboxed agent credential exfil via `openclaw config get` — no redaction in CLI path |
| [#13767](https://github.com/openclaw/openclaw/pull/13767) | [#13756](https://github.com/openclaw/openclaw/issues/13756) (MEDIUM) | MEDIUM | OPEN | `normalizeGatewayTokenInput()` accepts "undefined"/"null" strings as valid tokens |
| [#13876](https://github.com/openclaw/openclaw/pull/13876) | [#13196](https://github.com/openclaw/openclaw/issues/13196), [#13236](https://github.com/openclaw/openclaw/issues/13236) | HIGH | OPEN | CLI credential sync + config redaction regex fix + guard model |
| [#14029](https://github.com/openclaw/openclaw/pull/14029) | (token leakage in URL) | MEDIUM | MERGED | Twilio stream auth token via `<Parameter>` instead of query string; merged 2026-02-12; ALREADY SYNCED locally |
| [#14222](https://github.com/openclaw/openclaw/pull/14222) | (maintainer feedback on #8727) | MEDIUM | OPEN | Core `before_tool_call` hook extended with `needsApproval`; AgentShield moved to extension with encrypted approval store |
| [#14197](https://github.com/openclaw/openclaw/pull/14197) | (Codex CLI audit) | MEDIUM | OPEN | Shared `safeEqual()`, browser API auth (30+ unauthenticated endpoints), 7 timing-unsafe token comparisons, `hooks.allowQueryToken` deprecation |
| [#14061](https://github.com/openclaw/openclaw/pull/14061) | (Docker auth bypass) | HIGH | OPEN | Container on same Docker network can spoof `Host: localhost` to bypass `isLocalDirectRequest()` — adds `readDockerGatewayIp()` client IP verification |
| [#14218](https://github.com/openclaw/openclaw/pull/14218) | [#13765](https://github.com/openclaw/openclaw/issues/13765) | LOW | MERGED | Thinking block sanitization bypass via orphaned user-message repair path in `attempt.ts`; merged 2026-02-12 |
| [#14843](https://github.com/openclaw/openclaw/pull/14843) | [#14808](https://github.com/openclaw/openclaw/issues/14808) | MEDIUM | OPEN | `apiKey` credential exposure in `models.json` cache; local `models-config.ts:130` has same pattern |
| [#14665](https://github.com/openclaw/openclaw/pull/14665) | (homoglyph bypass) | MEDIUM | OPEN | Additional Unicode angle bracket homoglyphs bypass `foldMarkerChar()` at `external-content.ts:89-107` |
| [#14662](https://github.com/openclaw/openclaw/pull/14662) | (XSS in control UI) | MEDIUM | OPEN | `JSON.stringify` in `<script>` tag at `control-ui.ts:173-179` does not escape `<` |
| [#14661](https://github.com/openclaw/openclaw/pull/14661) | (canvas IP auth) | MEDIUM | OPEN | Canvas IP-based auth fallback at `server-http.ts:129` has no private network restriction; Greptile flags IPv6 `fe80::/10` misclass |
| [#14655](https://github.com/openclaw/openclaw/pull/14655) | [#14586](https://github.com/openclaw/openclaw/issues/14586) | LOW | OPEN | `includeSecrets` for config.get; Greptile flags Control UI client also gets access |
| [#14814](https://github.com/openclaw/openclaw/pull/14814) | (timing attack) | LOW | OPEN | Hook token at `server-http.ts:164` uses `!==` while `auth.ts:256` uses `safeEqual()` |
| [#14098](https://github.com/openclaw/openclaw/pull/14098) | (tool-call JSON leak) | LOW | OPEN | `stripJsonToolCallText()` prevents Ollama/local providers from leaking raw tool-call JSON to user surfaces |
| [#14350](https://github.com/openclaw/openclaw/pull/14350) | (security hardening CLI) | MEDIUM | OPEN | `--harden` flag forces loopback bind + token auth + no Tailscale; no equivalent exists locally |
| [#14318](https://github.com/openclaw/openclaw/pull/14318) | (outbound channel control) | MEDIUM | OPEN | `enforceOutboundAllowlist()` blocks Discord sends to non-allowed channels; local `send.outbound.ts` has zero guards |
| [#14689](https://github.com/openclaw/openclaw/pull/14689) | [#14688](https://github.com/openclaw/openclaw/issues/14688) | LOW | OPEN | Auto-set `per-channel-peer` dmScope during multi-channel onboarding; Greptile flags multi-account channels need `per-account-channel-peer` |
| [#14512](https://github.com/openclaw/openclaw/pull/14512) | [#14433](https://github.com/openclaw/openclaw/issues/14433) | LOW | CLOSED | Author banned; `isPrivateNetworkAddress()` for Docker bridge relay needs re-implementation |
| [#14224](https://github.com/openclaw/openclaw/pull/14224) | (admin enumeration) | LOW | OPEN | Telegram `getChatAdministrators` action — gate inconsistency may default to enabled |
| [#14112](https://github.com/openclaw/openclaw/pull/14112) | [#13132](https://github.com/openclaw/openclaw/issues/13132) | MEDIUM | OPEN | Integration test verifying `--ignore-scripts` blocks postinstall in plugin archive install; related to #11432 |
| [#13737](https://github.com/openclaw/openclaw/pull/13737) | (Docker privilege hardening) | LOW | OPEN | UID/GID remap in Dockerfile; Greptile flags GID collision not detected (silently reuses existing group) |
| [#13290](https://github.com/openclaw/openclaw/pull/13290) | (openclaw/trust#1) | LOW | OPEN | Security warnings in TOOLS.md templates and system-prompt docs that workspace files enter model context |
| [#13129](https://github.com/openclaw/openclaw/pull/13129) | (dmScope UX) | LOW | OPEN | Uses `formatCliCommand()` for dmScope remediation; local `audit.ts:547` still uses plain string |

### #1795: Prevent Auth Bypass Behind Unconfigured Reverse Proxy

**PR Status:** MERGED (2026-01-25)
**Category:** security-fix
**Closes:** (addresses unconfigured reverse proxy auth bypass)

**Changes:**
- `src/gateway/server/ws-connection/message-handler.ts:203-204` — detects untrusted proxy headers
- `src/gateway/auth.ts:93-113` — `isLocalDirectRequest()` returns `false` when proxy headers present but `trustedProxies` not configured (fail-secure)
- `src/security/audit.ts` — audit detection for misconfigured proxy setup

**Local Impact:** ALREADY SYNCED — `isLocalDirectRequest()` at `src/gateway/auth.ts:106-113` checks `hasForwarded` and `remoteIsTrustedProxy`

### #2016: Add Gateway Network Exposure Check to Doctor

**PR Status:** MERGED (2026-01-26)
**Category:** enhancement
**Closes:** [#2015](https://github.com/openclaw/openclaw/issues/2015)

**Changes:**
- `src/commands/doctor-security.ts` — new `noteSecurityWarnings()` function
- CRITICAL warning if gateway bound to network without auth
- Actionable fix commands in warning messages

**Local Impact:** ALREADY SYNCED — `noteSecurityWarnings()` at `src/commands/doctor-security.ts:11` with full exposure check

### #4880: Restrict Local Path Extraction in Media Parser (Prevent LFI)

**PR Status:** MERGED (2026-01-31)
**Category:** security-fix
**Closes:** (fixes LFI via MEDIA: tokens allowing arbitrary file read)

**Changes:**
- `src/media/parse.ts:36-64` — `isValidMedia()` refactored to accept all local path types (absolute, tilde, traversal, Windows, bare filenames); security validation deferred to load layer
- `src/web/media.ts:42-69` — new `assertLocalMediaAllowed()` validates local paths against allowed directory roots (`/tmp`, `~/.openclaw/media`, `~/.openclaw/agents`), resolves symlinks
- `src/media/parse.test.ts` — test coverage updated for new path acceptance + load-time validation

**Local Impact:** ALREADY SYNCED — LFI defense now two-layer: `isValidMedia()` at `src/media/parse.ts:36-64` accepts paths, `assertLocalMediaAllowed()` at `src/web/media.ts:42-69` enforces directory root guards

### #8241: Matrix Thread Session Isolation

**PR Status:** MERGED (2026-02-10)
**Category:** hardening
**Closes:** (adds Matrix thread support with isolated sessions)

**Changes:**
- `extensions/matrix/src/matrix/monitor/handler.ts` — thread messages get `:thread:${threadRootId}` session key suffix

**Local Impact:** ALREADY SYNCED — thread isolation at `extensions/matrix/src/matrix/monitor/handler.ts:446-447`

### #13182: Split Oversized Security Audit Files

**PR Status:** MERGED (2026-02-10)
**Category:** hardening (refactor)

**Changes:**
- `src/security/audit-extra.ts` — converted to barrel re-export (31 lines)
- `src/security/audit-extra.sync.ts` — config-only checks (559 lines)
- `src/security/audit-extra.async.ts` — I/O-based checks (668 lines)

**Local Impact:** ALREADY SYNCED — barrel re-export at `src/security/audit-extra.ts:1-31`

### #9436: Remove Query Token Acceptance from Gateway Hooks

**PR Status:** MERGED
**Category:** security-fix
**Closes:** [#9435](https://github.com/openclaw/openclaw/issues/9435) (HIGH — gateway auth token in URL), [#5120](https://github.com/openclaw/openclaw/issues/5120) (MEDIUM — webhook token via query params)

**Changes:**
- `src/gateway/hooks.ts` — `extractHookToken()` no longer reads `url.searchParams` for token
- `src/gateway/server-http.ts:154-161` — returns HTTP 400 when `?token=` query parameter is present
- `src/commands/dashboard.ts` — no longer constructs `?token=` URLs
- `src/commands/onboard-helpers.ts` — no longer passes token in URL

**Local Impact:** SYNC NEEDED — local `extractHookToken()` may still accept query tokens

### #9518: Add Canvas/A2UI Authorization

**PR Status:** MERGED
**Category:** security-fix
**Closes:** [#9517](https://github.com/openclaw/openclaw/issues/9517) (HIGH — canvas host auth bypass)

**Changes:**
- `src/gateway/server-http.ts:96-130` — new `authorizeCanvasRequest()` function
- `src/gateway/server-http.ts:393-412` — canvas HTTP handler now auth-wrapped
- `src/gateway/server-http.ts:461` — canvas WebSocket upgrade now auth-wrapped

**Local Impact:** SYNC NEEDED — local canvas endpoints may still be unauthenticated

### #11093: BlueBubbles Filename Sanitization

**PR Status:** MERGED
**Category:** security-fix
**Closes:** [#10333](https://github.com/openclaw/openclaw/issues/10333) (MEDIUM — multipart header injection)

**Changes:**
- `extensions/bluebubbles/src/attachments.ts:26-30` — `sanitizeFilename()` now strips `"`, `\r`, `\n`, and other Content-Disposition-dangerous characters beyond just `path.basename()`

**Local Impact:** SYNC NEEDED — local `sanitizeFilename()` may still use only `path.basename()`

### #13787: BlueBubbles Webhook Auth Bypass via Loopback Proxy Trust

**PR Status:** MERGED (2026-02-12)
**Category:** security-fix
**Closes:** [#13786](https://github.com/openclaw/openclaw/issues/13786) (HIGH — CVSS 8.6 loopback bypass)

**Changes:**
- `extensions/bluebubbles/src/monitor.ts` — removes 4 lines of loopback trust bypass logic from webhook handler
- `extensions/bluebubbles/src/monitor.test.ts` — updated test coverage

**Local Impact:** SYNC NEEDED — local `extensions/bluebubbles/src/monitor.ts` may still have the loopback auth bypass path

### #14029: Pass Twilio Stream Auth Token via Parameter

**PR Status:** MERGED (2026-02-12)
**Category:** security-fix

**Changes:**
- `extensions/voice-call/src/providers/twilio.ts:431-440` — `getStreamConnectXml()` extracts token from URL, passes via TwiML `<Parameter>` instead of query string
- `extensions/voice-call/src/media-stream.ts:149-152` — reads token from `customParameters` with query string fallback

**Local Impact:** ALREADY SYNCED — local `twilio.ts:431-440` already uses `<Parameter>` approach; tests at `twilio.test.ts:34,59` verify

### #14218: Thinking Signature Sanitization Bypass

**PR Status:** MERGED (2026-02-12)
**Category:** security-fix
**Closes:** [#13765](https://github.com/openclaw/openclaw/issues/13765) (LOW — thinking block sanitization bypass)

**Changes:**
- `src/agents/pi-embedded-runner/run/attempt.ts` — fixes orphaned user-message repair path that could strip thinking block sanitization
- `src/agents/pi-embedded-runner/model.ts` — new opus 4.6 forward-compat model handling
- `src/agents/pi-embedded-runner/model.test.ts` — test coverage

**Local Impact:** SYNC NEEDED — local `attempt.ts:765-782` has the orphaned user-message repair path but may lack the sanitization fix

### #14659: Add --ignore-scripts to Skills Install Commands

**PR Status:** MERGED (2026-02-12)
**Category:** hardening

**Changes:**
- `src/agents/skills-install.ts` — adds `--ignore-scripts` to all package manager commands in `buildNodeInstallCommand()`

**Local Impact:** ALREADY SYNCED — `--ignore-scripts` already present at lines 150, 152, 154, 156 for pnpm/yarn/bun/npm
