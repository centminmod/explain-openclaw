> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Open PRs](./open-upstream-prs.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## Open Upstream Security Issues

> **Status:** These issues are open in upstream openclaw/openclaw and confirmed to affect the local codebase. Monitor for patches.
>
> **Last checked:** 14-02-2026 (06:06 AEST)

| Issue | Severity | Summary | Local Impact |
|-------|----------|---------|--------------|
| [#8512](https://github.com/openclaw/openclaw/issues/8512) | CRITICAL | Plugin HTTP routes bypass gateway authentication | `src/gateway/server/plugins-http.ts:17-59` |
| [#3277](https://github.com/openclaw/openclaw/issues/3277) | HIGH | Path validation bypass via `startsWith` prefix | `src/infra/archive.ts:81,89` - zip/tar extraction |
| [#4949](https://github.com/openclaw/openclaw/issues/4949) | HIGH | Browser control server DNS rebinding | `src/browser/server.ts:133` - auth middleware at `:117-124` mitigates but no Host header validation |
| [#4950](https://github.com/openclaw/openclaw/issues/4950) | HIGH | Arbitrary JS execution via browser evaluate (default on) | `src/browser/constants.ts:2` - `DEFAULT_BROWSER_EVALUATE_ENABLED = true` |
| [#4995](https://github.com/openclaw/openclaw/issues/4995) | HIGH | iMessage dmPolicy auto-responds with pairing codes | `src/imessage/monitor/monitor-provider.ts:184,342-381` |
| [#5052](https://github.com/openclaw/openclaw/issues/5052) | HIGH | Config validation fail-open returns `{}` | `src/config/io.ts:505-510` - security settings reset |
| [#5255](https://github.com/openclaw/openclaw/issues/5255) | HIGH | Browser file upload arbitrary read | `src/browser/pw-tools-core.interactions.ts:531` |
| [#5995](https://github.com/openclaw/openclaw/issues/5995) | HIGH | Secrets exposed in session transcripts | `config.get` now redacted via `redactConfigSnapshot()` (PR #9858); transcripts still expose by design |
| [#6606](https://github.com/openclaw/openclaw/issues/6606) | HIGH (WONTFIX) | Telegram webhook binds to 0.0.0.0 with optional secret | Closed upstream as NOT_PLANNED (2026-02-13); still affects local code at `src/telegram/webhook.ts:26,36,46-48` |
| [#6609](https://github.com/openclaw/openclaw/issues/6609) | HIGH | Browser bridge server optional authentication | `src/browser/bridge-server.ts:33-42` |
| [#8054](https://github.com/openclaw/openclaw/issues/8054) | ~~HIGH~~ FIXED | Type coercion `"undefined"` credentials | Fixed upstream (COMPLETED 2026-02-13); `src/wizard/onboarding.gateway-config.ts:206` |
| [#8516](https://github.com/openclaw/openclaw/issues/8516) | HIGH | Browser download/trace endpoints arbitrary file write | `src/browser/routes/agent.act.ts:447-480` |
| [#8586](https://github.com/openclaw/openclaw/issues/8586) | HIGH | Configurable bypass allows unrestricted command exec | `src/agents/bash-tools.exec.ts:199-207,269-271` |
| [#8591](https://github.com/openclaw/openclaw/issues/8591) | HIGH (WONTFIX) | Env vars exposed via shell commands | Closed upstream as NOT_PLANNED (2026-02-13); still affects local code at `src/agents/bash-tools.exec.ts:290,298` |
| [#8590](https://github.com/openclaw/openclaw/issues/8590) | HIGH | Status endpoint exposes sensitive internal info | `src/gateway/server-methods/health.ts:28-31` |
| [#8696](https://github.com/openclaw/openclaw/issues/8696) | HIGH | Playwright download path traversal | `src/browser/pw-tools-core.downloads.ts:20-24` |
| [#8776](https://github.com/openclaw/openclaw/issues/8776) | ~~HIGH~~ FIXED | soul-evil hook silently hijacks agent | Fixed in PR [#14757](https://github.com/openclaw/openclaw/pull/14757) — soul-evil hook completely removed |
| [#9435](https://github.com/openclaw/openclaw/issues/9435) | ~~HIGH~~ FIXED | Gateway auth token exposed in URL query params | Fixed in PR [#9436](https://github.com/openclaw/openclaw/pull/9436) — query token acceptance removed from `src/gateway/hooks.ts`, dashboard URL no longer passes `?token=` |
| [#9512](https://github.com/openclaw/openclaw/issues/9512) | HIGH | Skill download archive path traversal | `src/agents/skills-install.ts:267,274` |
| [#9517](https://github.com/openclaw/openclaw/issues/9517) | ~~HIGH~~ FIXED | Gateway canvas host auth bypass | Fixed in PR [#9518](https://github.com/openclaw/openclaw/pull/9518) — new `authorizeCanvasRequest()` at `src/gateway/server-http.ts:109-155` |
| [#9627](https://github.com/openclaw/openclaw/issues/9627) | HIGH | Config secrets exposed in JSON after update/doctor | `src/config/io.ts:681-781` — partially mitigated by `redactConfigSnapshot()` (PR #9858) + env var reference preservation (commit `f59df9589`) |
| [#9813](https://github.com/openclaw/openclaw/issues/9813) | HIGH (DUPLICATE #9627) | Gateway expands ${ENV_VAR} on meta writeback | `src/config/io.ts:681-781` — partially mitigated by `redactConfigSnapshot()` (PR #9858) + env var reference preservation (commit `f59df9589`) |
| [#11126](https://github.com/openclaw/openclaw/issues/11126) | HIGH (DUP #9627, WONTFIX) | Config write paths resolve ${VAR} to cleartext | Closed upstream as NOT_PLANNED (2026-02-13); same as #9627/#9813 — `src/config/io.ts:681-781` |
| [#9795](https://github.com/openclaw/openclaw/issues/9795) | LOW | sanitizeMimeType regex not end-anchored (by design) | `src/media-understanding/apply.ts:96-106` |
| [#9792](https://github.com/openclaw/openclaw/issues/9792) | INVALID | validateHostEnv skips baseEnv (by design) | `src/agents/bash-tools.exec-runtime.ts:54` (definition) + `src/agents/bash-tools.exec.ts:295` (call) |
| [#9791](https://github.com/openclaw/openclaw/issues/9791) | INVALID | Fullwidth marker bypass (fold is length-preserving) | `src/security/external-content.ts:127-167` |
| [#9667](https://github.com/openclaw/openclaw/issues/9667) | INVALID | JWT verification in nonexistent file | `src/auth/jwt.ts` (does not exist) |
| [#4940](https://github.com/openclaw/openclaw/issues/4940) | MEDIUM | commands.restart bypass via exec tool | `src/agents/bash-tools.exec.ts` (no commands.restart check — remains open) |
| [#5120](https://github.com/openclaw/openclaw/issues/5120) | ~~MEDIUM~~ FIXED | Webhook token accepted via query parameters | Fixed in PR [#9436](https://github.com/openclaw/openclaw/pull/9436) — query token extraction removed from `src/gateway/hooks.ts` (note: upstream issue still OPEN) |
| [#5122](https://github.com/openclaw/openclaw/issues/5122) | ~~MEDIUM~~ MITIGATED | readJsonBody() Slowloris DoS (no read timeout) | `src/gateway/hooks.ts:177-194` — now delegates to `readJsonBodyWithLimit()` from `src/infra/http-body.ts` with 30s timeout (commit `3cbcba10c`) |
| [#5123](https://github.com/openclaw/openclaw/issues/5123) | MEDIUM | ReDoS in session filter regex | `src/infra/exec-approval-forwarder.ts:70-77` |
| [#5124](https://github.com/openclaw/openclaw/issues/5124) | MEDIUM | ReDoS in log redaction patterns | `src/logging/redact.ts:47-61` |
| [#6021](https://github.com/openclaw/openclaw/issues/6021) | MEDIUM (WONTFIX) | Timing attack in non-gateway token comparisons | Closed upstream as NOT_PLANNED (2026-02-13); partially mitigated locally (hook token + device pairing use `safeEqualSecret`); `src/infra/node-pairing.ts:277` still uses `===` |
| [#7862](https://github.com/openclaw/openclaw/issues/7862) | MEDIUM | Session transcripts 644 instead of 600 | `src/auto-reply/reply/session.ts:87` |
| [#8027](https://github.com/openclaw/openclaw/issues/8027) | MEDIUM | web_fetch hidden text prompt injection | `src/agents/tools/web-fetch-utils.ts:31-34` |
| [#8592](https://github.com/openclaw/openclaw/issues/8592) | MEDIUM | No detection of encoded/obfuscated commands | `src/infra/exec-safety.ts:1-44` |
| [#8588](https://github.com/openclaw/openclaw/issues/8588) | MEDIUM | Sensitive config files accessible when sandbox is home dir | `src/agents/sandbox/context.ts:39-46` (workspaceAccess=rw) |
| [#8589](https://github.com/openclaw/openclaw/issues/8589) | LOW | Sandbox file read lacks content filtering | `src/agents/pi-tools.read.ts:286-302` (no redaction on read) |
| [#8593](https://github.com/openclaw/openclaw/issues/8593) | MEDIUM | chat.send handler lacks input length validation | `src/gateway/protocol/schema/logs-chat.ts:34-45` |
| [#8594](https://github.com/openclaw/openclaw/issues/8594) | MEDIUM (WONTFIX) | No rate limiting on gateway endpoints | Closed upstream as NOT_PLANNED (2026-02-13); still affects local code at `src/gateway/server-constants.ts` (no rate limit controls) |
| [#9007](https://github.com/openclaw/openclaw/issues/9007) | LOW | Google Places URL path interpolation (skill, not core) | `skills/local-places/src/local_places/google_places.py:238` |
| [#9065](https://github.com/openclaw/openclaw/issues/9065) | LOW | ~/.openclaw group-writable after sudo install | Operational - code uses `0o700` but sudo bypasses |
| [#10324](https://github.com/openclaw/openclaw/issues/10324) | MEDIUM | Memory index multi-write lacks transactions | `src/memory/manager.ts:2198-2301` (DELETEs+INSERTs without BEGIN/COMMIT) |
| [#10326](https://github.com/openclaw/openclaw/issues/10326) | MEDIUM | Child process stop() lacks SIGKILL escalation | `src/imessage/client.ts:110-131`, `src/signal/daemon.ts:96-100` |
| [#10330](https://github.com/openclaw/openclaw/issues/10330) | MEDIUM | TOCTOU race in device auth token storage | `src/infra/device-auth-store.ts:92-119` (read+write with no lock) |
| [#10331](https://github.com/openclaw/openclaw/issues/10331) | MEDIUM | Session store stale cache inside write lock | `src/config/sessions/store.ts:667,729` (partially fixed — `updateSessionStore()` at :575 now uses `skipCache: true`; two callers remain unfixed) |
| [#10333](https://github.com/openclaw/openclaw/issues/10333) | ~~MEDIUM~~ FIXED | BlueBubbles filename multipart header injection | Fixed in PR [#11093](https://github.com/openclaw/openclaw/pull/11093) — `sanitizeFilename()` at `extensions/bluebubbles/src/attachments.ts:26-30` |
| [#10646](https://github.com/openclaw/openclaw/issues/10646) | HIGH | Weak UUID: Math.random() fallback + tool call IDs | `ui/src/ui/uuid.ts:23-33` (fallback), `src/auto-reply/reply/get-reply-inline-actions.ts:191` (toolCallId) |
| [#7139](https://github.com/openclaw/openclaw/issues/7139) | MEDIUM | Default config: sandbox off, plaintext creds | `src/agents/sandbox/config.ts:147` (mode="off"), gateway loopback is safe; creds 0o600 |
| [#9875](https://github.com/openclaw/openclaw/issues/9875) | MEDIUM | Orphaned tool_use blocks from backgrounded exec | `src/agents/session-transcript-repair.ts:166-318` (reactive repair, not proactive) |
| [#11900](https://github.com/openclaw/openclaw/issues/11900) | MEDIUM | Context files (USER.md, SOUL.md) loaded for all senders | `src/agents/bootstrap-files.ts:43-60` — no `senderIsOwner` check; `attempt.ts:192` calls unconditionally |
| [#12571](https://github.com/openclaw/openclaw/issues/12571) | MEDIUM | Session isolation leak in cron jobs after ~24h | `src/cron/service/jobs.ts` — isolated sessions leak to main session after extended runtime |
| [#11832](https://github.com/openclaw/openclaw/issues/11832) | MEDIUM | Per-agent tools.exec config not applied | `src/auto-reply/reply/get-reply-directives.ts:66-81` — `resolveExecOverrides()` ignores `agentCfg` |
| [#6304](https://github.com/openclaw/openclaw/issues/6304) | LOW | Matrix plugin transitive dep vuln (request pkg) | `extensions/matrix/package.json` — transitive via `@vector-im/matrix-bot-sdk` (CVE-2023-28155) |
| [#4807](https://github.com/openclaw/openclaw/issues/4807) | LOW | Sandbox setup script missing from npm package | `package.json` files array excludes `scripts/`; `scripts/sandbox-common-setup.sh` not shipped |
| [#3359](https://github.com/openclaw/openclaw/issues/3359) | ~~MEDIUM~~ MITIGATED | npm audit vulns in tar/hono | `package.json` pnpm.overrides: tar@7.5.7, hono@4.11.8 (above vuln thresholds) |
| [#3086](https://github.com/openclaw/openclaw/issues/3086) | ~~LOW~~ FIXED | Windows ACL false flag as mode=666 | `src/security/audit-fs.ts:86-116` + `src/security/windows-acl.ts` — icacls-based ACL checks implemented |
| [#10521](https://github.com/openclaw/openclaw/issues/10521) | INVALID | Security audit flags claude-opus-4-6 as below 4.5 | `src/security/audit-extra.sync.ts:177` (`isClaude45OrHigher` regex) correctly matches `claude-opus-4-6` in current code (2026.2.6) |
| [#10033](https://github.com/openclaw/openclaw/issues/10033) | ENHANCEMENT (WONTFIX) | Feature: secrets management integration | Closed upstream as NOT_PLANNED (2026-02-13); current state: plaintext creds with 0o600 perms |
| [#10927](https://github.com/openclaw/openclaw/issues/10927) | ENHANCEMENT | Random IDs for external content wrapper tags | `src/security/external-content.ts:47-48` — static tags; `replaceMarkers()` at `:127-167` already sanitizes injected markers |
| [#10890](https://github.com/openclaw/openclaw/issues/10890) | ENHANCEMENT | RFC: Skill Security Framework (manifests, signing, sandboxing) | Comprehensive proposal for phased skill security; relates to #9512 (skill path traversal) |
| [#11437](https://github.com/openclaw/openclaw/issues/11437) | CRITICAL | CWD .env → config path override → plugin code exec via jiti | `src/infra/dotenv.ts:10`, `src/config/paths.ts:87-105`, `src/plugins/config-state.ts:73,194` |
| [#11434](https://github.com/openclaw/openclaw/issues/11434) | CRITICAL | CWD .env → arbitrary dynamic import via OPENCLAW_BROWSER_CONTROL_MODULE | `src/gateway/server-browser.ts:13-14` — raw `await import(override)` |
| [#11431](https://github.com/openclaw/openclaw/issues/11431) | ~~CRITICAL~~ FIXED | Hook/plugin npm install runs lifecycle scripts (no --ignore-scripts) | Fixed in PRs: `92702af7a` (plugins+hooks, Feb 12 sync 1) + [#14659](https://github.com/openclaw/openclaw/pull/14659) (skills, Feb 13 sync 1) — `--ignore-scripts` added to all install commands |
| [#11023](https://github.com/openclaw/openclaw/issues/11023) | HIGH (WONTFIX) | Sandbox browser bridge started without auth token | Closed upstream as NOT_PLANNED (2026-02-13); still affects local code at `src/agents/sandbox/browser.ts:192`; relates to #6609 |
| [#11945](https://github.com/openclaw/openclaw/issues/11945) | HIGH (WONTFIX) | config.patch bypasses commands.restart restriction | Closed upstream as NOT_PLANNED (2026-02-13); still affects local code at `src/gateway/server-methods/config.ts:330` |
| [#13683](https://github.com/openclaw/openclaw/issues/13683) | HIGH | CLI `config get` returns unredacted secrets to sandboxed agents | `src/cli/config-cli.ts:269-270` — reads `snapshot.config` without `redactConfigObject()`; gateway RPC at `server-methods/config.ts:108` correctly redacts |
| [#13786](https://github.com/openclaw/openclaw/issues/13786) | ~~HIGH~~ FIXED | BlueBubbles webhook auth bypass via loopback proxy trust | Fixed in PR [#13787](https://github.com/openclaw/openclaw/pull/13787) — loopback bypass removed; all requests require password auth |
| [#13718](https://github.com/openclaw/openclaw/issues/13718) | ~~HIGH~~ FIXED | Unauthenticated Nostr profile API allows remote config tampering | Fixed in PR [#13719](https://github.com/openclaw/openclaw/pull/13719) — gateway-auth required for `/api/channels/` plugin routes (`server-http.ts:484-497`) |
| [#13937](https://github.com/openclaw/openclaw/issues/13937) | ~~MEDIUM~~ FIXED | HTML not escaped in Control UI webchat (XSS) | Closed as COMPLETED 2026-02-11; `ui/` webchat HTML escaping fix applied upstream |
| [#14137](https://github.com/openclaw/openclaw/issues/14137) | HIGH | Gateway auth has no rate limiting (CWE-307) | `src/gateway/auth.ts` — no brute-force protection; ~645 attempts/sec; fix PR [#13680](https://github.com/openclaw/openclaw/pull/13680) pending; relates to #8594 |
| [#13274](https://github.com/openclaw/openclaw/issues/13274) | HIGH | SSRF guard bypassed by IPv4-compatible IPv6 addresses | `src/infra/net/ssrf.ts:144-165` — `isPrivateIpAddress()` does not handle `::127.0.0.1` or `::7f00:1` |
| [#11738](https://github.com/openclaw/openclaw/issues/11738) | HIGH | Canvas authorization IP co-tenancy bypass | `src/gateway/server-http.ts:100-105` — `hasAuthorizedWsClientForIp()` trusts any request from same IP as authenticated WS client; bypasses auth in NAT/proxy environments |
| [#11793](https://github.com/openclaw/openclaw/issues/11793) | HIGH | HTTP API session keys lack ownership validation | `src/gateway/http-utils.ts:71-73` — `x-openclaw-session-key` header accepted as-is with no ownership check; cross-user session access in multi-user deployments |
| [#11024](https://github.com/openclaw/openclaw/issues/11024) | HIGH | Gmail push endpoint embeds auth token in URL query string | `src/hooks/gmail-setup-utils.ts:315` — push endpoint constructed as `?token=<secret>`; same CWE-598 as fixed #9435/#5120 but Gmail-specific path |
| [#11811](https://github.com/openclaw/openclaw/issues/11811) | HIGH | MSTeams attachment fetch follows redirects before allowlist checks (SSRF) | `extensions/msteams/src/attachments/download.ts:93` — `fetchFn(params.url)` with default redirect behavior; allowlist checked on initial URL only |
| [#14875](https://github.com/openclaw/openclaw/issues/14875) | ~~HIGH~~ FIXED | Feishu channel hardcodes CommandAuthorized bypassing access groups | Fixed upstream (COMPLETED 2026-02-13); `extensions/feishu/src/bot.ts:818,906` |
| [#14117](https://github.com/openclaw/openclaw/issues/14117) | MEDIUM | Session isolation & message attribution failure | Cross-session message leakage between main + remote sessions; raw cron output exposed; relates to #12571 |
| [#14808](https://github.com/openclaw/openclaw/issues/14808) | MEDIUM (WONTFIX, DUP #9627) | apiKey resolved to plaintext in models.json cache | Closed upstream as NOT_PLANNED (2026-02-13); `src/agents/models-config.ts:126-142` — `normalizeProviders()` includes resolved apiKey; relates to #9627/#13683 |
| [#11202](https://github.com/openclaw/openclaw/issues/11202) | MEDIUM | Model catalog apiKeys injected into LLM prompt context every turn | `src/agents/models-config.ts` — `normalizeProviders()` includes resolved `apiKey` in model catalog serialized to LLM; all provider keys sent to active provider |
| [#12173](https://github.com/openclaw/openclaw/issues/12173) | MEDIUM | apply_patch tool path traversal when sandbox disabled | `src/agents/apply-patch.ts:215-236` — `resolvePatchPath()` falls through to `resolvePathFromCwd()` with no containment when `sandboxRoot` is undefined (default) |
| [#10659](https://github.com/openclaw/openclaw/issues/10659) | ENHANCEMENT | Feature: Masked secrets to prevent agent reading raw API keys | Enhancement request; relates to #10033 (secrets management) |
| [#9325](https://github.com/openclaw/openclaw/issues/9325) | NOT APPLICABLE | Skill removal without notification | ClawHub platform moderation issue, not a codebase vulnerability |
| [#11879](https://github.com/openclaw/openclaw/issues/11879) | NOT APPLICABLE | Malicious ClawHub skill exfiltrating to Feishu | Ecosystem/marketplace issue; 13,981 installs; relates to #10890 (Skill Security Framework) |

### #10646: Weak UUID / Math.random() in Tool Call IDs

**Vulnerability:** Two distinct Math.random() usages in security-relevant contexts.

1. **UI UUID fallback** (`ui/src/ui/uuid.ts:23-33`): `weakRandomBytes()` uses `Math.floor(Math.random() * 256)` when Web Crypto API is unavailable. Low practical risk since modern environments always have crypto.

2. **Tool call IDs** (`src/auto-reply/reply/get-reply-inline-actions.ts:191`): `cmd_${Date.now()}_${Math.random().toString(16).slice(2)}` — always uses Math.random(), no crypto path. Used for tool call/result correlation in session transcripts.

**Secondary:** Tmp file suffixes in `src/cron/store.ts:52`, `src/cron/run-log.ts:38`, `src/browser/trash.ts:16`, `src/tts/tts.ts:380` (low risk, collision-resistant via PID/timestamp).

**Confirmed safe:** 52 files use `crypto.randomUUID`/`randomBytes` for proper crypto paths.

### #7139: Default Config — Sandbox Disabled, Plaintext Credentials

**Sandbox defaults to "off"** at `src/agents/sandbox/config.ts:147`:

```
mode: agentSandbox?.mode ?? agent?.mode ?? "off"
```

A Docker sandbox implementation exists with proper isolation (`--network none`, `--cap-drop ALL`, `--read-only`) but is opt-in only. Gateway bind defaults to "loopback" (safe). Credentials are plaintext with 0o600 permissions, flagged by `collectSecretsInConfigFindings()`. File permission hardening is extensive (80+ locations use 0o600/0o700).

### #3277: Path Validation Bypasses

**Vulnerability:** `startsWith(params.destDir)` is bypassable when paths share prefixes (e.g., `/tmp/foo` vs `/tmp/foobar`). Tar extraction has zero path validation.

**Affected code:**
- `src/infra/archive.ts:81,89` - zip extraction
- `src/infra/archive.ts:112` - tar extraction with no filter

### #5052: Config Validation Silently Drops Security Settings

**Vulnerability:** When config validation fails, the entire config including `dmPolicy`, `allowFrom`, and all security settings are silently reset to `{}`. The bot will respond to ANY sender.

**Affected code:** `src/config/io.ts:505-510`

### #5255: Browser File Upload Arbitrary Read

**Vulnerability:** The `setInputFilesViaPlaywright` function accepts user-controlled file paths and passes them directly to Playwright without validation. Attackers with browser control access can read arbitrary files readable by the OpenClaw process.

**Affected code:** `src/browser/pw-tools-core.interactions.ts:531` - `opts.paths` passed directly to `setInputFiles()` without path validation.

### #5995: Secrets in Session Transcripts

**Vulnerability:** When agents call `gateway config.get` or shell commands like `env`, resolved secret values are persisted to session transcript `.jsonl` files. Even users following best practices (env vars, 1Password) have secrets logged.

**Note:** `logging.redactSensitive` only affects console output, not transcripts.

### #8027: web_fetch Hidden Text Prompt Injection

**Vulnerability:** HTML elements with `style="display:none"` or `visibility:hidden` pass through to agent context. Malicious web pages can inject hidden instructions.

**Affected code:** `src/agents/tools/web-fetch-utils.ts:31-34` strips `<script>/<style>/<noscript>` but not CSS-hidden elements.

### #8054: Type Coercion "undefined" Credentials

**Status: FIXED** -- closed as COMPLETED upstream 2026-02-13

**Severity:** ~~HIGH~~ FIXED

**Vulnerability:** `String(undefined).trim()` produces the literal string `"undefined"`, not an empty string. Attackers may authenticate with password/token `"undefined"`.

**Affected code:** `src/wizard/onboarding.gateway-config.ts:206` and similar patterns across CLI files.

### #8512: Plugin HTTP Routes Bypass Gateway Authentication (CRITICAL)

**Severity:** CRITICAL (CVSS 10.0)
**CWE:** CWE-862 (Missing Authorization)

**Vulnerability:** Gateway plugin HTTP routes are dispatched without any gateway authentication checks. `createGatewayPluginRequestHandler()` accepts only `registry` and `log` parameters — no authentication context, token validator, or security gate is passed. Any network client can reach plugin HTTP endpoints even when the gateway token/password is configured.

**Affected code:**
- `src/gateway/server/plugins-http.ts:12-61` - `createGatewayPluginRequestHandler` with no auth check before `route.handler(req, res)` dispatch

**Verification:**
- No imports for `authorizeGatewayConnect` or `resolvedAuth` validation in the file
- Other endpoints (OpenAI, tools-invoke, open-responses) DO call `authorizeGatewayConnect`
- Plugin HTTP dispatch at `server-http.ts:498` (now gateway-auth protected for `/api/channels/` routes at `:484-497`, PR #13719)

### #6609: Browser Bridge Server Optional Authentication

**Severity:** HIGH (CVSS 7.7)
**CWE:** CWE-306 (Missing Authentication for Critical Function)

**Vulnerability:** Browser bridge server's authentication token is optional. When started without `authToken`, all browser automation endpoints are exposed without authentication.

**Affected code:**
- `src/browser/bridge-server.ts:24` - `authToken?: string` (optional parameter)
- `src/browser/bridge-server.ts:33-42` - Auth middleware only added IF token provided

**Verification:**
- `startBrowserBridgeServer` called at `src/agents/sandbox/browser.ts:192-200` WITHOUT `authToken` parameter
- Browser bridge runs unauthenticated in sandbox context

### #8696: Playwright Download Path Traversal

**Severity:** HIGH (CVSS 8.8)
**CWE:** CWE-22 (Path Traversal)

**Vulnerability:** Playwright download helpers use the server's suggested filename without sanitization, allowing path traversal writes outside `/tmp/openclaw/downloads`. A malicious `Content-Disposition` header like `filename=../../../../../etc/passwd` can write files to arbitrary locations.

**Affected code:**
- `src/browser/pw-tools-core.downloads.ts:20-24` - `buildTempDownloadPath` uses `fileName.trim()` with no `path.basename()` or traversal stripping
- `src/browser/pw-tools-core.downloads.ts:182-183` - `suggestedFilename()` passed directly without validation

**Verification:**
- No calls to `path.basename()` or path sanitization in the download path construction
- `path.join()` does not prevent traversal — it concatenates path segments

### #9512: Skill Download Archive Path Traversal

**Severity:** HIGH (CVSS 7.6)
**CWE:** CWE-22 (Path Traversal)

**Vulnerability:** `skills.install` with download installer extracts archives via system `tar`/`unzip` without validating entry paths. Malicious archives with `../` sequences can write files outside the target directory (Zip Slip attack).

**Affected code:**
- `src/agents/skills-install.ts:255-279` - `extractArchive` function
- `src/agents/skills-install.ts:263-268` - zip: `["unzip", "-q", archivePath, "-d", targetDir]` with no path filter
- `src/agents/skills-install.ts:271-278` - tar: `["tar", "xf", archivePath, "-C", targetDir]` with no path filter

**Verification:**
- No entry path validation, `tar --transform`, or filter callback found
- Directly related to ClawHavoc campaign (341 malicious skills could exploit this)

### #9517: Gateway Canvas Host Auth Bypass

**Severity:** HIGH (CVSS 7.5)
**CWE:** CWE-862 (Missing Authorization)

**Vulnerability:** Gateway HTTP server serves Canvas host and A2UI endpoints without enforcing gateway auth, allowing unauthenticated access to canvas files.

**Affected code:**
- `src/gateway/server-http.ts:527-539` - Canvas/A2UI handler dispatch (now auth-wrapped via `authorizeCanvasRequest()` at `:109-150`, PR #9518)
- `src/gateway/server-http.ts:589-619` - WebSocket upgrade for canvas (now auth-wrapped via `authorizeCanvasRequest()` at `:596`, PR #9518)

**Verification:**
- No `authorizeGatewayConnect` call before `canvasHost.handleHttpRequest(req, res)`
- Other endpoints in the same file DO call authorization methods

### #5120: Webhook Token Accepted via Query Parameters

**Status: FIXED** in PR [#9436](https://github.com/openclaw/openclaw/pull/9436)

**Severity:** ~~MEDIUM~~ FIXED
**CWE:** CWE-598 (Sensitive Query Strings)

**Vulnerability:** Webhook endpoint accepted authentication tokens via URL query parameters, causing credential leakage through logs, browser history, and Referer headers.

**Fix:** Query token extraction removed entirely from `src/gateway/hooks.ts`. `extractHookToken()` now only accepts `Authorization: Bearer` header and `X-OpenClaw-Token` header. Server returns HTTP 400 when `?token=` is present (`src/gateway/server-http.ts:236-243`).

### #4949: Browser Control Server DNS Rebinding

**Severity:** HIGH
**CWE:** CWE-350 (Reliance on Reverse DNS Resolution)

**Vulnerability:** Browser control server binds to `127.0.0.1` but performs no Host header validation. DNS rebinding attacks can bypass localhost restriction to reach browser automation endpoints from a remote origin.

**Affected code:**
- `src/browser/server.ts:133` - binds to `"127.0.0.1"` with auth middleware at `:117-124`
- `src/browser/server.ts:79-166` - auth now required via `isAuthorizedBrowserRequest()` (commit `9230a2ae1`), but still no Host header or origin validation

**Partial mitigation (Feb 13 sync 5):** Commit `9230a2ae1` adds bearer token / password auth middleware to all browser control HTTP routes. DNS rebinding still possible but requests now need valid authentication credentials, significantly raising the bar. New `browser.control_no_auth` audit check flags when no auth is configured.

### #4950: Arbitrary JS Execution via Browser Evaluate (Default On)

**Severity:** HIGH
**CWE:** CWE-94 (Improper Control of Code Generation)

**Vulnerability:** Browser evaluate tool is enabled by default (`DEFAULT_BROWSER_EVALUATE_ENABLED = true`), allowing execution of arbitrary JavaScript in the browser context without sandboxing.

**Affected code:**
- `src/browser/constants.ts:2` - `DEFAULT_BROWSER_EVALUATE_ENABLED = true`
- `src/browser/pw-tools-core.interactions.ts:219-268` - `evaluateViaPlaywright()` passes JS directly to `page.evaluate()` without sandbox

**Note:** Config flag exists (commit `78f0bc3`) but defaults to enabled. Users must explicitly opt out.

### #4995: iMessage dmPolicy Auto-Responds with Pairing Codes

**Severity:** HIGH
**CWE:** CWE-200 (Exposure of Sensitive Information)

**Vulnerability:** Default `dmPolicy` is `"pairing"`, which automatically responds to unknown contacts with valid pairing codes. Any sender can receive a pairing code without owner verification.

**Affected code:**
- `src/imessage/monitor/monitor-provider.ts:184` - default `dmPolicy` is `"pairing"`
- `src/imessage/monitor/monitor-provider.ts:342-381` - auto-responds with pairing code to unknown contacts

### #5122: readJsonBody() Slowloris DoS (No Read Timeout)

**Severity:** MEDIUM
**CWE:** CWE-400 (Uncontrolled Resource Consumption)

**Vulnerability:** `readJsonBody()` has a body size limit but no read timeout. An attacker can hold connections open indefinitely by sending data one byte at a time (Slowloris attack).

**Affected code:** `src/gateway/hooks.ts:177-194` - size limit present, timeout absent.

### #5123: ReDoS in Session Filter Regex

**Severity:** MEDIUM
**CWE:** CWE-1333 (Inefficient Regular Expression Complexity)

**Vulnerability:** User-supplied strings are compiled into regexes via `new RegExp()` without safeguards. Malicious patterns can cause catastrophic backtracking.

**Affected code:**
- `src/infra/exec-approval-forwarder.ts:70-77` - `matchSessionFilter()` compiles arbitrary user regex
- `src/discord/monitor/exec-approvals.ts:235-239` - same vulnerable pattern

### #5124: ReDoS in Log Redaction Patterns

**Severity:** MEDIUM
**CWE:** CWE-1333 (Inefficient Regular Expression Complexity)

**Vulnerability:** Log redaction `parsePattern()` compiles arbitrary regex patterns that could cause catastrophic backtracking on large log entries.

**Affected code:** `src/logging/redact.ts:47-61` - `parsePattern()` compiles arbitrary regex for log processing.

### #6021: Timing Attack in Non-Gateway Token Comparisons

**Status: WONTFIX** -- closed upstream as NOT_PLANNED 2026-02-13. Partially mitigated locally.

**Severity:** MEDIUM (WONTFIX)
**CWE:** CWE-208 (Observable Timing Discrepancy)

**Vulnerability:** Gateway auth correctly uses `safeEqual` (timing-safe), but hook tokens, node pairing, and device pairing use direct `===`/`!==` comparisons vulnerable to timing attacks.

**Affected code:**
- `src/security/secret-equal.ts:3-16` - `safeEqualSecret` uses `timingSafeEqual` (correct)
- `src/gateway/server-http.ts:254` - hook token now uses `safeEqualSecret()` (fixed in Feb 13 sync 4, commit `113ebfd6a`)
- `src/infra/node-pairing.ts:277` - node token uses direct `===` (vulnerable)
- `src/infra/device-pairing.ts:435` - device token now uses `safeEqualSecret()` (fixed in Feb 13 sync 4, commit `113ebfd6a`)

### #6606: Telegram Webhook Binds to 0.0.0.0 with Optional Secret

**Status: WONTFIX** -- closed upstream as NOT_PLANNED 2026-02-13. Vulnerability still present in local code.

**Severity:** HIGH (WONTFIX)
**CWE:** CWE-668 (Exposure of Resource to Wrong Sphere)

**Vulnerability:** Telegram webhook server defaults to binding on `0.0.0.0` (all interfaces), and the webhook secret token is optional. Without a secret, any network client can send fake webhook events.

**Affected code:**
- `src/telegram/webhook.ts:26` - defaults to `0.0.0.0` binding
- `src/telegram/webhook.ts:36` - `webhookSecret` is optional
- `src/telegram/webhook.ts:46-48` - secret validation only IF configured

### #7862: Session Transcripts 644 Instead of 600

**Severity:** MEDIUM
**CWE:** CWE-732 (Incorrect Permission Assignment for Critical Resource)

**Vulnerability:** Session transcript `.jsonl` files are created with default permissions (0o644) instead of restrictive permissions (0o600). Other local users can read session data containing tool calls, messages, and potentially secrets.

**Affected code:**
- `src/auto-reply/reply/session.ts:87` - `writeFileSync` with no explicit mode
- `src/gateway/server-methods/chat.ts:73,81` - transcript file creation with no explicit mode

**Note:** `src/security/fix.ts:442,451` applies 0o600 to `auth-profiles.json` and `sessions.json` but NOT individual `.jsonl` transcript files.

### #8516: Browser Download/Trace Endpoints Arbitrary File Write

**Severity:** HIGH
**CWE:** CWE-22 (Path Traversal)

**Vulnerability:** Browser download and trace endpoints accept arbitrary file paths without validation or authentication. POST `/download` and `/trace/stop` pass `body.path` directly to file system operations.

**Affected code:**
- `src/browser/routes/agent.act.ts:447-480` - POST `/download` passes `body.path` directly
- `src/browser/routes/agent.debug.ts:119-150` - POST `/trace/stop` uses `body.path` without validation

**Note:** Related to #8696 (Playwright download path traversal) but affects different endpoints.

### #8586: Configurable Bypass Allows Unrestricted Command Exec

**Severity:** HIGH
**CWE:** CWE-269 (Improper Privilege Management)

**Vulnerability:** When `elevatedMode=full` is configured, all security controls on command execution are bypassed. The `bypassApprovals` flag skips `resolveExecApprovals` entirely, allowing any command without user confirmation.

**Affected code:**
- `src/agents/bash-tools.exec.ts:199-207` — `elevatedMode` resolution; `elevatedMode=full` sets security to "full", ask to "off"
- `src/agents/bash-tools.exec.ts:269-271` — `bypassApprovals` flag; skips all approval checks when `elevatedMode === "full"`

### #8590: Status Endpoint Exposes Sensitive Internal Info

**Severity:** HIGH
**CWE:** CWE-200 (Exposure of Sensitive Information)

**Vulnerability:** Gateway status/health endpoint returns unredacted internal information including file paths, session IDs, agent IDs, and model configuration to any connected client.

**Affected code:**
- `src/gateway/server-methods/health.ts:28-31` - returns full unredacted `getStatusSummary()`
- `src/commands/status.summary.ts:134-195` - exposes paths, session IDs, agent IDs, model configs

### #8591: Env Vars Exposed via Shell Commands

**Status: WONTFIX** -- closed upstream as NOT_PLANNED 2026-02-13. Vulnerability still present in local code.

**Severity:** HIGH (WONTFIX)
**CWE:** CWE-526 (Exposure of Sensitive Information Through Environmental Variables)

**Vulnerability:** Full `process.env` is passed as the base environment to child processes. An agent can run `env` or `printenv` to dump all environment variables, including API keys and secrets.

**Affected code:**
- `src/agents/bash-tools.exec.ts:290,298` — full `process.env` passed to child spawn via `coerceEnv(process.env)` + `{ ...baseEnv, ...params.env }`
- `src/agents/bash-tools.exec-runtime.ts:32-50` — `DANGEROUS_HOST_ENV_VARS` blocks injection INTO env, but doesn't filter what children can READ

### #8592: No Detection of Encoded/Obfuscated Commands

**Severity:** MEDIUM (partially affected)
**CWE:** CWE-116 (Improper Encoding or Escaping)

**Vulnerability:** `isSafeExecutableValue()` validates executable names against an allowlist but does not detect base64-encoded, hex-encoded, or otherwise obfuscated command arguments.

**Affected code:** `src/infra/exec-safety.ts:1-44` - validates executable names only; obfuscated arguments pass through.

### #8593: chat.send Handler Lacks Input Length Validation

**Severity:** MEDIUM (partially affected)
**CWE:** CWE-20 (Improper Input Validation)

**Vulnerability:** `ChatSendParamsSchema` validates structure but the message field has no `maxLength` constraint. Extremely large messages could cause resource exhaustion.

**Affected code:** `src/gateway/protocol/schema/logs-chat.ts:34-45` - schema exists but lacks length limits on message field.

### #8776: soul-evil Hook Silently Hijacks Agent

**Status: FIXED** — closed via PR [#14757](https://github.com/openclaw/openclaw/pull/14757) (Feb 13 sync 1)

**Severity:** ~~HIGH~~ FIXED
**CWE:** CWE-506 (Embedded Malicious Code)

**Vulnerability:** The `soul-evil` hook shipped bundled with OpenClaw and could silently replace the agent's SOUL.md (system prompt) content. When activated, it overrode the agent's personality and behavior without explicit user notification.

**Fix:** Complete removal of soul-evil hook. Deleted `src/hooks/soul-evil.ts` (280 lines), `src/hooks/soul-evil.test.ts` (252 lines), bundled handler directory (`src/hooks/bundled/soul-evil/`), documentation (`docs/hooks/soul-evil.md`), and package.json references (-981 lines total). Thanks @Imccccc.

### #9007: Google Places URL Path Interpolation (Skill, Not Core)

**Severity:** LOW (partially affected)
**CWE:** CWE-918 (Server-Side Request Forgery)

**Vulnerability:** Google Places API URL construction interpolates `place_id` without sanitization. Located in the optional `local-places` skill, not core code.

**Affected code:**
- `skills/local-places/src/local_places/google_places.py:238` - `place_id` interpolated into URL
- `skills/local-places/src/local_places/main.py:52-54` - FastAPI route passes `place_id` directly

### #9065: ~/.openclaw Group-Writable After sudo Install

**Severity:** LOW (partially affected)
**CWE:** CWE-276 (Incorrect Default Permissions)

**Vulnerability:** Code correctly uses `mode: 0o700` for directory creation (`src/config/io.ts:720`), but when installed via `sudo`, the directory inherits root ownership. Subsequent user-space operations may create group-writable files.

**Note:** This is an operational issue (sudo usage), not a code bug. `src/security/audit.ts:160-177` already detects group-writable state directories.

### #9435: Gateway Auth Token Exposed in URL Query Params

**Status: FIXED** in PR [#9436](https://github.com/openclaw/openclaw/pull/9436)

**Severity:** ~~HIGH~~ FIXED
**CWE:** CWE-598 (Sensitive Query Strings)

**Vulnerability:** Gateway authentication tokens were passed via URL query parameters (`?token=...`) in dashboard and onboarding flows, exposing credentials through logs, browser history, and Referer headers.

**Fix:** Query token acceptance completely removed. `extractHookToken()` in `src/gateway/hooks.ts:158-174` no longer reads `url.searchParams`. `src/commands/dashboard.ts` no longer constructs `?token=` URLs. `src/commands/onboard-helpers.ts` no longer passes token in URL. Server now returns HTTP 400 when `?token=` is present (`src/gateway/server-http.ts:243-249`).

### #9627: Config Secrets Exposed in JSON After Update/Doctor

**Severity:** HIGH
**CWE:** CWE-312 (Cleartext Storage of Sensitive Information)

**Vulnerability:** When `openclaw doctor` or `openclaw config set` writes the config file, environment variable references (`${VAR}`) are resolved to plaintext values. The write-back serializes resolved secrets to disk in cleartext JSON, destroying the original `${VAR}` references.

**Affected code:**
- `src/config/io.ts:681-781` - `writeConfigFile` now preserves env var references for unchanged paths (commit `f59df9589`), but resolved values still leak for paths that change
- `src/config/env-substitution.ts:83-89` - `substituteString` is a one-way transformation
- `src/commands/doctor.ts:285` - `writeConfigFile(cfg)` writes env-resolved config back to disk

### #9813: Gateway Expands ${ENV_VAR} on Meta Writeback (DUPLICATE of #9627)

**Severity:** HIGH (DUPLICATE)
**CWE:** CWE-312 (Cleartext Storage of Sensitive Information)

**Vulnerability:** Same root cause as #9627. When the gateway updates `meta.lastTouchedAt` in the config file, the writeback path resolves `${ENV_VAR}` references to plaintext values, destroying the original references.

**Affected code:**
- `src/config/io.ts:681-781` - `writeConfigFile` now preserves env var references for unchanged paths (commit `f59df9589`), partially mitigating meta writeback expansion

**Our analysis:** This is the same `writeConfigFile` code path documented in #9627. The trigger differs (gateway meta writeback vs `doctor`/`config set`), but the underlying bug — env var expansion on write — is identical. Fix for #9627 would resolve this as well.

### #9795: sanitizeMimeType Regex Not End-Anchored (By Design)

**Severity:** LOW/INFORMATIONAL
**CWE:** N/A

**Vulnerability claimed:** The `sanitizeMimeType` regex `/^([\w-]+\/[\w.+-]+)/` is not end-anchored, potentially allowing MIME type confusion.

**Affected code:**
- `src/media-understanding/apply.ts:96-106` - `sanitizeMimeType` function

**Our analysis:** The unanchored end is **standard MIME parsing behavior**. MIME types can include parameters like `; charset=utf-8` which the regex correctly discards by capturing only `type/subtype`. The function also calls `.toLowerCase()` (line 100) before the regex match, handling case-insensitivity. The reporter's suggested fix (adding `$` anchor) would break legitimate MIME types with parameters. This is a Qodo AI automated finding that misidentifies correct behavior as a vulnerability.

### #9792: validateHostEnv Skips baseEnv (By Design)

**Severity:** INVALID
**CWE:** N/A

**Vulnerability claimed:** `validateHostEnv` only validates agent-supplied `params.env` but not the host's own `baseEnv`, potentially allowing dangerous environment variables.

**Affected code:**
- `src/agents/bash-tools.exec-runtime.ts:54` (definition) + `src/agents/bash-tools.exec.ts:295` (call) — validation scoped to `params.env` only

**Our analysis:** `baseEnv = coerceEnv(process.env)` is the **host's own environment**, not untrusted input. The code comment at line 969 states: "We validate BEFORE merging to prevent any dangerous vars from entering the stream." Validating `baseEnv` would **break the gateway** — the host always has `PATH` set, which `validateHostEnv` explicitly rejects (it's designed to block agents from injecting `PATH` overrides). The validation boundary is intentionally scoped to untrusted agent-supplied variables. This is a Qodo AI automated finding.

### #9791: Fullwidth Character Markers Bypass (Fold Is Length-Preserving)

**Severity:** INVALID
**CWE:** N/A

**Vulnerability claimed:** Fullwidth Unicode characters (e.g., `＜` U+FF1C) could bypass marker detection in `replaceMarkers`, causing index misalignment when mapping between folded and original strings.

**Affected code:**
- `src/security/external-content.ts:127-167` - `replaceMarkers` and `foldMarkerChar`

**Our analysis:** `foldMarkerChar` maps each fullwidth character to a single ASCII character (`\uFF21`→`A`, `\uFF1C`→`<`, etc.). Both fullwidth characters and their ASCII replacements are **single BMP UTF-16 code units**, so the fold is **length-preserving**. Indices from `pattern.regex.exec(folded)` map correctly back to `content.slice()` positions on the original string. The reporter suggests "perform all operations on folded string" — this would **lose original content** between markers, which is the opposite of correct behavior. This is a Qodo AI automated finding.

### #9667: JWT Token Verification Incomplete (File Does Not Exist)

**Severity:** INVALID
**CWE:** N/A

**Vulnerability claimed:** Incomplete JWT token verification in `src/auth/jwt.ts` allows authentication bypass.

**Our analysis:** The file `src/auth/jwt.ts` **does not exist** in the codebase. Grep confirms zero JWT-related files in the entire `src/` directory. This was filed by an "automated bug hunting system" with a generic proposed fix referencing a nonexistent file. Entirely fabricated.

### #4940: commands.restart Bypass via Exec Tool

**Severity:** MEDIUM
**CWE:** CWE-863 (Incorrect Authorization)

**Vulnerability:** The gateway tool correctly checks `commands.restart=true` before allowing restart actions (`src/agents/tools/gateway-tool.ts:77-80`), but the exec tool can run `openclaw gateway restart` without checking this config flag. The exec approval system (`src/infra/exec-approvals.ts`) only validates executable paths against allowlist patterns, not the semantic meaning of commands.

**Affected code:**
- `src/agents/tools/gateway-tool.ts:77-80` - correctly checks `commands.restart` (this is the SECURE path)
- `src/agents/bash-tools.exec.ts` - no check for `commands.restart` (BYPASS path)
- `src/infra/exec-approvals.ts` - no command-semantic filtering

**Note:** Requires agent to have exec access (security mode `allowlist` with `openclaw` allowlisted, or `full`).

### #8588: Sensitive Config Files Accessible When Sandbox Is Home Dir

**Severity:** MEDIUM (partially affected)
**CWE:** CWE-552 (Files or Directories Accessible to External Parties)

**Vulnerability:** When users configure `workspace: "~"` (home directory) with `workspaceAccess: "rw"`, the sandbox root becomes the home directory, exposing `~/.openclaw/openclaw.json` (API keys, bot tokens) and `~/.openclaw/credentials/` to the agent. The sandbox path validation only prevents path escape, with no exclusion of sensitive subdirectories.

**Affected code:**
- `src/agents/sandbox/context.ts:39-46` - `workspaceAccess === "rw"` uses `agentWorkspaceDir` as sandbox root
- `src/agents/sandbox-paths.ts:33-47` - path escape protection only, no sensitive dir exclusion
- `src/agents/sandbox/config.ts:149` - default `workspaceAccess: "none"` (safe)

**Note:** Default configuration is SAFE. Only manifests with explicit non-default workspace + rw config.

### #8589: Sandbox File Read Lacks Content Filtering

**Severity:** LOW
**CWE:** CWE-200 (Exposure of Sensitive Information)

**Vulnerability:** The file read tool returns raw file contents without content filtering or sensitive-data redaction. Comprehensive redaction patterns exist in `src/logging/redact.ts` (API keys, tokens, PEM keys, etc.) but are only used for log/UI display, not applied to file read results returned to the agent.

**Affected code:**
- `src/agents/pi-tools.read.ts:286-302` - read tool returns raw content, processes only image MIME
- `src/logging/redact.ts:13-36,124-137` - redaction patterns exist but NOT applied to read results

**Note:** Sandbox path enforcement is the primary control. This is a defense-in-depth gap, not a boundary breach. Low severity because the sandbox boundary itself is correctly enforced.

### #8594: No Rate Limiting on Gateway Endpoints

**Status: WONTFIX** -- closed upstream as NOT_PLANNED 2026-02-13. Vulnerability still present in local code.

**Severity:** MEDIUM (WONTFIX)
**CWE:** CWE-770 (Allocation of Resources Without Limits or Throttling)

**Vulnerability:** The gateway server lacks rate limiting on all RPC, HTTP, and WebSocket endpoints. No per-client, per-IP, or per-connection request throttling exists. Authenticated clients can send unlimited requests.

**Affected code:**
- `src/gateway/server-constants.ts` - payload size limits only (`MAX_PAYLOAD_BYTES=512KB`, `MAX_BUFFERED_BYTES=1.5MB`), no rate constants
- `src/gateway/server-http.ts` - no rate limiting middleware
- `src/gateway/server/ws-connection/message-handler.ts` - no per-message rate limiting

**Existing protections (not rate limiting):**
- Authentication required (not anonymous)
- Payload size limits (512KB frame, 256KB HTTP body)
- Deduplication cache (prevents duplicate processing but NOT request frequency)

### #10324: Memory Index Multi-Write Lacks Transactions

**Severity:** MEDIUM (CVSS 6.8)
**CWE:** CWE-362 (Concurrent Execution Using Shared Resource with Improper Synchronization)

**Vulnerability:** The `indexFile()` method performs multiple DELETE + INSERT operations across chunks, vector, FTS, and files tables without wrapping them in a database transaction. A crash or concurrent write mid-operation leaves the memory index in an inconsistent state (orphaned vectors, missing chunks, stale file records).

**Affected code:**
- `src/memory/manager.ts:2198-2301` — `indexFile()` performs 5+ SQL operations (DELETE vector :2222, DELETE FTS :2230, DELETE chunks :2234, INSERT loop :2237-2289, UPSERT files :2291-2300) with **no BEGIN/COMMIT**
- Contrast: `src/memory/manager.ts:745,757` correctly uses `BEGIN`/`COMMIT` for similar batch operations

### #10326: Child Process stop() Lacks SIGKILL Escalation

**Severity:** MEDIUM (CVSS 5.5)
**CWE:** CWE-404 (Improper Resource Shutdown or Release)

**Vulnerability:** Child process termination in iMessage and Signal daemons sends only SIGTERM with no fallback to SIGKILL. Misbehaving or hung child processes can remain alive indefinitely, consuming resources.

**Affected code:**
- `src/imessage/client.ts:110-131` — `stop()` sends SIGTERM at :125 after 500ms timeout; `Promise.race` at :120 resolves regardless, but never force-kills
- `src/signal/daemon.ts:96-100` — `stop()` sends SIGTERM at :98; fire-and-forget with no timeout or SIGKILL fallback

### #10330: TOCTOU Race in Device Auth Token Storage

**Severity:** MEDIUM (CVSS 6.2)
**CWE:** CWE-367 (Time-of-check Time-of-use Race Condition)

**Vulnerability:** `storeDeviceAuthToken()` reads the auth store from disk, modifies the in-memory object, and writes back without any file locking. Two concurrent authentication flows can race, with the second overwriting the first's token.

**Affected code:**
- `src/infra/device-auth-store.ts:92-119` — read at :100 (`readStore`), mutate :102-116, write at :117 (`writeStore`) with **no lock between read and write**
- Called from `src/gateway/client.ts:254` during device authentication

### #10331: Session Store Stale Cache Inside Write Lock

**Severity:** MEDIUM (CVSS 5.9)
**CWE:** CWE-662 (Improper Synchronization)

**Vulnerability:** Two session store write methods call `loadSessionStore()` without `skipCache: true`, reading stale cached data even though they hold the write lock. Concurrent requests to the same session can silently lose metadata updates.

**Affected code:**
- `src/config/sessions/store.ts:667` — `updateSessionStoreEntry()` inside `withSessionStoreLock()` calls `loadSessionStore(storePath)` **without `skipCache: true`**
- `src/config/sessions/store.ts:729` — `updateLastRoute()` same pattern
- `src/config/sessions/store.ts:575` — `updateSessionStore()` NOW correctly uses `{ skipCache: true }` (bug partially fixed in session pruning refactor)

**Impact:** 8 callers in hot paths (agent runner, channels, Slack, LINE, web) can lose session metadata updates under concurrent load.

### #10333: BlueBubbles Filename Multipart Header Injection

**Severity:** MEDIUM (CVSS 5.4)
**CWE:** CWE-93 (Improper Neutralization of CRLF Sequences in HTTP Headers)

**Vulnerability:** BlueBubbles attachment handling uses `path.basename()` as sole filename sanitization, which strips directory components but preserves `"`, `\r`, `\n`, and other characters that can inject into Content-Disposition headers.

**Affected code:**
- `extensions/bluebubbles/src/attachments.ts:26-30` — `sanitizeFilename()` uses only `path.basename()` at :28
- `extensions/bluebubbles/src/attachments.ts:224-228` — `addFile()` interpolates filename unescaped into `Content-Disposition: form-data; name="${name}"; filename="${fileName}"` at :227
- `extensions/bluebubbles/src/chat.ts:340-342` — constructs Content-Disposition header with `filename="${filename}"` at :342 with **no sanitization at all**

### #10927: Random IDs for External Content Wrapper Tags (Enhancement)

**Category:** ENHANCEMENT (defense-in-depth)
**CWE:** N/A

**Proposal:** Add random 16-char IDs to external content wrapper tags (`<<<EXTERNAL_UNTRUSTED_CONTENT id="a7f3b2c1...">>>`) to prevent tag spoofing by malicious content.

**Current defense:** `replaceMarkers()` at `src/security/external-content.ts:127-167` already sanitizes injected `<<<EXTERNAL_UNTRUSTED_CONTENT>>>` tags (case-insensitive, including fullwidth Unicode variants) to `[[MARKER_SANITIZED]]`. The existing defense is functional; random IDs would add defense-in-depth and improve content correlation for debugging.

**Related:** #8027 (web_fetch hidden text prompt injection)

### #10890: RFC: Skill Security Framework (Enhancement)

**Category:** ENHANCEMENT (architecture proposal)
**CWE:** N/A

**Proposal:** Phased skill security framework:
- Phase 1: `openclaw skills audit` CLI, permission manifests, hash verification, install warnings
- Phase 2: Author verification, skill signing, version pinning
- Phase 3: Runtime sandboxing, tool allowlists per skill, anomaly detection

**Relevance:** Directly addresses the attack surface documented in #9512 (skill archive path traversal) and the ClawHavoc campaign. Proposes Deno-style deny-by-default permissions for the skill system. Not a vulnerability report; comprehensive RFC for skill ecosystem security.

### #11900: Context Files Loaded for All Senders Regardless of IsOwner

**Vulnerability:** Bootstrap context files (USER.md, SOUL.md, etc.) are loaded for every sender, including non-owners on public channels. The `resolveBootstrapContextForRun()` function has no `senderIsOwner` parameter.
**CWE:** CWE-200 (Exposure of Sensitive Information)

**Affected code:**
- `src/agents/bootstrap-files.ts:43-60` — `resolveBootstrapContextForRun()` loads all bootstrap files unconditionally
- `src/agents/pi-embedded-runner/run/attempt.ts:192` — calls `resolveBootstrapContextForRun()` without `senderIsOwner`
- `src/agents/pi-embedded-runner/run/attempt.ts:229` — `senderIsOwner` only passed to `createOpenClawCodingTools()` for tool gating

**Impact:** Non-owner senders on public channels receive responses shaped by the owner's personal context files (personality, preferences, private notes). The content is not directly exposed but indirectly leaks through response behavior. Tool access is correctly gated by `senderIsOwner`, but context/personality files are not.

### #11832: Per-Agent tools.exec Config Not Applied

**Vulnerability:** Per-agent `tools.exec` configuration (host, security, ask, node) is silently ignored. Agents run with global exec defaults regardless of per-agent settings.
**CWE:** CWE-269 (Improper Privilege Management)

**Affected code:**
- `src/auto-reply/reply/get-reply-directives.ts:66-81` — `resolveExecOverrides()` reads from `directives` (inline `!exec=docker`) and `sessionEntry` only
- `src/auto-reply/reply/get-reply-directives.ts:93` — `agentCfg: AgentDefaults` is in scope but not consulted for exec settings
- `src/agents/pi-embedded-runner/run/attempt.ts:212-216` — `execOverrides` passed to tool creation, but populated only from directives/session

**Impact:** If an operator configures per-agent exec restrictions (e.g., `agents.mybot.tools.exec.host = "docker"` for sandboxed execution), those restrictions are silently ignored. The agent runs with global exec defaults. Global config still applies; only per-agent overrides are lost.

### #11945: config.patch Bypasses commands.restart Restriction

**Status: WONTFIX** -- closed upstream as NOT_PLANNED 2026-02-13. Vulnerability still present in local code.

**Severity:** HIGH (WONTFIX)
**CWE:** CWE-863 (Incorrect Authorization)

**Vulnerability:** The `config.patch` gateway RPC method writes arbitrary config changes and triggers an automatic SIGUSR1 restart without checking `commands.restart`. The restart action correctly gates on `commands.restart`, but `config.patch` bypasses this by pre-authorizing the SIGUSR1 via `authorizeGatewaySigusr1Restart()`.

**Attack surface:** An agent with config.patch access can:
1. Disable gateway auth (`gateway.auth.mode`)
2. Bind to 0.0.0.0 (expose loopback-only gateway)
3. Add attacker-controlled channels
4. Swap the model to an attacker-controlled endpoint
5. Change workspace paths to sensitive directories
6. Modify auth profiles/API keys

All changes take effect immediately via automatic restart.

**Affected code:**
- `src/gateway/server-methods/config.ts:296,330` — writes config then calls `scheduleGatewaySigusr1Restart()` with NO `commands.restart` check
- `src/infra/restart.ts:192` — `authorizeGatewaySigusr1Restart(delayMs)` pre-authorizes the SIGUSR1 signal
- `src/cli/gateway-cli/run-loop.ts:104-105` — `consumeGatewaySigusr1RestartAuthorization()` returns true (pre-authorized), bypassing `isGatewaySigusr1RestartExternallyAllowed()`
- **Contrast:** `src/agents/tools/gateway-tool.ts:78` — the explicit `restart` action correctly checks `commands.restart`

**No key-level authorization:** Config validation is structural (JSON schema) only. No allowlist/denylist restricts which keys agents may modify.

### #12571: Session Isolation Leak in Cron Jobs After Extended Runtime

**Severity:** MEDIUM
**CWE:** CWE-362 (Race Condition) / CWE-404 (Improper Resource Shutdown)

**Vulnerability:** After ~24 hours of continuous operation, cron jobs configured with `sessionTarget: "isolated"` begin leaking messages into the main session. The reporter observed ~165 successful isolated runs before the leak began, with 5 prompt injection payloads (agent identity overrides) delivered to the main agent over 4 hours.

**Affected code:**
- `src/cron/service/jobs.ts` — cron job execution with `sessionTarget` handling
- `src/agents/tools/cron-tool.ts` — cron tool with `sessionTarget: "isolated"` support
- Session isolation code paths in 25+ files

**Root cause hypothesis:** Session pool exhaustion, session ID collision after rollover, isolation context corruption, or WebSocket routing table degradation. The consistent ~24-hour timeframe suggests a periodic cleanup (GC, connection pool reset) that breaks isolation context.

**Impact:** Prompt injection via session leak — isolated agent identity payloads delivered to the wrong session. Requires specific cron config + extended runtime + multiple concurrent isolated sessions.

### #13683: CLI `config get` Returns Unredacted Secrets to Sandboxed Agents

**Severity:** HIGH
**CWE:** CWE-200 (Exposure of Sensitive Information)

**Vulnerability:** The CLI `openclaw config get` command reads and outputs resolved config values (including secrets from `${ENV_VAR}` substitution) without applying the redaction system. The gateway RPC `config.get` handler correctly redacts via `redactConfigSnapshot()`, but the CLI path bypasses this entirely. A sandboxed agent with exec access can extract any API key configured via env var substitution.

**Affected code:**
- `src/cli/config-cli.ts:269-270` — `loadValidConfig()` returns resolved `snapshot.config`; `getAtPath()` reads values directly
- `src/cli/config-cli.ts:277-288` — output paths (`defaultRuntime.log`) emit unredacted values

**Correct implementation (for comparison):**
- `src/gateway/server-methods/config.ts:107-108` — RPC handler calls `redactConfigSnapshot(snapshot)` before `respond()`
- `src/config/redact-snapshot.ts:273-275` — `redactConfigObject()` is exported and available for use in CLI

**Relationship to existing issues:**
- #9627: Config *write-back* destroys `${VAR}` references (different attack: disk persistence)
- #5995: Secrets in session *transcripts* (different attack: log files)
- #8591: Env vars via `env`/`printenv` (related: alternate exfiltration path via `process.env`)

**Fix:** Apply `redactConfigObject()` to the value before output in CLI `config get`, or use `redactConfigSnapshot()` on the entire snapshot and read from the redacted copy.

### #13786: BlueBubbles Webhook Auth Bypass via Loopback Proxy Trust

**Status: FIXED** — fixed in PR [#13787](https://github.com/openclaw/openclaw/pull/13787) (Feb 13 sync 1)

**Severity:** ~~HIGH~~ FIXED (was CVSS 8.6)
**CWE:** CWE-288 (Authentication Bypass Using an Alternate Path or Channel)

**Vulnerability:** The BlueBubbles webhook handler unconditionally trusted loopback remote addresses, bypassing the shared-secret check. In same-host reverse-proxy deployments, all external traffic arrived as `127.0.0.1`, so attackers could inject webhook events without knowing the BlueBubbles password.

**Fix:** Removed the loopback remoteAddress bypass from `extensions/bluebubbles/src/monitor.ts`. All requests now require password authentication regardless of source IP. Test fixtures updated to require authenticated webhooks.

### #13718: Unauthenticated Nostr Profile API Allows Remote Config Tampering

**Status: FIXED** — fixed in PR [#13719](https://github.com/openclaw/openclaw/pull/13719) (Feb 13 sync 1)

**Severity:** ~~HIGH~~ FIXED (was CVSS 8.6)
**CWE:** CWE-306 (Missing Authentication for Critical Function)

**Vulnerability:** The Nostr plugin registered HTTP endpoints for profile management (GET/PUT/POST on `/api/channels/nostr/:accountId/profile`) that accepted unauthenticated requests. The PUT path wrote attacker-controlled profile data directly to the gateway config file and triggered relay publish operations.

**Fix:** Gateway now requires `authorizeGatewayConnect` for all `/api/channels/` plugin HTTP routes (`src/gateway/server-http.ts:484-497`). Channel plugin endpoints are gateway-auth protected by default; non-channel plugin routes remain plugin-owned. New `server.plugin-http-auth.test.ts` (174 lines). Also adds UI-side Nostr profile management in `ui/src/ui/app-channels.ts` (+23 lines).

### #13937: HTML Not Escaped in Control UI Webchat (XSS)

**Status: FIXED** — closed as COMPLETED upstream 2026-02-11

**Severity:** ~~MEDIUM~~ FIXED
**CWE:** CWE-79 (Cross-Site Scripting)

**Vulnerability:** When HTML content is posted as a message in the Control UI webchat, it was rendered as live HTML rather than escaped as plain text. User-confirmed with screenshot showing rendered `<h1>`, `<p>` tags from a pasted HTML error page.

**Affected code:** The webchat markdown rendering pipeline in `ui/` passed raw HTML through without sanitization (per CommonMark spec, which allows inline HTML). No explicit `innerHTML`/`dangerouslySetInnerHTML` usage found outside test files — the issue was in the markdown-to-HTML rendering configuration.

**Fix:** Issue closed as COMPLETED on 2026-02-11T23:40:42Z. No directly linked PR, but stateReason=COMPLETED indicates fix was applied. Verify fix lands in next upstream sync.

### #14137: Gateway Authentication Has No Rate Limiting (CWE-307)

**Severity:** HIGH
**CWE:** CWE-307 (Improper Restriction of Excessive Authentication Attempts)

**Vulnerability:** `authorizeGatewayConnect()` accepts unlimited failed authentication attempts with no rate limiting, lockout, or backoff. A PoC using 50 concurrent WebSocket connections achieves ~645 brute-force attempts/second with zero resistance. `safeEqual()` correctly uses `timingSafeEqual` (timing attacks mitigated), but the lack of attempt throttling means weak tokens can be brute-forced in seconds.

**Affected code:**
- `src/security/secret-equal.ts:3-16` — `safeEqualSecret()` (extracted from auth.ts, timing-safe)
- `src/gateway/server-http.ts:252-264` — hook auth failure rate limiting added (Feb 13 sync 4, commit `113ebfd6a`): 20 failures/60s per client IP, HTTP 429 response
- `src/gateway/server/ws-connection/message-handler.ts` — no per-connection attempt limiting

**Fix available:** PR [#13680](https://github.com/openclaw/openclaw/pull/13680) (OPEN, not merged) — per-IP sliding window: 10 failures in 60s → IP blocked for 5 minutes; HTTP 429 with Retry-After; localhost exempt.

**Relationship:** Subset of #8594 (general rate limiting, CWE-770, MEDIUM) but higher severity — specifically targets auth brute-force with demonstrated PoC.

### #14117: Session Isolation & Message Attribution Failure

**Severity:** MEDIUM
**CWE:** CWE-200 (Exposure of Sensitive Information) / CWE-362 (Race Condition)

**Vulnerability:** Three distinct session management failures:
1. **Session crossover:** Messages sent to the main session appear in remote Triager sessions (connected via Tailscale) as if the user sent them to the Triager
2. **Raw cron output:** Cron job internals (session keys, stats, system prompts) displayed to users instead of clean notifications
3. **Message attribution:** System cannot distinguish between user input, leaked messages from other sessions, and automated system content

**Affected code:**
- Session routing code across `src/cron/service/jobs.ts`, `src/infra/outbound/`, `src/gateway/server/hooks.ts`
- 25+ files involved in session isolation paths

**Impact:** Private conversations in the main session are visible in remote sessions. Agents may respond to "user" messages that the user never sent, creating both privacy and integrity failures.

**Relationship:** Related to #12571 (session isolation leak in cron jobs after ~24h) — different manifestation. #12571 is cron-specific after extended runtime; #14117 is cross-session routing between main and remote sessions. May share root cause in session routing/isolation code.

### #14808: apiKey Resolved to Plaintext in models.json Cache File

**Status: WONTFIX** -- closed upstream as NOT_PLANNED 2026-02-13.

**Severity:** MEDIUM (WONTFIX, DUPLICATE of #9627/#13683 family)
**CWE:** CWE-312 (Cleartext Storage of Sensitive Information)

**Vulnerability:** When using `${VAR}` syntax for `apiKey` in `openclaw.json`, OpenClaw resolves the environment variable to plaintext at runtime and writes the resolved value to the agent's `models.json` cache file (`~/.openclaw/agents/main/agent/models.json`). A sandboxed agent with file read access can extract any API key configured via env var substitution.

**Affected code:**
- `src/agents/models-config.ts:126-130` — `normalizeProviders()` returns provider objects including resolved `apiKey` fields; `JSON.stringify({ providers: normalizedProviders })` serializes them to disk
- `src/agents/models-config.ts:142` — file written with `mode: 0o600` (correct permissions, owner-only)

**Mitigation:** File has 0o600 permissions (only owner-readable), so external users cannot read it. However, the agent process itself can read the file, and a sandboxed agent with exec access can `cat` the file to extract all provider API keys.

**Relationship to existing issues:**
- #9627: Config *writeback* destroys `${VAR}` references in `openclaw.json` (same root cause, different file)
- #13683: CLI `config get` returns unredacted secrets (different path: CLI stdout)
- #5995: Secrets in session *transcripts* (different path: `.jsonl` files)
- #8591: Env vars via `env`/`printenv` (different path: `process.env`)

**Proposed fix (from issue):** Strip `apiKey` from provider objects before writing to `models.json`. Resolve credentials at HTTP request time instead of at cache write time.

### #14875: Feishu Channel Hardcodes CommandAuthorized Bypassing Access Groups

**Status: FIXED** -- closed as COMPLETED upstream 2026-02-13

**Severity:** ~~HIGH~~ FIXED (was CVSS 7.1)
**CWE:** CWE-862 (Missing Authorization)

**Vulnerability:** The Feishu channel extension unconditionally sets `CommandAuthorized: true` for every inbound message, bypassing the access group command gating system. All 16 other channel implementations (Discord, Mattermost, Matrix, Zalo, ZaloUser, BlueBubbles, WhatsApp, Google Chat, IRC, Nextcloud Talk, MSTeams, Telegram, Slack, iMessage) properly compute this value dynamically via `resolveCommandAuthorizedFromAuthorizers`.

**Affected code:**
- `extensions/feishu/src/bot.ts:818` — `CommandAuthorized: true` hardcoded in permission error context
- `extensions/feishu/src/bot.ts:906` — `CommandAuthorized: true` hardcoded in main message context
- Zero imports of `resolveCommandAuthorizedFromAuthorizers` in the Feishu extension

**Verification:**
- `src/channels/command-gating.ts:8` defines `resolveCommandAuthorizedFromAuthorizers()`
- 16 other channel implementations all use dynamic `commandAuthorized` computation
- `src/gateway/server-methods/chat.ts:484` also hardcodes `true` but is gateway-auth protected (owner-facing webchat, expected behavior)

**Impact:** Any Feishu user can execute admin/control commands (e.g., `/model`, `/new`, `/reset`, `/elevated`) regardless of access group configuration. Requires Feishu channel to be enabled with access groups configured. Without access groups, all users are already allowed, so this only affects Feishu deployments with access restrictions.

### #13274: SSRF Guard Bypassed by IPv4-Compatible IPv6 Addresses

**Severity:** HIGH
**CWE:** CWE-918 (Server-Side Request Forgery)

**Vulnerability:** The `isPrivateIpAddress()` function does not recognize IPv4-compatible IPv6 addresses like `::127.0.0.1` or `::7f00:1` as private/loopback. It only handles `::ffff:` mapped addresses and checks `PRIVATE_IPV6_PREFIXES = ["fe80:", "fec0:", "fc", "fd"]`. An attacker controlling DNS can return AAAA records with `::127.0.0.1` to bypass both pre-resolution and post-resolution SSRF checks, reaching internal services.

**Affected code:**
- `src/infra/net/ssrf.ts:26` — `PRIVATE_IPV6_PREFIXES` does not include `::` prefix patterns for IPv4-compatible addresses
- `src/infra/net/ssrf.ts:144-173` — `isPrivateIpAddress()` handles `::ffff:` (line 153) and exact `::` / `::1` (line 162) but not `::127.0.0.1`
- `src/infra/net/ssrf.ts:165` — fallback prefix check misses IPv4-compatible form
- `src/infra/net/ssrf.ts:276,289` — both pre-resolution and post-resolution checks use same vulnerable function

### #11738: Canvas Authorization IP Co-Tenancy Bypass

**Severity:** HIGH
**CWE:** CWE-287 (Improper Authentication)

**Vulnerability:** `hasAuthorizedWsClientForIp()` trusts any HTTP request whose source IP matches an already-authenticated WebSocket client. In shared-IP deployments (NAT, corporate proxy, or trusted reverse proxy), one authenticated client implicitly authorizes all other users on the same IP, creating a cross-user auth bypass for Canvas endpoints.

**Affected code:**
- `src/gateway/server-http.ts:100-105` — `hasAuthorizedWsClientForIp()` does pure IP-based matching with no per-user or per-session distinction
- `src/gateway/server-http.ts:150` — called in `authorizeCanvasRequest()` as fallback authorization method
- Relies on `client.clientIp` field which is the same for all users behind NAT

### #11793: HTTP API Session Keys Lack Ownership Validation

**Severity:** HIGH
**CWE:** CWE-639 (Authorization Bypass Through User-Controlled Key)

**Vulnerability:** Multiple HTTP API endpoints accept user-controlled session keys via the `x-openclaw-session-key` header or request body without ownership validation. In multi-user deployments (Tailscale Serve), any authenticated user can read and write another user's conversation history, memories, and tool execution context by supplying a predictable session key.

**Affected code:**
- `src/gateway/http-utils.ts:65-79` — `resolveSessionKey()` returns `x-openclaw-session-key` header value as-is (line 71-73) with no ownership check
- `src/gateway/tools-invoke-http.ts:62-65` — `resolveSessionKeyFromBody()` accepts arbitrary session key from request body
- Affected endpoints: `/v1/chat/completions`, `/v1/responses`, `/tools/invoke`, `/hooks/agent`

### #11024: Gmail Push Endpoint Embeds Auth Token in URL Query String

**Severity:** HIGH
**CWE:** CWE-598 (Sensitive Query Strings)

**Vulnerability:** Gmail webhook setup constructs Pub/Sub push endpoints as `https://...?token=<secret>`, exposing the shared secret via URL telemetry surfaces (reverse-proxy logs, access logs, traces, analytics). Additionally, the setup prints tokens in CLI output and `--json` mode.

**Affected code:**
- `src/hooks/gmail-setup-utils.ts:315` — URL constructed with `?token=<secret>` parameter
- Same vulnerability class as previously fixed #9435 (gateway auth in URL) and #5120 (webhook query token), but in Gmail-specific code path that was missed

### #11811: MSTeams Attachment Fetch Follows Redirects Before Allowlist Checks (SSRF)

**Severity:** HIGH
**CWE:** CWE-918 (Server-Side Request Forgery)

**Vulnerability:** The MSTeams attachment downloader's `fetchWithAuthFallback()` performs the initial fetch with default redirect behavior (follows redirects automatically). If an allowed URL redirects to an internal/disallowed host, the HTTP client follows the redirect and returns the response without checking the redirect target against the allowlist.

**Affected code:**
- `extensions/msteams/src/attachments/download.ts:93` — `await fetchFn(params.url)` with default redirect behavior (no `redirect: "manual"`)
- Line 94-95: if `firstAttempt.ok`, returns immediately — redirect target URL was never validated
- Line 111-113: the authenticated retry correctly uses `redirect: "manual"`, but line 93 (unauthenticated first attempt) does not
- `extensions/msteams/src/attachments/shared.ts` — `isUrlAllowed()` only applied to initial URL at download.ts:237, not redirect targets

**Verification:**
- Line 237 calls `isUrlAllowed(candidate.url, allowHosts)` before `fetchWithAuthFallback` — initial URL is validated
- But `fetch()` at line 93 follows 30x redirects automatically — redirect target is not validated
- Contrast with line 111-113: authenticated path correctly uses `redirect: "manual"` and validates redirect at line 119

**Note:** Requires MSTeams channel to be enabled AND a compromised or attacker-controlled host in the `allowHosts` configuration. Relates to tracked #13274 (SSRF guard IPv6 bypass) but different attack vector: redirect-following vs DNS resolution.

### #11202: Model Catalog with Resolved apiKey in LLM Prompt Context

**Severity:** MEDIUM
**CWE:** CWE-200 (Exposure of Sensitive Information)

**Vulnerability:** The runtime model catalog (resolved from `openclaw.json` providers) is serialized into every LLM request payload as system prompt context. Environment variable references (`${VAR}`) are resolved to plaintext before serialization, so all provider API keys are sent to whichever LLM provider handles the request. Every provider sees every other provider's keys.

**Affected code:**
- `src/agents/models-config.ts:126-142` — `normalizeProviders()` returns provider objects including resolved `apiKey` fields
- `src/agents/models-config.providers.ts` — provider normalization includes credential resolution

**Relationship:** Related to #14808 (apiKey in models.json cache, now WONTFIX), #13683 (CLI config get unredacted), and #9627 (config write-back). Different vector: keys leaked to LLM providers via prompt, not just disk/CLI.

### #12173: apply_patch Tool Path Traversal When Sandbox Disabled

**Severity:** MEDIUM
**CWE:** CWE-22 (Path Traversal)

**Vulnerability:** `resolvePatchPath()` has two code paths. When `sandboxRoot` is set, it calls `assertSandboxPath()` (secure). When `sandboxRoot` is `undefined` (gateway/non-sandboxed mode), it falls through to `resolvePathFromCwd()` which accepts absolute paths and normalizes `../` via `path.resolve()` with zero containment checks. Default sandbox mode is "off" (`src/agents/sandbox/config.ts:147`).

**Affected code:**
- `src/agents/apply-patch.ts:215-236` — `resolvePatchPath()` function
- Lines 219-228: sandbox path (secure, calls `assertSandboxPath`)
- Lines 231-234: non-sandbox path (vulnerable, no containment)
- Called at lines 141, 149, 155, 159 for add/delete/update/move patch operations

**Mitigation note:** When sandbox is disabled, the exec tool already has unrestricted file access. This is an additional vector (apply_patch vs exec) but does not expand the attack surface beyond what exec provides. Severity is MEDIUM rather than HIGH because it does not bypass a security boundary that is otherwise enforced.

### Notable Non-Core Issues

#### #9860: System Prompt Hijacking in google-antigravity Provider

**Not a core OpenClaw vulnerability.** This is a dependency issue in the `@mariozechner/pi-ai` library used by the google-antigravity provider. The attack uses triple identity injection combined with role corruption (`system` → `user` role remapping) to hijack the system prompt. Affects users of the Gemini provider through OpenClaw. Worth monitoring but the fix must come from the upstream dependency.

#### #9828: Config Schema Injected Into Every Session (~100-150k Tokens)

**Design concern, not a vulnerability.** The full config schema is injected into every agent session's system prompt, consuming an estimated 50-75% of the 200k context window. This creates both an information disclosure vector (config schema reveals internal structure) and a resource waste problem (reduced effective context for actual conversation). A potential optimization would be to inject only relevant config keys per session.
