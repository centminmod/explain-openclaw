> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Open PRs](./open-upstream-prs.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Hudson Rock](./hudson-rock-infostealer-analysis.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## Open Upstream Security Pull Requests

> **Status:** These PRs in upstream openclaw/openclaw fix or harden security-related code. Monitor merge status and sync locally when merged.
>
> **Last checked:** 16-02-2026 (02:49 AEST)

### OPEN/DRAFT PRs (monitor for merge)

| PR | Status | Category | Summary | Related Issue | Local Impact |
|----|--------|----------|---------|---------------|--------------|
| [#17182](https://github.com/openclaw/openclaw/pull/17182) | OPEN | security-fix | LINE webhook fail closed when token/secret are missing (fail-open risk) | [#17158](https://github.com/openclaw/openclaw/issues/17158) | OPEN/PENDING |
| [#16992](https://github.com/openclaw/openclaw/pull/16992) | OPEN | security-fix | Escape XML entities in file.filename to prevent prompt injection in OpenResponses | — | OPEN/PENDING |
| [#16990](https://github.com/openclaw/openclaw/pull/16990) | OPEN | security-fix | Strip auth headers on cross-origin redirect in downloadToFile (credential leakage) | — | OPEN/PENDING |
| [#16967](https://github.com/openclaw/openclaw/pull/16967) | OPEN | security-fix | OC-19: Sanitize CWD path injection in LLM system prompts (control char stripping) | — | OPEN/PENDING |
| [#16965](https://github.com/openclaw/openclaw/pull/16965) | OPEN | security-fix | Validate tool access against Docker bind mounts (workspace guard) | [#16379](https://github.com/openclaw/openclaw/issues/16379) | OPEN/PENDING |
| [#16963](https://github.com/openclaw/openclaw/pull/16963) | OPEN | hardening | Enable auth rate limiting by default (was opt-in only) | [#16876](https://github.com/openclaw/openclaw/issues/16876) | OPEN/PENDING |
| [#16961](https://github.com/openclaw/openclaw/pull/16961) | OPEN | docs | Warn against storing secrets in injected workspace files (supersedes closed #13290) | — | OPEN/PENDING |
| [#16959](https://github.com/openclaw/openclaw/pull/16959) | OPEN | security-fix | Prevent symlink and path traversal in skill packager (CWE-426; CVSS 7.7) | [#12536](https://github.com/openclaw/openclaw/issues/12536) | OPEN/PENDING |
| [#16958](https://github.com/openclaw/openclaw/pull/16958) | OPEN | security-fix | Escape user input in HTML gallery to prevent stored XSS (CWE-79; CVSS 7.5) | [#12538](https://github.com/openclaw/openclaw/issues/12538) | OPEN/PENDING |
| [#16947](https://github.com/openclaw/openclaw/pull/16947) | OPEN | security-fix | OC-13: Block dangerous Docker sandbox configs to prevent container escape (CWE-250; CVSS 8.2) | — | OPEN/PENDING |
| [#16936](https://github.com/openclaw/openclaw/pull/16936) | OPEN | security-fix | Feishu mention stripping vulnerable to regex injection (unsanitized RegExp construction) | — | OPEN/PENDING |
| [#16929](https://github.com/openclaw/openclaw/pull/16929) | OPEN | security-fix | Block access to sensitive directories (~/.openclaw/) from within sandbox | [#8588](https://github.com/openclaw/openclaw/issues/8588) | OPEN/PENDING |
| [#16928](https://github.com/openclaw/openclaw/pull/16928) | OPEN | security-fix | OC-07: Redact session history credentials and enforce Telegram webhook secret (CWE-209, CWE-346) | — | OPEN/PENDING |
| [#16907](https://github.com/openclaw/openclaw/pull/16907) | OPEN | hardening | Detect obfuscated commands (base64, hex, variable expansion) that bypass allowlist filters | [#8592](https://github.com/openclaw/openclaw/issues/8592) | OPEN/PENDING |
| [#16898](https://github.com/openclaw/openclaw/pull/16898) | OPEN | security-fix | Zalo webhook secret comparison vulnerable to timing attacks (plain `===`) | — | OPEN/PENDING |
| [#16036](https://github.com/openclaw/openclaw/pull/16036) | OPEN | hardening | Use 0o600 permissions for session transcript files (CWE-732; world-readable -> owner-only) | [#8751](https://github.com/openclaw/openclaw/issues/8751) | OPEN/PENDING |
| [#15757](https://github.com/openclaw/openclaw/pull/15757) | OPEN | hardening | Add hardening gap audit checks (sandbox.mode_not_all, tools.dangerous_not_denied, etc.) | — | OPEN/PENDING |
| [#15756](https://github.com/openclaw/openclaw/pull/15756) | OPEN | security-fix | Strip provider apiKey from models.json before prompt serialization (credential exposure) | — | OPEN/PENDING |
| [#15615](https://github.com/openclaw/openclaw/pull/15615) | OPEN | security-fix | Restrict PATH override to exact match in node-host sanitizeEnv | — | OPEN/PENDING |
| [#15379](https://github.com/openclaw/openclaw/pull/15379) | OPEN | hardening | Strip unsigned thinking blocks in agent loop via `transformContext` hook | [#13826](https://github.com/openclaw/openclaw/issues/13826) | OPEN/PENDING |
| [#15360](https://github.com/openclaw/openclaw/pull/15360) | OPEN | hardening | Prevent subagent announce from leaking internal prompts and stats | [#6669](https://github.com/openclaw/openclaw/issues/6669) | OPEN/PENDING |
| [#15296](https://github.com/openclaw/openclaw/pull/15296) | OPEN | hardening | Harden config redaction defaults; add explicit `--show-secrets` opt-in | — | OPEN/PENDING |
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
| [#11054](https://github.com/openclaw/openclaw/pull/11054) | OPEN | security-fix | Add auth token to sandbox browser bridge | [#6609](https://github.com/openclaw/openclaw/issues/6609) | OPEN/PENDING |
| [#10257](https://github.com/openclaw/openclaw/pull/10257) | OPEN | security-fix | Anchor MIME sanitization regex and block fullwidth bypass | — | OPEN/PENDING |
| [#10238](https://github.com/openclaw/openclaw/pull/10238) | OPEN | security-fix | Fix TwiML injection via unescaped locale/language/voice parameters | — | OPEN/PENDING |
| [#9529](https://github.com/openclaw/openclaw/pull/9529) | OPEN | security-fix | Validate archive entries against path traversal (Zip Slip) | [#3277](https://github.com/openclaw/openclaw/issues/3277) | OPEN/PENDING |
| [#8718](https://github.com/openclaw/openclaw/pull/8718) | OPEN | security-fix | Sanitize download filenames to prevent path traversal (CWE-22) | [#8696](https://github.com/openclaw/openclaw/issues/8696) | OPEN/PENDING |
| [#8305](https://github.com/openclaw/openclaw/pull/8305) | OPEN | hardening | Add SSRF protection to browser navigation | — | OPEN/PENDING |
| [#8186](https://github.com/openclaw/openclaw/pull/8186) | OPEN | security-fix | Validate sandbox `setupCommand` to prevent shell injection | — | OPEN/PENDING |
| [#7616](https://github.com/openclaw/openclaw/pull/7616) | OPEN | security-fix | Harden zip extraction against path traversal | [#3277](https://github.com/openclaw/openclaw/issues/3277) | OPEN/PENDING |
| [#5401](https://github.com/openclaw/openclaw/pull/5401) | OPEN | security-fix | Detect audio binary by magic bytes to prevent context injection | — | OPEN/PENDING |
| [#2580](https://github.com/openclaw/openclaw/pull/2580) | OPEN | security-fix | SSRF, path traversal, shell injection, and rate limiting protections | — | OPEN/PENDING |
| [#2544](https://github.com/openclaw/openclaw/pull/2544) | OPEN | security-fix | XSS vulnerability in Canvas Host | — | OPEN/PENDING |
| [#14689](https://github.com/openclaw/openclaw/pull/14689) | OPEN | hardening | Auto-set `per-channel-peer` dmScope default during multi-channel onboarding | [#14688](https://github.com/openclaw/openclaw/issues/14688) | OPEN/PENDING |
| [#14222](https://github.com/openclaw/openclaw/pull/14222) | OPEN | hardening | Add `needsApproval` to `before_tool_call` hook; move AgentShield to extension | — | OPEN/PENDING |
| [#3926](https://github.com/openclaw/openclaw/pull/3926) | OPEN | security-fix | Disable `browser.evaluateEnabled` by default (ACE via browser evaluate endpoint) | — | OPEN/PENDING |
| [#14662](https://github.com/openclaw/openclaw/pull/14662) | OPEN | security-fix | Escape `<` in inline JSON to prevent script tag XSS | — | OPEN/PENDING |
| [#14814](https://github.com/openclaw/openclaw/pull/14814) | OPEN | hardening | Use timing-safe comparison for hook token auth | — | OPEN/PENDING |
| [#14197](https://github.com/openclaw/openclaw/pull/14197) | OPEN | security-fix | Harden browser API auth, token comparisons (7 locations), and hook tokens | — | OPEN/PENDING |
| [#14098](https://github.com/openclaw/openclaw/pull/14098) | OPEN | hardening | Sanitize JSON tool-call payload text to prevent leak via Ollama/local providers | — | OPEN/PENDING |
| [#14061](https://github.com/openclaw/openclaw/pull/14061) | OPEN | security-fix | Docker gateway auth bypass via Host header spoofing — verify client IP matches Docker gateway | — | OPEN/PENDING |
| [#14318](https://github.com/openclaw/openclaw/pull/14318) | OPEN | hardening | Enforce outbound allowlist on Discord send functions (blocks agent writes to non-allowed channels) | — | OPEN/PENDING |
| [#14224](https://github.com/openclaw/openclaw/pull/14224) | OPEN | hardening | Telegram member-info action exposes chat administrators (admin enumeration) | — | OPEN/PENDING |
| [#14112](https://github.com/openclaw/openclaw/pull/14112) | OPEN | hardening | Test verifying `--ignore-scripts` blocks postinstall execution in plugin archive install | [#13132](https://github.com/openclaw/openclaw/issues/13132) | OPEN/PENDING |
| [#13894](https://github.com/openclaw/openclaw/pull/13894) | OPEN | hardening | Add manifest scanner for SKILL.md trust analysis (8 threat categories) | — | OPEN/PENDING |
| [#13876](https://github.com/openclaw/openclaw/pull/13876) | OPEN | hardening | Auth & security enhancements — CLI sync, guard models, config protection | [#13196](https://github.com/openclaw/openclaw/issues/13196), [#13236](https://github.com/openclaw/openclaw/issues/13236) | OPEN/PENDING |
| [#13817](https://github.com/openclaw/openclaw/pull/13817) | DRAFT | hardening | Configurable prompt injection monitor for tool results | — | OPEN/PENDING |
| [#13777](https://github.com/openclaw/openclaw/pull/13777) | OPEN | security-fix | Redact sensitive values in CLI `config get` output | [#13683](https://github.com/openclaw/openclaw/issues/13683) | OPEN/PENDING |
| [#13737](https://github.com/openclaw/openclaw/pull/13737) | OPEN | hardening | Docker UID/GID remap hardening and docker-setup privilege isolation | — | OPEN/PENDING |
| [#13521](https://github.com/openclaw/openclaw/pull/13521) | OPEN | security-fix | Require webhook secret in Telegram runtime webhook mode | [#13116](https://github.com/openclaw/openclaw/issues/13116) | OPEN/PENDING |
| [#13321](https://github.com/openclaw/openclaw/pull/13321) | OPEN | hardening | Android gateway device identity hardening and A2UI UX improvements | — | OPEN/PENDING |
| [#13308](https://github.com/openclaw/openclaw/pull/13308) | OPEN | hardening | Address audit findings (gateway, CI, Docker) | — | OPEN/PENDING |
| [#13254](https://github.com/openclaw/openclaw/pull/13254) | OPEN | hardening | Harden archive extraction and plugin update rollback | — | OPEN/PENDING |
| [#13169](https://github.com/openclaw/openclaw/pull/13169) | OPEN | hardening | Add `--ignore-scripts` to npm install during plugin/hook installation | — | OPEN/PENDING |
| [#13144](https://github.com/openclaw/openclaw/pull/13144) | OPEN | hardening | Harden archive extraction, auth tokens, hook transforms, and queue limits | — | OPEN/PENDING |
| [#13090](https://github.com/openclaw/openclaw/pull/13090) | OPEN | security-fix | Device-pair extension leaks gateway credentials in chat messages | — | OPEN/PENDING |
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
| [#15924](https://github.com/openclaw/openclaw/pull/15924) | MERGED | security-fix | OC-28: Prevent shell injection in macOS keychain credential write (CWE-78; execSync -> execFileSync; merged 2026-02-14) | — | ALREADY SYNCED |
| [#15940](https://github.com/openclaw/openclaw/pull/15940) | MERGED | hardening | Add trusted-proxy auth mode types and schema (delegated auth to reverse proxy; merged 2026-02-14) | [#1560](https://github.com/openclaw/openclaw/issues/1560) | ALREADY SYNCED |
| [#15848](https://github.com/openclaw/openclaw/pull/15848) | MERGED | hardening | Prune expired hook auth failure entries instead of clearing all rate-limit state (merged 2026-02-14; supersedes #15608) | — | ALREADY SYNCED |
| [#15652](https://github.com/openclaw/openclaw/pull/15652) | MERGED | hardening | Constrain browser trace/download output paths to OpenClaw temp roots (merged 2026-02-13) | — | ALREADY SYNCED |
| [#15604](https://github.com/openclaw/openclaw/pull/15604) | MERGED | security-fix | Block private/loopback/metadata IPs in link-understanding URL detection (SSRF hardening) | — | ALREADY SYNCED |
| [#15592](https://github.com/openclaw/openclaw/pull/15592) | MERGED | hardening | Sanitize and truncate untrusted WebSocket header values before logging (merged 2026-02-13) | — | ALREADY SYNCED |
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
| [#13474](https://github.com/openclaw/openclaw/pull/13474) | MERGED | hardening | Distinguish webhooks from internal hooks in audit summary | [#13466](https://github.com/openclaw/openclaw/issues/13466) | ALREADY SYNCED |
| [#14659](https://github.com/openclaw/openclaw/pull/14659) | MERGED | hardening | Add `--ignore-scripts` to skills install commands | — | ALREADY SYNCED |
| [#15390](https://github.com/openclaw/openclaw/pull/15390) | MERGED | security-fix | OC-02: Block `sessions_spawn` via HTTP gateway + fix ACP auto-approval (CVSS 9.8) | — | ALREADY SYNCED |
| [#14665](https://github.com/openclaw/openclaw/pull/14665) | MERGED | security-fix | Handle additional Unicode angle bracket homoglyphs in content sanitization | — | SYNC NEEDED |
| [#13073](https://github.com/openclaw/openclaw/pull/13073) | MERGED | security-fix | Finish credential redaction that was merged unfinished | [#10090](https://github.com/openclaw/openclaw/issues/10090) | SYNC NEEDED |
| [#4026](https://github.com/openclaw/openclaw/pull/4026) | MERGED | security-fix | Execute sandboxed file ops inside containers (symlink race fix) | — | SYNC NEEDED |
| [#15035](https://github.com/openclaw/openclaw/pull/15035) | MERGED | hardening | Auth rate-limiting & brute-force protection (CWE-307) | — | ALREADY SYNCED |
| [#10525](https://github.com/openclaw/openclaw/pull/10525) | MERGED | hardening | Use `openFileWithinRoot` for A2UI file serving | — | ALREADY SYNCED |
| [#10529](https://github.com/openclaw/openclaw/pull/10529) | MERGED | hardening | Enforce `0o600` permissions on WhatsApp credential files | — | ALREADY SYNCED |
| [#13129](https://github.com/openclaw/openclaw/pull/13129) | MERGED | hardening | Clarify dmScope remediation with explicit `openclaw config set` CLI command (merged 2026-02-13) | — | ALREADY SYNCED |
| [#13184](https://github.com/openclaw/openclaw/pull/13184) | MERGED | hardening | Default standalone servers (Telegram webhook, canvas host) to loopback bind (merged 2026-02-13) | — | ALREADY SYNCED |
| [#13185](https://github.com/openclaw/openclaw/pull/13185) | MERGED | hardening | Sanitize error responses to prevent information leakage across gateway HTTP endpoints (merged 2026-02-13) | — | ALREADY SYNCED |
| [#13767](https://github.com/openclaw/openclaw/pull/13767) | MERGED | security-fix | Reject literal "undefined" and "null" gateway auth tokens (merged 2026-02-13) | [#13756](https://github.com/openclaw/openclaw/issues/13756) | ALREADY SYNCED |
| [#14661](https://github.com/openclaw/openclaw/pull/14661) | MERGED | hardening | Restrict canvas IP-based auth to private networks (merged 2026-02-13) | — | ALREADY SYNCED |

### CLOSED PRs (not merged)

| PR | Status | Category | Summary | Related Issue | Local Impact |
|----|--------|----------|---------|---------------|--------------|
| [#8604](https://github.com/openclaw/openclaw/pull/8604) | CLOSED | security-fix | Unauthenticated Nostr profile endpoints allow remote profile takeover (closed 2026-02-15) | — | NOT AFFECTED |
| [#9513](https://github.com/openclaw/openclaw/pull/9513) | CLOSED | security-fix | Skill download archive path traversal checks (closed 2026-02-15; #9529 covers Zip Slip scope) | [#9512](https://github.com/openclaw/openclaw/issues/9512) | NOT AFFECTED |
| [#13290](https://github.com/openclaw/openclaw/pull/13290) | CLOSED | docs | Warn against storing secrets in workspace files (closed 2026-02-15; superseded by #16961) | — | NOT AFFECTED |
| [#13293](https://github.com/openclaw/openclaw/pull/13293) | CLOSED | hardening | Block tainted sink calls from untrusted tool outputs (closed 2026-02-15) | — | NOT AFFECTED |
| [#8757](https://github.com/openclaw/openclaw/pull/8757) | CLOSED | security-fix | Validate redirect destinations to prevent SSRF via open redirect (MS Teams) (closed 2026-02-13) | — | NOT AFFECTED |
| [#14843](https://github.com/openclaw/openclaw/pull/14843) | CLOSED | security-fix | Strip `apiKey` from `models.json` cache to prevent credential exposure (closed: author banned) | [#14808](https://github.com/openclaw/openclaw/issues/14808) | NOT AFFECTED |
| [#14655](https://github.com/openclaw/openclaw/pull/14655) | CLOSED | hardening | Allow trusted clients to request unredacted config via `includeSecrets` (closed: author banned) | [#14586](https://github.com/openclaw/openclaw/issues/14586) | NOT AFFECTED |
| [#14512](https://github.com/openclaw/openclaw/pull/14512) | CLOSED | hardening | Allow Docker bridge connections to extension relay via `OPENCLAW_RELAY_ALLOW_PRIVATE` (closed: author banned) | [#14433](https://github.com/openclaw/openclaw/issues/14433) | NOT AFFECTED |
| [#15384](https://github.com/openclaw/openclaw/pull/15384) | CLOSED | security-fix | OC-01: Validate `setupCommand` to prevent shell injection (CWE-78) (closed 2026-02-13; #8186 still OPEN) | — | NOT AFFECTED |
| [#15314](https://github.com/openclaw/openclaw/pull/15314) | CLOSED | security-fix | Arbitrary Code Execution via Browser Evaluate Endpoint (closed 2026-02-13; #3926 still OPEN) | [#15313](https://github.com/openclaw/openclaw/issues/15313) | NOT AFFECTED |
| [#13680](https://github.com/openclaw/openclaw/pull/13680) | CLOSED | hardening | Per-IP rate limiting for gateway auth (closed 2026-02-13; superseded by #15035 MERGED) | — | NOT AFFECTED |
| [#11152](https://github.com/openclaw/openclaw/pull/11152) | CLOSED | security-fix | Add `place_id` validation to prevent path traversal SSRF (closed 2026-02-13; no replacement) | — | NOT AFFECTED |
| [#14350](https://github.com/openclaw/openclaw/pull/14350) | CLOSED | hardening | Add `--harden` CLI flag for security-hardened gateway mode (closed 2026-02-13; no replacement) | — | NOT AFFECTED |
| [#15608](https://github.com/openclaw/openclaw/pull/15608) | CLOSED | hardening | Prune expired hook auth failure entries (closed 2026-02-14; superseded by #15848 MERGED) | — | NOT AFFECTED |

**Total:** 125 tracked PRs (34 merged, 77 open, 2 draft, 12 closed)

> **Status change log (16-02-2026 02:49 AEST):** 6 state changes detected. #15924 OPEN->MERGED (ALREADY SYNCED), #15940 OPEN->MERGED (ALREADY SYNCED), #8604 OPEN->CLOSED, #9513 OPEN->CLOSED, #13290 OPEN->CLOSED (superseded by #16961), #13293 OPEN->CLOSED. 15 new PRs added: #16898, #16907, #16928, #16929, #16936, #16947, #16958, #16959, #16961, #16963, #16965, #16967, #16990, #16992, #17182.
>
> **Status change log (14-02-2026 16:13 AEST):** 0 state changes detected. 1 new PR added: #16036 (OPEN hardening, session transcript file permissions CWE-732).
>
> **Status change log (14-02-2026 14:08 AEST):** 0 state changes detected. 2 new PRs added: #15924 (OPEN security-fix, macOS keychain shell injection CWE-78), #15940 (OPEN hardening, trusted-proxy auth mode).
>
> **Status change log (14-02-2026 11:36 AEST):** 1 state change detected. #15608 OPEN->CLOSED (superseded by #15848). 1 new PR added: #15848 (MERGED hardening, prune hook auth failure state, ALREADY SYNCED).
>
> **Status change log (14-02-2026 06:04 AEST):** 1 state change detected. #15652 OPEN->MERGED (ALREADY SYNCED). No new PRs added (#15734 assessed as feature addition, not tracked).
>
> **Status change log (14-02-2026 03:12 AEST):** 6 state changes detected. #13129 OPEN->MERGED, #13184 OPEN->MERGED, #13185 OPEN->MERGED, #13767 OPEN->MERGED, #14661 OPEN->MERGED (all ALREADY SYNCED). #14350 OPEN->CLOSED (no replacement). 1 new PR added: #15592 (WebSocket log header sanitization, OPEN).
>
> **Status change log (14-02-2026 01:33 AEST):** 7 state changes detected. #15390 OPEN->MERGED, #14665 OPEN->MERGED, #13073 OPEN->MERGED, #15384 OPEN->CLOSED, #15314 OPEN->CLOSED, #13680 OPEN->CLOSED (superseded by #15035), #11152 OPEN->CLOSED. 5 new PRs added: #15035, #10525, #10529, #4026 (newly merged security PRs), #3926 (replacement for closed #15314).

### Cross-Reference: PRs and Tracked Issues

| PR # | Fixes Issue(s) | Issue Severity | PR Status | Notes |
|------|---------------|----------------|-----------|-------|
| [#1795](https://github.com/openclaw/openclaw/pull/1795) | (unconfigured proxy bypass) | HIGH | MERGED | Fail-secure proxy detection in `isLocalDirectRequest()` at `src/gateway/auth.ts:101-122` |
| [#2016](https://github.com/openclaw/openclaw/pull/2016) | [#2015](https://github.com/openclaw/openclaw/issues/2015) | HIGH | MERGED | `noteSecurityWarnings()` at `src/commands/doctor-security.ts:11` checks gateway bind + auth |
| [#4880](https://github.com/openclaw/openclaw/pull/4880) | (LFI via MEDIA tokens) | HIGH | MERGED | `isValidMedia()` at `src/media/parse.ts:36-64` accepts all path types; LFI guard moved to `assertLocalMediaAllowed()` at `src/web/media.ts:47-89` |
| [#8241](https://github.com/openclaw/openclaw/pull/8241) | (Matrix thread isolation) | MEDIUM | MERGED | `:thread:${threadRootId}` suffix at `extensions/matrix/src/matrix/monitor/handler.ts:446-447` |
| [#8513](https://github.com/openclaw/openclaw/pull/8513) | [#8512](https://github.com/openclaw/openclaw/issues/8512) (CRITICAL) | CRITICAL | OPEN | Adds auth requirement for plugin HTTP routes in gateway |
| [#9436](https://github.com/openclaw/openclaw/pull/9436) | [#9435](https://github.com/openclaw/openclaw/issues/9435) (HIGH), [#5120](https://github.com/openclaw/openclaw/issues/5120) (MEDIUM) | HIGH | MERGED | Query token acceptance removed from `extractHookToken()` in `src/gateway/hooks.ts`; server returns HTTP 400 when `?token=` present |
| [#9518](https://github.com/openclaw/openclaw/pull/9518) | [#9517](https://github.com/openclaw/openclaw/issues/9517) (HIGH) | HIGH | MERGED | New `authorizeCanvasRequest()` at `src/gateway/server-http.ts:109-155` wraps canvas/A2UI endpoints |
| [#9529](https://github.com/openclaw/openclaw/pull/9529) | [#3277](https://github.com/openclaw/openclaw/issues/3277) (HIGH) | HIGH | OPEN | Validates archive entries against Zip Slip path traversal |
| [#9513](https://github.com/openclaw/openclaw/pull/9513) | [#9512](https://github.com/openclaw/openclaw/issues/9512) (HIGH) | HIGH | OPEN | Adds path traversal checks to skill download archive extraction |
| [#11054](https://github.com/openclaw/openclaw/pull/11054) | [#6609](https://github.com/openclaw/openclaw/issues/6609) (HIGH) | HIGH | OPEN | Adds auth token to sandbox browser bridge |
| [#11093](https://github.com/openclaw/openclaw/pull/11093) | [#10333](https://github.com/openclaw/openclaw/issues/10333) (MEDIUM) | MEDIUM | MERGED | `sanitizeFilename()` at `extensions/bluebubbles/src/attachments.ts:27-29` strips dangerous chars from Content-Disposition filenames |
| [#11169](https://github.com/openclaw/openclaw/pull/11169) | [#8776](https://github.com/openclaw/openclaw/issues/8776) (HIGH) | HIGH | OPEN | Removes bundled soul-evil hook that enables silent agent hijacking |
| [#11435](https://github.com/openclaw/openclaw/pull/11435) | [#11437](https://github.com/openclaw/openclaw/issues/11437) (CRITICAL) | CRITICAL | OPEN | Validates `OPENCLAW_BROWSER_CONTROL_MODULE` env var before dynamic `import()` |
| [#11432](https://github.com/openclaw/openclaw/pull/11432) | [#11434](https://github.com/openclaw/openclaw/issues/11434) (HIGH) | HIGH | OPEN | Adds `--ignore-scripts` to npm install in hook and plugin installers |
| [#11439](https://github.com/openclaw/openclaw/pull/11439) | [#11431](https://github.com/openclaw/openclaw/issues/11431) (HIGH) | HIGH | OPEN | Warns on relative `OPENCLAW_CONFIG_PATH` and disables config-origin plugin auto-enable |
| [#12260](https://github.com/openclaw/openclaw/pull/12260) | [#5995](https://github.com/openclaw/openclaw/issues/5995) (HIGH) | HIGH | OPEN | Redacts secrets from tool results before persisting to session transcripts |
| [#13117](https://github.com/openclaw/openclaw/pull/13117) | [#6606](https://github.com/openclaw/openclaw/issues/6606) (HIGH) | HIGH | OPEN | Enforces webhook authentication for Telegram webhook mode |
| [#13170](https://github.com/openclaw/openclaw/pull/13170) | [#6606](https://github.com/openclaw/openclaw/issues/6606) (HIGH) | HIGH | OPEN | Enforces `webhookSecret` at runtime for Telegram webhook mode |
| [#13182](https://github.com/openclaw/openclaw/pull/13182) | (refactor) | LOW | MERGED | Barrel re-export at `src/security/audit-extra.ts` → `.sync.ts` + `.async.ts` |
| [#13680](https://github.com/openclaw/openclaw/pull/13680) | (CWE-307 brute force) | MEDIUM | CLOSED | Superseded by #15035 (MERGED); rate limiter at `src/gateway/auth-rate-limit.ts` ALREADY SYNCED |
| [#13521](https://github.com/openclaw/openclaw/pull/13521) | [#13116](https://github.com/openclaw/openclaw/issues/13116) (HIGH) | HIGH | OPEN | Fail-close webhook secret enforcement; related to #13170 and #13117 |
| [#13474](https://github.com/openclaw/openclaw/pull/13474) | [#13466](https://github.com/openclaw/openclaw/issues/13466) (LOW) | LOW | MERGED | Split `hooks:` → `hooks.webhooks:` + `hooks.internal:` at `audit-extra.sync.ts:389-391`; merged 2026-02-13; ALREADY SYNCED |
| [#8718](https://github.com/openclaw/openclaw/pull/8718) | [#8696](https://github.com/openclaw/openclaw/issues/8696) (HIGH) | HIGH | OPEN | Sanitizes download filenames to prevent path traversal (CWE-22) |
| [#13787](https://github.com/openclaw/openclaw/pull/13787) | [#13786](https://github.com/openclaw/openclaw/issues/13786) (HIGH) | HIGH | MERGED | Loopback bypass in BlueBubbles webhook auth (CVSS 8.6); related to #11742; merged 2026-02-12 |
| [#13777](https://github.com/openclaw/openclaw/pull/13777) | [#13683](https://github.com/openclaw/openclaw/issues/13683) (HIGH) | HIGH | OPEN | Sandboxed agent credential exfil via `openclaw config get` — no redaction in CLI path |
| [#13767](https://github.com/openclaw/openclaw/pull/13767) | [#13756](https://github.com/openclaw/openclaw/issues/13756) (MEDIUM) | MEDIUM | MERGED | `normalizeGatewayTokenInput()` rejects "undefined"/"null" strings; merged 2026-02-13; ALREADY SYNCED |
| [#13876](https://github.com/openclaw/openclaw/pull/13876) | [#13196](https://github.com/openclaw/openclaw/issues/13196), [#13236](https://github.com/openclaw/openclaw/issues/13236) | HIGH | OPEN | CLI credential sync + config redaction regex fix + guard model |
| [#14029](https://github.com/openclaw/openclaw/pull/14029) | (token leakage in URL) | MEDIUM | MERGED | Twilio stream auth token via `<Parameter>` instead of query string; merged 2026-02-12; ALREADY SYNCED locally |
| [#14222](https://github.com/openclaw/openclaw/pull/14222) | (maintainer feedback on #8727) | MEDIUM | OPEN | Core `before_tool_call` hook extended with `needsApproval`; AgentShield moved to extension with encrypted approval store |
| [#14197](https://github.com/openclaw/openclaw/pull/14197) | (Codex CLI audit) | MEDIUM | OPEN | Shared `safeEqual()`, browser API auth (30+ unauthenticated endpoints), 7 timing-unsafe token comparisons, `hooks.allowQueryToken` deprecation |
| [#14061](https://github.com/openclaw/openclaw/pull/14061) | (Docker auth bypass) | HIGH | OPEN | Container on same Docker network can spoof `Host: localhost` to bypass `isLocalDirectRequest()` — adds `readDockerGatewayIp()` client IP verification |
| [#14218](https://github.com/openclaw/openclaw/pull/14218) | [#13765](https://github.com/openclaw/openclaw/issues/13765) | LOW | MERGED | Thinking block sanitization bypass via orphaned user-message repair path in `attempt.ts`; merged 2026-02-12 |
| [#14843](https://github.com/openclaw/openclaw/pull/14843) | [#14808](https://github.com/openclaw/openclaw/issues/14808) | MEDIUM | CLOSED | `apiKey` credential exposure in `models.json` cache; author banned; duplicate #14836 still OPEN |
| [#14665](https://github.com/openclaw/openclaw/pull/14665) | (homoglyph bypass) | MEDIUM | MERGED | `ANGLE_BRACKET_MAP` adds 10 Unicode homoglyphs to `foldMarkerChar()`; merged 2026-02-13; local code missing these -- SYNC NEEDED |
| [#14662](https://github.com/openclaw/openclaw/pull/14662) | (XSS in control UI) | MEDIUM | OPEN | `JSON.stringify` in `<script>` tag at `control-ui.ts:173-179` does not escape `<` |
| [#14661](https://github.com/openclaw/openclaw/pull/14661) | (canvas IP auth) | MEDIUM | MERGED | Canvas IP auth restricted to private networks; merged 2026-02-13; ALREADY SYNCED |
| [#14655](https://github.com/openclaw/openclaw/pull/14655) | [#14586](https://github.com/openclaw/openclaw/issues/14586) | LOW | CLOSED | `includeSecrets` for config.get; author banned; no replacement PR identified |
| [#14814](https://github.com/openclaw/openclaw/pull/14814) | (timing attack) | LOW | OPEN | Hook token at `server-http.ts:254` uses `safeEqualSecret()` while `auth.ts:256` uses `safeEqual()` |
| [#14098](https://github.com/openclaw/openclaw/pull/14098) | (tool-call JSON leak) | LOW | OPEN | `stripJsonToolCallText()` prevents Ollama/local providers from leaking raw tool-call JSON to user surfaces |
| [#14350](https://github.com/openclaw/openclaw/pull/14350) | (security hardening CLI) | MEDIUM | CLOSED | `--harden` flag closed without merge 2026-02-13; no replacement PR |
| [#14318](https://github.com/openclaw/openclaw/pull/14318) | (outbound channel control) | MEDIUM | OPEN | `enforceOutboundAllowlist()` blocks Discord sends to non-allowed channels; local `send.outbound.ts` has zero guards |
| [#14689](https://github.com/openclaw/openclaw/pull/14689) | [#14688](https://github.com/openclaw/openclaw/issues/14688) | LOW | OPEN | Auto-set `per-channel-peer` dmScope during multi-channel onboarding; Greptile flags multi-account channels need `per-account-channel-peer` |
| [#14512](https://github.com/openclaw/openclaw/pull/14512) | [#14433](https://github.com/openclaw/openclaw/issues/14433) | LOW | CLOSED | Author banned; `isPrivateNetworkAddress()` for Docker bridge relay needs re-implementation |
| [#14224](https://github.com/openclaw/openclaw/pull/14224) | (admin enumeration) | LOW | OPEN | Telegram `getChatAdministrators` action — gate inconsistency may default to enabled |
| [#14112](https://github.com/openclaw/openclaw/pull/14112) | [#13132](https://github.com/openclaw/openclaw/issues/13132) | MEDIUM | OPEN | Integration test verifying `--ignore-scripts` blocks postinstall in plugin archive install; related to #11432 |
| [#13737](https://github.com/openclaw/openclaw/pull/13737) | (Docker privilege hardening) | LOW | OPEN | UID/GID remap in Dockerfile; Greptile flags GID collision not detected (silently reuses existing group) |
| [#13290](https://github.com/openclaw/openclaw/pull/13290) | (openclaw/trust#1) | LOW | OPEN | Security warnings in TOOLS.md templates and system-prompt docs that workspace files enter model context |
| [#15390](https://github.com/openclaw/openclaw/pull/15390) | (OC-02 RCE via HTTP gateway) | CRITICAL | MERGED | `DEFAULT_GATEWAY_HTTP_TOOL_DENY` at `tools-invoke-http.ts:43-52` blocks sessions_spawn + 3 others; merged 2026-02-13; ALREADY SYNCED |
| [#15384](https://github.com/openclaw/openclaw/pull/15384) | (OC-01 shell injection) | CRITICAL | CLOSED | Closed 2026-02-13; #8186 still OPEN with same fix; local `docker.ts:360-361` still vulnerable |
| [#15314](https://github.com/openclaw/openclaw/pull/15314) | [#15313](https://github.com/openclaw/openclaw/issues/15313) (HIGH) | HIGH | CLOSED | Closed 2026-02-13; #3926 still OPEN with same fix; local `DEFAULT_BROWSER_EVALUATE_ENABLED` still true |
| [#13073](https://github.com/openclaw/openclaw/pull/13073) | [#10090](https://github.com/openclaw/openclaw/issues/10090) (HIGH) | HIGH | MERGED | Credential redaction completion; `zod-schema.sensitive.ts` and schema hint updates; merged 2026-02-13; SYNC NEEDED |
| [#4026](https://github.com/openclaw/openclaw/pull/4026) | (symlink race) | HIGH | MERGED | `SandboxFsBridge` routes file ops through `docker exec`; merged 2026-02-13; SYNC NEEDED |
| [#15035](https://github.com/openclaw/openclaw/pull/15035) | (CWE-307 brute force) | MEDIUM | MERGED | `AuthRateLimiter` at `src/gateway/auth-rate-limit.ts`; merged 2026-02-13; ALREADY SYNCED |
| [#10525](https://github.com/openclaw/openclaw/pull/10525) | (A2UI path traversal) | MEDIUM | MERGED | `openFileWithinRoot()` at `a2ui.ts:75`; merged 2026-02-13; ALREADY SYNCED |
| [#10529](https://github.com/openclaw/openclaw/pull/10529) | (WhatsApp cred perms) | MEDIUM | MERGED | `0o600` chmod at `auth-store.ts:72`, `session.ts:77,91`; merged 2026-02-13; ALREADY SYNCED |
| [#15379](https://github.com/openclaw/openclaw/pull/15379) | [#13826](https://github.com/openclaw/openclaw/issues/13826) | LOW | OPEN | No `transformContext` hook in local `pi-embedded-runner`; sanitization only at session load (`attempt.ts:778-779`), not during tool-call iterations |
| [#15360](https://github.com/openclaw/openclaw/pull/15360) | [#6669](https://github.com/openclaw/openclaw/issues/6669) | LOW | OPEN | `statsLine` with token counts, session IDs, file paths, costs in user-visible `message` at `subagent-announce.ts:545-559` |
| [#15296](https://github.com/openclaw/openclaw/pull/15296) | (config secret hardening) | MEDIUM | OPEN | No `--show-secrets` opt-in for CLI `config get`; gateway `config.get` returns unredacted by default |
| [#8757](https://github.com/openclaw/openclaw/pull/8757) | (MS Teams SSRF redirect) | MEDIUM | CLOSED | Closed without merge 2026-02-13; MS Teams redirect validation SSRF fix; no replacement PR identified |
| [#13129](https://github.com/openclaw/openclaw/pull/13129) | (dmScope UX) | LOW | MERGED | Uses `formatCliCommand()` for dmScope remediation; local `audit.ts:603` already uses it; ALREADY SYNCED |
| [#13184](https://github.com/openclaw/openclaw/pull/13184) | (standalone bind hardening) | LOW | MERGED | Default standalone servers to loopback bind; local `webhook.ts:41` and `canvas-host/server.ts:452` already default to `127.0.0.1`; ALREADY SYNCED |
| [#13185](https://github.com/openclaw/openclaw/pull/13185) | (error info leakage) | MEDIUM | MERGED | Sanitize error responses across gateway HTTP; local `tools-invoke-http.ts:382-384` returns generic "tool execution failed"; ALREADY SYNCED |
| [#13767](https://github.com/openclaw/openclaw/pull/13767) | [#13756](https://github.com/openclaw/openclaw/issues/13756) | MEDIUM | MERGED | Reject "undefined"/"null" token strings; local `onboard-helpers.ts:79` already rejects; ALREADY SYNCED |
| [#14350](https://github.com/openclaw/openclaw/pull/14350) | (security hardening CLI) | MEDIUM | CLOSED | `--harden` flag closed without merge 2026-02-13; no replacement PR |
| [#14661](https://github.com/openclaw/openclaw/pull/14661) | (canvas IP auth) | MEDIUM | MERGED | Canvas IP auth restricted to private networks; local `server-http.ts:150` uses `isPrivateOrLoopbackAddress()`; ALREADY SYNCED |
| [#15592](https://github.com/openclaw/openclaw/pull/15592) | (log injection) | LOW | MERGED | Sanitize WebSocket log headers; local `ws-connection.ts:40-54` has `sanitizeLogValue()`; ALREADY SYNCED |
| [#15604](https://github.com/openclaw/openclaw/pull/15604) | (link-understanding SSRF) | MEDIUM | MERGED | Block private/loopback/metadata IPs in link-understanding; local `detect.ts:35-42` has `isBlockedHost()`; ALREADY SYNCED |
| [#15615](https://github.com/openclaw/openclaw/pull/15615) | (PATH shadow attack) | MEDIUM | OPEN | Restrict PATH override to exact match; local `runner.ts:230-233` allows prepending via `endsWith()` |
| [#15652](https://github.com/openclaw/openclaw/pull/15652) | (browser path traversal) | MEDIUM | MERGED | Constrain browser trace/download output paths; local `path-output.ts` has `resolvePathWithinRoot()`; ALREADY SYNCED |
| [#15608](https://github.com/openclaw/openclaw/pull/15608) | (rate-limit state reset) | LOW | CLOSED | Superseded by #15848 (MERGED); NOT AFFECTED |
| [#15848](https://github.com/openclaw/openclaw/pull/15848) | (rate-limit state pruning) | LOW | MERGED | Prune expired hook auth failure entries; local `server-http.ts:209-226` has prune-then-evict logic; ALREADY SYNCED |
| [#15924](https://github.com/openclaw/openclaw/pull/15924) | (macOS keychain shell injection) | HIGH | MERGED | OC-28; merged 2026-02-14; local `cli-credentials.ts:338-376` uses `execFileSync` with array args; ALREADY SYNCED |
| [#15940](https://github.com/openclaw/openclaw/pull/15940) | [#1560](https://github.com/openclaw/openclaw/issues/1560) | MEDIUM | MERGED | Trusted-proxy auth mode; merged 2026-02-14; `authorizeTrustedProxy()` + `GatewayTrustedProxyConfig` ALREADY SYNCED |
| [#16036](https://github.com/openclaw/openclaw/pull/16036) | [#8751](https://github.com/openclaw/openclaw/issues/8751) | MEDIUM | OPEN | Session transcript `.jsonl` files use default `0o644` (world-readable); fix: `0o600` across 7+ write paths; Greptile 5/5 confidence |
| [#16929](https://github.com/openclaw/openclaw/pull/16929) | [#8588](https://github.com/openclaw/openclaw/issues/8588) | HIGH | OPEN | Block sensitive dirs (~/.openclaw/) from sandbox; `isSensitivePath()` + post-symlink realpath check |
| [#16907](https://github.com/openclaw/openclaw/pull/16907) | [#8592](https://github.com/openclaw/openclaw/issues/8592) | MEDIUM | OPEN | Detect obfuscated commands (base64, hex, variable expansion) bypassing allowlist |
| [#16963](https://github.com/openclaw/openclaw/pull/16963) | [#16876](https://github.com/openclaw/openclaw/issues/16876) | MEDIUM | OPEN | Enable auth rate limiter by default; add `enabled` boolean for explicit opt-out |
| [#16959](https://github.com/openclaw/openclaw/pull/16959) | [#12536](https://github.com/openclaw/openclaw/issues/12536) | MEDIUM | OPEN | Skill packager follows symlinks; `is_symlink()` check + path traversal validation |
| [#16958](https://github.com/openclaw/openclaw/pull/16958) | [#12538](https://github.com/openclaw/openclaw/issues/12538) | MEDIUM | OPEN | Gallery `write_gallery()` embeds user prompts without `html.escape()` |
| [#16965](https://github.com/openclaw/openclaw/pull/16965) | [#16379](https://github.com/openclaw/openclaw/issues/16379) | MEDIUM | OPEN | Workspace guard blocks Docker bind-mounted directories; `additionalRoots` option |
| [#17182](https://github.com/openclaw/openclaw/pull/17182) | [#17158](https://github.com/openclaw/openclaw/issues/17158) | MEDIUM | OPEN | LINE webhook startup accepts empty auth; strict guards for `channelAccessToken` and `channelSecret` |

### #1795: Prevent Auth Bypass Behind Unconfigured Reverse Proxy

**PR Status:** MERGED (2026-01-25)
**Category:** security-fix
**Closes:** (addresses unconfigured reverse proxy auth bypass)

**Changes:**
- `src/gateway/server/ws-connection/message-handler.ts:210-216` — detects untrusted proxy headers
- `src/gateway/auth.ts:101-122` — `isLocalDirectRequest()` returns `false` when proxy headers present but `trustedProxies` not configured (fail-secure)
- `src/security/audit.ts` — audit detection for misconfigured proxy setup

**Local Impact:** ALREADY SYNCED — `isLocalDirectRequest()` at `src/gateway/auth.ts:101-122` checks `hasForwarded` and `remoteIsTrustedProxy`

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
- `src/web/media.ts:47-89` — new `assertLocalMediaAllowed()` validates local paths against allowed directory roots (`/tmp`, `~/.openclaw/media`, `~/.openclaw/agents`), resolves symlinks
- `src/media/parse.test.ts` — test coverage updated for new path acceptance + load-time validation

**Local Impact:** ALREADY SYNCED — LFI defense now two-layer: `isValidMedia()` at `src/media/parse.ts:36-64` accepts paths, `assertLocalMediaAllowed()` at `src/web/media.ts:47-89` enforces directory root guards

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
- `src/security/audit-extra.sync.ts` — config-only checks (884 lines)
- `src/security/audit-extra.async.ts` — I/O-based checks (933 lines)

**Local Impact:** ALREADY SYNCED — barrel re-export at `src/security/audit-extra.ts:1-31`

### #9436: Remove Query Token Acceptance from Gateway Hooks

**PR Status:** MERGED
**Category:** security-fix
**Closes:** [#9435](https://github.com/openclaw/openclaw/issues/9435) (HIGH — gateway auth token in URL), [#5120](https://github.com/openclaw/openclaw/issues/5120) (MEDIUM — webhook token via query params)

**Changes:**
- `src/gateway/hooks.ts` — `extractHookToken()` no longer reads `url.searchParams` for token
- `src/gateway/server-http.ts:243-249` — returns HTTP 400 when `?token=` query parameter is present
- `src/commands/dashboard.ts` — no longer constructs `?token=` URLs
- `src/commands/onboard-helpers.ts` — no longer passes token in URL

**Local Impact:** SYNC NEEDED — local `extractHookToken()` may still accept query tokens

### #9518: Add Canvas/A2UI Authorization

**PR Status:** MERGED
**Category:** security-fix
**Closes:** [#9517](https://github.com/openclaw/openclaw/issues/9517) (HIGH — canvas host auth bypass)

**Changes:**
- `src/gateway/server-http.ts:109-155` — new `authorizeCanvasRequest()` function
- `src/gateway/server-http.ts:527-539` — canvas HTTP handler now auth-wrapped
- `src/gateway/server-http.ts:596` — canvas WebSocket upgrade now auth-wrapped

**Local Impact:** SYNC NEEDED — local canvas endpoints may still be unauthenticated

### #11093: BlueBubbles Filename Sanitization

**PR Status:** MERGED
**Category:** security-fix
**Closes:** [#10333](https://github.com/openclaw/openclaw/issues/10333) (MEDIUM — multipart header injection)

**Changes:**
- `extensions/bluebubbles/src/attachments.ts:27-29` — `sanitizeFilename()` now strips `"`, `\r`, `\n`, and other Content-Disposition-dangerous characters beyond just `path.basename()`

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

### #13474: Distinguish Webhooks from Internal Hooks in Audit Summary

**PR Status:** MERGED (2026-02-13)
**Category:** hardening
**Closes:** [#13466](https://github.com/openclaw/openclaw/issues/13466) (LOW)

**Changes:**
- `src/security/audit-extra.sync.ts` — splits `hooks:` summary into `hooks.webhooks:` and `hooks.internal:` in attack surface output
- `src/security/audit-extra.sync.test.ts` — updated test coverage

**Local Impact:** ALREADY SYNCED — `hooks.webhooks:` at `src/security/audit-extra.sync.ts:389` and `hooks.internal:` at line 391 already present in local code

### #15390: OC-02 Block sessions_spawn via HTTP Gateway + Fix ACP Auto-Approval

**PR Status:** MERGED (2026-02-13)
**Category:** security-fix
**Closes:** (OC-02 Aether AI security audit finding)

**Security Impact:** OC-02 Critical RCE (CWE-78, CVSS 9.8). Two attack vectors:
1. HTTP POST to `/tools/invoke` with `tool=sessions_spawn` spawns agent sessions with full exec access
2. ACP client auto-approves all permission requests including exec, fs_write

**Changes:**
- `src/config/types.gateway.ts` — adds `GatewayToolsConfig` type with `tools.allow` / `tools.deny`
- `src/gateway/tools-invoke-http.ts:43-52` — adds `DEFAULT_GATEWAY_HTTP_TOOL_DENY` constant blocking `sessions_spawn`, `sessions_send`, `gateway`, `whatsapp_login`
- `src/gateway/tools-invoke-http.ts:325` — applies deny list filtering before tool lookup
- `src/acp/client.ts` — replaces auto-approve with danger-aware permission handler

**Local Validation:**
- `src/gateway/tools-invoke-http.ts:43-52` — `DEFAULT_GATEWAY_HTTP_TOOL_DENY` exists with all 4 blocked tools
- `src/gateway/tools-invoke-http.ts:325` — deny list filtering applied before tool lookup

**Local Impact:** ALREADY SYNCED — deny list and filtering present locally (synced in Feb 14 sync 1)

### #15384: OC-01 Validate setupCommand to Prevent Shell Injection

**PR Status:** CLOSED (2026-02-13, without merge)
**Category:** security-fix
**Closes:** (OC-01 Aether AI security audit finding)

**Security Impact:** CWE-78 shell injection. `setupCommand` config field passed verbatim to `sh -lc` in Docker container creation.

**Greptile Review:** 4 files, 2 security comments:
1. Allowlist only covers `npm`, `pip`, etc.; existing configs with `echo work` will break (breaking change)
2. On validation failure, code logs and continues (should fail hard); allowed prefixes still allow `npm run postinstall` / `npx <pkg>`

**Note:** This PR was closed without merge. The same vulnerability is still addressed by [#8186](https://github.com/openclaw/openclaw/pull/8186) which remains OPEN.

**Local Validation:**
- `src/agents/sandbox/docker.ts:360-361` — `cfg.setupCommand` passed directly to `["exec", "-i", name, "sh", "-lc", cfg.setupCommand]` with only `?.trim()` check
- No `validateSetupCommand` function exists locally

**Local Impact:** NOT AFFECTED (PR closed) — vulnerability remains; monitor #8186 for fix.

### #15314: Arbitrary Code Execution via Browser Evaluate Endpoint

**PR Status:** CLOSED (2026-02-13, without merge)
**Category:** security-fix
**Closes:** [#15313](https://github.com/openclaw/openclaw/issues/15313) (HIGH — CVSS 8.8)

**Security Impact:** The `/act` endpoint with `kind: "evaluate"` allows arbitrary JavaScript execution via `eval()` and `new Function()` in browser context when `evaluateEnabled=true` (current default).

**Note:** This PR was closed without merge. The same vulnerability is still addressed by [#3926](https://github.com/openclaw/openclaw/pull/3926) which remains OPEN.

**Local Validation:**
- `src/browser/constants.ts:2` — `DEFAULT_BROWSER_EVALUATE_ENABLED = true` (enabled by default!)
- `src/browser/config.ts:143` — fallback to default when not configured
- `src/browser/routes/agent.act.ts:293-299` — evaluate handler checks flag but default is permissive
- Existing test at `server.agent-contract-form-layout-act-commands.test.ts:374` verifies blocking when false

**Local Impact:** NOT AFFECTED (PR closed) — vulnerability remains; monitor #3926 for fix.

### #15379: Strip Unsigned Thinking Blocks in Agent Loop via transformContext

**PR Status:** OPEN (2026-02-13)
**Category:** hardening
**Closes:** [#13826](https://github.com/openclaw/openclaw/issues/13826)

**Changes:**
- `src/agents/pi-embedded-runner/run/attempt.ts` — uses `Agent.transformContext` hook to run `sanitizeAntigravityThinkingBlocks` before every API call (not just at session load)

**Local Validation:**
- `src/agents/pi-embedded-runner/run/attempt.ts:893` — `sanitizeAntigravityThinkingBlocks` only at orphaned user-message repair (session load)
- No `transformContext` hook usage in local `pi-embedded-runner` (grep confirmed)
- `src/agents/pi-embedded-runner/google.ts:67` — `sanitizeAntigravityThinkingBlocks` function exists but only called from `attempt.ts:893` and `google.ts:458`

**Local Impact:** OPEN/PENDING — PR not yet merged. Relates to #14218 (thinking sanitization bypass).

### #15360: Prevent Subagent Announce from Leaking Internal Prompts and Stats

**PR Status:** OPEN (2026-02-13)
**Category:** hardening
**Closes:** [#6669](https://github.com/openclaw/openclaw/issues/6669)

**Changes:**
- `src/agents/subagent-announce.ts` — moves findings and LLM instructions from user-visible `message` to `extraSystemPrompt` (hidden)
- `src/agents/subagent-announce-queue.ts` — adds `extraSystemPrompt` field to `AnnounceQueueItem`
- Removes `statsLine` (token counts, session IDs, file paths, costs) from announce payload; logs internally instead

**Greptile Review:** Notes queue draining logic may drop `extraSystemPrompt` when messages are combined or summarized.

**Local Validation:**
- `src/agents/subagent-announce.ts:545-559` — `statsLine` (token counts, session IDs, file paths, costs) in user-visible `message`; `Findings:` heading and `Summarize this naturally` instructions also exposed
- No `extraSystemPrompt` field in local `AnnounceQueueItem`

**Local Impact:** OPEN/PENDING — PR not yet merged. Internal stats and instructions leak to users.

### #15296: Harden Config Redaction Defaults; Add --show-secrets Opt-in

**PR Status:** OPEN (2026-02-13)
**Category:** hardening

**Changes:**
- `src/cli/config-cli.ts` — CLI `openclaw config get` redacts by default; adds `--show-secrets` flag with warning
- `src/gateway/protocol/schema/config.ts` — gateway `config.get` supports `showSecrets?: boolean`
- `src/gateway/server-methods/config.ts` — default behavior returns redacted config

**Local Validation:**
- No `--show-secrets` flag exists in local CLI (grep confirmed on `src/cli/`)
- `src/commands/status-all/channels.ts` has `showSecrets` parameter for channel display but `config get` path has no redaction by default

**Local Impact:** OPEN/PENDING — PR not yet merged. Config get returns unredacted values locally by default.

### #14659: Add --ignore-scripts to Skills Install Commands

**PR Status:** MERGED (2026-02-12)
**Category:** hardening

**Changes:**
- `src/agents/skills-install.ts` — adds `--ignore-scripts` to all package manager commands in `buildNodeInstallCommand()`

**Local Impact:** ALREADY SYNCED — `--ignore-scripts` already present at lines 150, 152, 154, 156 for pnpm/yarn/bun/npm

### #15390: OC-02 Block sessions_spawn via HTTP Gateway + Fix ACP Auto-Approval

**PR Status:** MERGED (2026-02-13)
**Category:** security-fix
**Closes:** (OC-02 Aether AI security audit finding)

**Security Impact:** OC-02 Critical RCE (CWE-78, CVSS 9.8). Two attack vectors:
1. HTTP POST to `/tools/invoke` with `tool=sessions_spawn` spawns agent sessions with full exec access
2. ACP client auto-approves all permission requests including exec, fs_write

**Changes:**
- `src/config/types.gateway.ts` — adds `GatewayToolsConfig` type with `tools.allow` / `tools.deny`
- `src/gateway/tools-invoke-http.ts:43-52` — adds `DEFAULT_GATEWAY_HTTP_TOOL_DENY` constant blocking `sessions_spawn`, `sessions_send`, `gateway`, `whatsapp_login`
- `src/gateway/tools-invoke-http.ts:325` — applies deny list filtering before tool lookup
- `src/acp/client.ts` — replaces auto-approve with danger-aware permission handler

**Local Impact:** ALREADY SYNCED — deny list at `src/gateway/tools-invoke-http.ts:43-52` with filtering at line 325; synced in Feb 14 sync 1

### #14665: Handle Additional Unicode Angle Bracket Homoglyphs in Content Sanitization

**PR Status:** MERGED (2026-02-13)
**Category:** security-fix

**Security Impact:** 10 additional Unicode angle bracket homoglyphs can bypass `foldMarkerChar()` to craft fake external content boundary markers that LLMs may interpret as trusted content boundaries.

**Changes:**
- `src/security/external-content.ts` — replaces `FULLWIDTH_LEFT_ANGLE`/`FULLWIDTH_RIGHT_ANGLE` constants with `ANGLE_BRACKET_MAP` lookup table covering:
  - Mathematical angle brackets (U+27E8/U+27E9)
  - CJK angle brackets (U+3008/U+3009)
  - Left/right-pointing angle brackets (U+2329/U+232A)
  - Single angle quotation marks (U+2039/U+203A)
  - Small less-than/greater-than signs (U+FE64/U+FE65)
- `foldMarkerText()` regex expanded to match all new Unicode ranges

**Local Validation:**
- `src/security/external-content.ts:89-125` — `foldMarkerChar()` now handles fullwidth ASCII and 12 Unicode angle bracket homoglyphs via `ANGLE_BRACKET_MAP`
- `ANGLE_BRACKET_MAP` lookup table at lines 90-103 covers FF1C/FF1E, 2329/232A, 3008/3009, 2039/203A, 27E8/27E9, FE64/FE65
- `foldMarkerText()` regex at line 122 matches `[\uFF21-\uFF3A\uFF41-\uFF5A\uFF1C\uFF1E\u2329\u232A\u3008\u3009\u2039\u203A\u27E8\u27E9\uFE64\uFE65]`

**Local Impact:** SYNC NEEDED — local code only handles fullwidth homoglyphs; 10 additional Unicode angle bracket variants can bypass sanitization

### #13073: Finish Credential Redaction That Was Merged Unfinished

**PR Status:** MERGED (2026-02-13)
**Category:** security-fix
**Closes:** [#10090](https://github.com/openclaw/openclaw/issues/10090)

**Changes:**
- `src/config/zod-schema.sensitive.ts` (NEW) — centralized sensitive field metadata
- `src/config/schema.hints.ts` — updated hints for sensitive fields
- `src/config/schema.ts` — integrated sensitive field metadata
- `src/config/zod-schema.*.ts` — multiple Zod schema files updated with sensitive markers
- `src/gateway/server-methods/config.ts` — server-side redaction improvements
- `ui/src/ui/views/config-form.*.ts` — UI form redaction

**Local Validation:**
- `src/config/zod-schema.sensitive.ts` does NOT exist locally
- `src/config/redact-snapshot.ts` exists with basic redaction but lacks centralized sensitive metadata from the new module

**Local Impact:** SYNC NEEDED — missing `zod-schema.sensitive.ts` and related schema/hint updates

### #4026: Execute Sandboxed File Ops Inside Containers

**PR Status:** MERGED (2026-02-13)
**Category:** security-fix

**Security Impact:** Moves sandboxed file operations from host into Docker containers via `SandboxFsBridge`. Prevents symlink race-condition attacks (TOCTOU) that are impossible to fix on the host side since Node.js does not expose `openat()`.

**Changes (20 files):**
- `src/agents/apply-patch-update.ts` — routes apply_patch through sandbox bridge
- `src/agents/apply-patch.ts` — sandbox-aware patch application
- `src/agents/openclaw-tools.ts` — tool registry updated for sandbox bridge
- `src/agents/pi-embedded-runner/run/attempt.ts` — sandbox bridge integration in runner
- `src/agents/pi-tools.read.ts` — read tool uses bridge in sandbox mode
- `src/agents/pi-tools.ts` — tool definitions updated

**Local Validation:**
- `SandboxFsBridge` does NOT exist locally (grep confirmed across `src/`)
- File operations in sandbox mode still execute on the host side
- Host-side file ops are vulnerable to symlink race conditions (TOCTOU)

**Local Impact:** SYNC NEEDED — local code executes sandboxed file operations on the host, vulnerable to symlink race attacks

### #15035: Auth Rate-Limiting & Brute-Force Protection

**PR Status:** MERGED (2026-02-13)
**Category:** hardening

**Changes (24 files):**
- `src/gateway/auth-rate-limit.ts` (NEW) — sliding-window rate limiter with per-scope counters
- `src/gateway/auth.ts` — integrates rate limiter with `check()` / `recordFailure()` / `reset()`
- `src/gateway/server.impl.ts:301-304` — creates rate limiter from `gateway.auth.rateLimit` config
- All gateway entry points (HTTP, WebSocket, tools-invoke, openai-http, openresponses-http) — pass rate limiter

**Local Impact:** ALREADY SYNCED — `auth-rate-limit.ts` exists with full implementation; integrated across all gateway entry points

### #10525: Use openFileWithinRoot for A2UI File Serving

**PR Status:** MERGED (2026-02-13)
**Category:** hardening

**Changes:**
- `src/canvas-host/a2ui.ts` — uses `openFileWithinRoot()` instead of unchecked `readFile` for A2UI asset serving

**Local Impact:** ALREADY SYNCED — `openFileWithinRoot` at `src/canvas-host/a2ui.ts:75` and `src/canvas-host/server.ts:167`

### #10529: Enforce 0o600 Permissions on WhatsApp Credential Files

**PR Status:** MERGED (2026-02-13)
**Category:** hardening

**Changes:**
- `src/web/auth-store.ts` — `chmodSync(credsPath, 0o600)` after writing credentials
- `src/web/session.ts` — `chmodSync` on session backup and credentials file

**Local Impact:** ALREADY SYNCED — `0o600` chmod at `auth-store.ts:72`, `session.ts:77`, `session.ts:91`

### #13129: Clarify dmScope Remediation with Explicit CLI Command

**PR Status:** MERGED (2026-02-13)
**Category:** hardening

**Changes:**
- `src/security/audit.ts` — uses `formatCliCommand('openclaw config set session.dmScope "per-channel-peer"')` in audit warning remediation text

**Local Impact:** ALREADY SYNCED — `formatCliCommand()` already at `src/security/audit.ts:603`, `src/commands/doctor-security.ts:128`, `src/commands/onboard-channels.ts:198,254`

### #13184: Default Standalone Servers to Loopback Bind

**PR Status:** MERGED (2026-02-13)
**Category:** hardening

**Changes:**
- `src/telegram/webhook.ts` — default bind host to `127.0.0.1` instead of all interfaces
- `src/canvas-host/server.ts` — default bind host to `127.0.0.1`
- `extensions/telegram/src/channel.ts`, `docs/channels/telegram.md` — updated config and documentation

**Local Impact:** ALREADY SYNCED — `src/telegram/webhook.ts:41` has `opts.host ?? "127.0.0.1"`, `src/canvas-host/server.ts:452` has `opts.listenHost?.trim() || "127.0.0.1"`

### #13185: Sanitize Error Responses to Prevent Information Leakage

**PR Status:** MERGED (2026-02-13)
**Category:** hardening

**Changes:**
- `src/gateway/tools-invoke-http.ts` — sanitize error messages in HTTP responses
- `src/gateway/openai-http.ts` — generic "internal error" for 500 responses
- `src/gateway/openresponses-http.ts` — generic "internal error" for 500 responses
- `src/agents/tools/common.ts` — error message formatting

**Local Impact:** ALREADY SYNCED — `tools-invoke-http.ts:382-384` returns generic "tool execution failed" for 500; `openai-http.ts:270` and `openresponses-http.ts:629,930` return "internal error"

### #13767: Reject Literal "undefined" and "null" Gateway Auth Tokens

**PR Status:** MERGED (2026-02-13)
**Category:** security-fix
**Closes:** [#13756](https://github.com/openclaw/openclaw/issues/13756) (MEDIUM)

**Changes:**
- `src/commands/onboard-helpers.ts` — `normalizeGatewayTokenInput()` rejects literal "undefined" and "null" strings
- `src/commands/configure.gateway-auth.ts` — comment documenting the behavior
- Test coverage for both rejected values

**Local Impact:** ALREADY SYNCED — `onboard-helpers.ts:79` has `if (trimmed === "undefined" || trimmed === "null") return ""`, tests at `onboard-helpers.e2e.test.ts:127,131`

### #14661: Restrict Canvas IP-Based Auth to Private Networks

**PR Status:** MERGED (2026-02-13)
**Category:** hardening

**Changes:**
- `src/gateway/server-http.ts` — IP-based auth fallback restricted to private/loopback addresses
- `src/gateway/net.ts` — `isPrivateOrLoopbackAddress()` utility function

**Local Impact:** ALREADY SYNCED — `server-http.ts:150-155` with `isPrivateOrLoopbackAddress(clientIp)` check; test at `server.canvas-auth.e2e.test.ts:84` confirms behavior

### #15592: Gateway: Sanitize WebSocket Log Headers

**PR Status:** MERGED (2026-02-13T17:11:54Z)
**Category:** hardening

**Changes:**
- `src/gateway/server/ws-connection.ts` — adds `sanitizeLogValue()` to truncate and strip control characters from untrusted header values (host, origin, user-agent, x-forwarded-for) and close reasons before logging

**Greptile Review:** Flags that `sanitizeLogValue` only strips C0/C1 control characters; Unicode bidi overrides (U+202E, U+2066-U+2069) still pass through and could produce misleading log output.

**Local Validation:**
- `src/gateway/server/ws-connection.ts:39-54` — local code has `sanitizeLogValue()` function with control character stripping, format regex, whitespace normalization, and UTF-16-safe truncation
- Lines 195-199 — applied to `forwardedFor`, `requestOrigin`, `requestHost`, `requestUserAgent`, and `reason`

**Local Impact:** ALREADY SYNCED — `sanitizeLogValue()` present locally with full sanitization pipeline

### #14350: Add --harden CLI Flag for Security-Hardened Gateway Mode

**PR Status:** CLOSED (2026-02-13, without merge)
**Category:** hardening

**Description:** Proposed `--harden` CLI flag that would force loopback bind, token auth, and disable Tailscale. Closed without merge; no replacement PR identified.

**Local Impact:** NOT AFFECTED — PR closed without merge

### #15604: Block Private/Loopback/Metadata IPs in Link-Understanding URL Detection

**PR Status:** MERGED (2026-02-13T17:38:40Z)
**Category:** security-fix

**Security Impact:** `isAllowedUrl()` in `src/link-understanding/detect.ts` only blocked `127.0.0.1`. Missing blocks for `localhost`, `0.0.0.0`, `[::1]`, RFC1918 private ranges, link-local (`169.254.x.x`), cloud metadata (`169.254.169.254`), and CGNAT (`100.64.0.0/10`). URLs extracted here are passed to CLI tools for fetching, creating an SSRF vector.

**Changes:**
- `src/link-understanding/detect.ts` — adds `isBlockedHost()` that checks loopback hostnames, IPv6 loopback, all private IPv4 ranges, link-local, and CGNAT
- `src/link-understanding/detect.test.ts` — tests for localhost, private ranges, link-local, metadata, CGNAT

**Greptile Review:** Confidence 2/5. Flags that `isBlockedHost()` only blocks private IP literals, not hostnames that resolve/redirect to private IPs (DNS-based SSRF still possible). The repo's existing SSRF guard (`fetch-guard.ts`/`ssrf.ts`) handles DNS resolution but is not used here.

**Local Validation:**
- `src/link-understanding/detect.ts:1` — imports `isBlockedHostname`, `isPrivateIpAddress` from `../infra/net/ssrf.js`
- `src/link-understanding/detect.ts:35-42` — `isBlockedHost()` uses both `isBlockedHostname()` and `isPrivateIpAddress()` with `localhost.localdomain` check

**Local Impact:** ALREADY SYNCED — local `detect.ts` has complete `isBlockedHost()` implementation

### #15615: Restrict PATH Override to Exact Match in node-host sanitizeEnv

**PR Status:** OPEN (2026-02-13)
**Category:** security-fix

**Security Impact:** `sanitizeEnv()` accepted any PATH value ending with the current base PATH via `endsWith()` check, allowing prepending arbitrary directories (e.g., `/tmp/evil:/usr/local/bin:/usr/bin`). An authenticated caller could shadow allowlisted binaries with attacker-controlled executables.

**Changes:**
- `src/node-host/runner.ts` — only accept PATH overrides that exactly match the current base PATH; reject any prepending or modification

**Greptile Review:** Confidence 4/5. Flags that empty or undefined PATH overrides are silently ignored, which may leave inherited PATH in effect.

**Local Validation:**
- `src/node-host/runner.ts:221-234` — local `sanitizeEnv()` has the vulnerable pattern:
  - Line 226: `if (!basePath || trimmed === basePath)` — exact match OK
  - Lines 230-233: `if (trimmed.endsWith(suffix))` — allows prepending arbitrary directories

**Local Impact:** OPEN/PENDING — PR not yet merged. Local `sanitizeEnv()` at `runner.ts:230-233` allows PATH prepending.

### #15652: Constrain Browser Trace/Download Output Paths to OpenClaw Temp Roots

**PR Status:** MERGED (2026-02-13T19:24:33Z)
**Category:** hardening

**Security Impact:** Browser file-output routes (`POST /trace/stop`, `POST /wait/download`, `POST /download`) accept arbitrary paths, creating a path traversal/escape vector. This PR constrains them to OpenClaw temp roots.

**Changes:**
- `src/browser/routes/path-output.ts` (NEW) — shared helper for route-level output path resolution
- `src/browser/routes/agent.debug.ts`, `src/browser/routes/agent.act.ts` — wired into browser routes
- Tests, CLI, docs updated

**Greptile Review:** Confidence 4/5. Flags edge case where `resolvePathWithinRoot` rejects paths that resolve exactly to the root directory.

**Local Validation:**
- `src/browser/routes/path-output.ts:1` — re-exports from `src/browser/paths.ts` (implementation moved Feb 15 sync 2)
- `src/browser/routes/agent.debug.ts:7,17,135-146` — imports and uses `resolvePathWithinRoot()` for trace endpoint
- `src/browser/routes/agent.act.ts:20,450-518` — imports and uses `resolvePathWithinRoot()` for downloads

**Local Impact:** ALREADY SYNCED — all changes from PR #15652 are present in local code

### #15608: Prune Expired Hook Auth Failure Entries Instead of Clearing All State

**PR Status:** CLOSED (2026-02-14, without merge — superseded by #15848)
**Category:** hardening

**Security Impact:** Hook auth failure tracking used `hookAuthFailures.clear()` when at capacity (2048 entries), resetting ALL rate limiting state. Previously-throttled clients could bypass limits by triggering the capacity clear. The fix prunes expired entries first, then evicts the oldest half.

**Note:** This PR was closed without merge. The same fix was merged via [#15848](https://github.com/openclaw/openclaw/pull/15848) at 2026-02-14T00:46:12Z.

**Local Impact:** NOT AFFECTED — PR closed without merge; see #15848 (ALREADY SYNCED)

### #15848: Prune Expired Hook Auth Failure Entries (Merged Implementation)

**PR Status:** MERGED (2026-02-14T00:46:12Z)
**Category:** hardening
**Supersedes:** #15608 (CLOSED)

**Security Impact:** Hook auth failure tracking used `hookAuthFailures.clear()` when at capacity (2048 entries), resetting ALL rate limiting state. Fix: two-phase eviction — prune expired entries first, then evict oldest half. Refresh Map insertion order on update via delete+re-insert so active clients are not evicted before dormant ones.

**Changes:**
- `src/gateway/server-http.ts` — replaces `clear()` with prune-then-evict logic + insertion order refresh

**Greptile Review:** 1 comment. Flags HTTP status regression: `readJsonBody()` timeout case collapsed into 400 instead of 408.

**Local Validation:**
- `src/gateway/server-http.ts:209-226` — prune expired entries at lines 211-214, drop oldest half at lines 217-225
- `src/gateway/server-http.ts:233-238` — delete-before-set refreshes insertion order for recently-active clients
- No `hookAuthFailures.clear()` call exists locally

**Local Impact:** ALREADY SYNCED — all changes from PR #15848 are present in local code

### #15924: OC-28 Prevent Shell Injection in macOS Keychain Credential Write

**PR Status:** MERGED (2026-02-14T16:06:10Z)
**Category:** security-fix
**Closes:** (OC-28 shell injection in keychain write)

**Security Impact:** OC-28 — CWE-78 shell injection. `writeClaudeCliKeychainCredentials()` previously used `execSync()` with string interpolation for `security add-generic-password`. The single-quote escaping was insufficient; `$()` command substitution and backtick expansion bypass the protection.

**Changes:**
- `src/agents/cli-credentials.ts` — replace `execSync` with `execFileSync` (array args, no shell parsing)
- `src/agents/cli-credentials.e2e.test.ts` — updated tests for `execFileSync` API + new injection payload tests

**Local Validation:**
- `src/agents/cli-credentials.ts:338` — `execFileSyncImpl` used (not `execSync`)
- `src/agents/cli-credentials.ts:361-363` — Comment documents security rationale: "Use execFileSync to avoid shell interpretation of user-controlled token values"
- `src/agents/cli-credentials.ts:363-376` — Array args passed to `execFileSync`, no shell parsing

**Local Impact:** ALREADY SYNCED — local code uses `execFileSync` with array args at the keychain write path

### #15940: Add Trusted-Proxy Auth Mode Types and Schema

**PR Status:** MERGED (2026-02-14T11:32:17Z)
**Category:** hardening
**Closes:** [#1560](https://github.com/openclaw/openclaw/issues/1560)

**Security Impact:** Adds `trusted-proxy` to `GatewayAuthMode` union for delegating authentication to reverse proxies (Pomerium, Caddy, nginx + OAuth). Includes `authorizeTrustedProxy()` helper, CIDR matching for proxy IP validation, security audit integration (critical finding when enabled), and 10 new auth tests.

**Changes (25 files):**
- `src/gateway/auth.ts` — new `authorizeTrustedProxy()` helper, updated `authorizeGatewayConnect()` for trusted-proxy mode
- `src/config/types.gateway.ts` — `GatewayTrustedProxyConfig` type with `userHeader`, `requiredHeaders`, `allowUsers`
- `src/config/zod-schema.ts` — validation for new config fields
- `src/gateway/net.ts` — CIDR matching for proxy IP validation
- `src/security/audit.ts` and `audit-extra.sync.ts` — critical audit finding when trusted-proxy auth enabled

**Greptile Review:** 25 files reviewed, 2 comments. 1 inline comment on `src/gateway/net.ts:183` about bitwise left shift on negative values needing unsigned right shift for mask calculation.

**Local Validation:**
- `src/config/types.gateway.ts:79` — `GatewayAuthMode` includes `"trusted-proxy"`
- `src/config/types.gateway.ts:86` — `GatewayTrustedProxyConfig` type with `userHeader`, `requiredHeaders`, `allowUsers`
- `src/gateway/auth.ts:244` — `authorizeTrustedProxy()` function present
- `src/gateway/auth.ts:300-315` — trusted-proxy auth path in `authorizeGatewayConnect()`
- `src/security/audit.ts:377-436` — audit integration with critical findings for trusted-proxy
- `src/config/zod-schema.ts:402` — Zod validation for trusted-proxy mode
- 10+ test files with trusted-proxy coverage

**Local Impact:** ALREADY SYNCED — full trusted-proxy auth infrastructure present locally

### #16036: Use 0o600 Permissions for Session Transcript Files

**PR Status:** OPEN (2026-02-14)
**Category:** hardening
**Closes:** [#8751](https://github.com/openclaw/openclaw/issues/8751)

**Security Impact:** CWE-732 — Session transcript `.jsonl` files contain full conversation history including user messages, tool call arguments, and model responses. Created with default `0o644` (world-readable) permissions. Fix restricts all session file write paths to `0o600` (owner-only), matching the permission model already used by `saveJsonFile()` for credential and config files.

**Changes (8 files):**
- `src/config/sessions/transcript.ts` — `ensureSessionHeader()` adds `mode: 0o600`
- `src/agents/pi-embedded-helpers/bootstrap.ts` — `ensureSessionHeader()` adds `mode: 0o600`
- `src/agents/pi-embedded-runner/session-manager-init.ts` — session reset adds `mode: 0o600`
- `src/auto-reply/reply/session.ts` — `forkSessionFromParent()` adds `mode: 0o600`
- `src/config/sessions/store.ts` — Windows-specific write path adds `mode: 0o600`
- `src/gateway/server-methods/chat.ts` — `ensureTranscriptFile()` adds `mode: 0o600`
- `src/gateway/server-methods/sessions.ts` — manual compaction adds `mode: 0o600`
- `src/agents/session-file-repair.ts` — backup and repair writes add `mode: 0o600`

**Greptile Review:** Confidence 5/5. "Safe to merge — focused security fix with no logic changes." All modifications follow the same `{ encoding: "utf-8", mode: 0o600 }` pattern consistently.

**Local Validation:**
- `src/config/sessions/transcript.ts:75` — `writeFile(params.sessionFile, ..., "utf-8")` — no `mode: 0o600`
- `src/agents/pi-embedded-helpers/bootstrap.ts:159` — `writeFile(file, ..., "utf-8")` — no `mode: 0o600`
- `src/gateway/server-methods/chat.ts:89` — `writeFileSync(params.transcriptPath, ..., "utf-8")` — no `mode: 0o600`
- `src/auto-reply/reply/session.ts:92` — `writeFileSync(sessionFile, ..., "utf-8")` — no `mode: 0o600`
- `src/agents/pi-embedded-runner/session-manager-init.ts:46` — `writeFile(params.sessionFile, "", "utf-8")` — no `mode: 0o600`
- `src/gateway/server-methods/sessions.ts:490` — `writeFileSync(filePath, ..., "utf-8")` — no `mode: 0o600`
- `src/agents/session-file-repair.ts:77,81` — `writeFile(..., "utf-8")` — no `mode: 0o600`
- Note: `src/config/sessions/store.ts:544,559` already uses `{ mode: 0o600 }` for sessions.json

**Local Impact:** OPEN/PENDING — PR not yet merged. 7+ local transcript write paths use default world-readable permissions.
