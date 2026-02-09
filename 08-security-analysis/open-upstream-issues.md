> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## Open Upstream Security Issues

> **Status:** These issues are open in upstream openclaw/openclaw and confirmed to affect the local codebase. Monitor for patches.
>
> **Last checked:** 09-02-2026 (21:45 AEST)

| Issue | Severity | Summary | Local Impact |
|-------|----------|---------|--------------|
| [#8512](https://github.com/openclaw/openclaw/issues/8512) | CRITICAL | Plugin HTTP routes bypass gateway authentication | `src/gateway/server/plugins-http.ts:17-59` |
| [#3277](https://github.com/openclaw/openclaw/issues/3277) | HIGH | Path validation bypass via `startsWith` prefix | `src/infra/archive.ts:81,89` - zip/tar extraction |
| [#4949](https://github.com/openclaw/openclaw/issues/4949) | HIGH | Browser control server DNS rebinding | `src/browser/server.ts:36` - no Host header validation |
| [#4950](https://github.com/openclaw/openclaw/issues/4950) | HIGH | Arbitrary JS execution via browser evaluate (default on) | `src/browser/constants.ts:2` - `DEFAULT_BROWSER_EVALUATE_ENABLED = true` |
| [#4995](https://github.com/openclaw/openclaw/issues/4995) | HIGH | iMessage dmPolicy auto-responds with pairing codes | `src/imessage/monitor/monitor-provider.ts:184,342-381` |
| [#5052](https://github.com/openclaw/openclaw/issues/5052) | HIGH | Config validation fail-open returns `{}` | `src/config/io.ts:317-321` - security settings reset |
| [#5255](https://github.com/openclaw/openclaw/issues/5255) | HIGH | Browser file upload arbitrary read | `src/browser/pw-tools-core.interactions.ts:531` |
| [#5995](https://github.com/openclaw/openclaw/issues/5995) | HIGH | Secrets exposed in session transcripts | `config.get` now redacted via `redactConfigSnapshot()` (PR #9858); transcripts still expose by design |
| [#6606](https://github.com/openclaw/openclaw/issues/6606) | HIGH | Telegram webhook binds to 0.0.0.0 with optional secret | `src/telegram/webhook.ts:26,36,46-48` |
| [#6609](https://github.com/openclaw/openclaw/issues/6609) | HIGH | Browser bridge server optional authentication | `src/browser/bridge-server.ts:33-42` |
| [#8054](https://github.com/openclaw/openclaw/issues/8054) | HIGH | Type coercion `"undefined"` credentials | `src/wizard/onboarding.gateway-config.ts:206` |
| [#8516](https://github.com/openclaw/openclaw/issues/8516) | HIGH | Browser download/trace endpoints arbitrary file write | `src/browser/routes/agent.act.ts:447-480` |
| [#8586](https://github.com/openclaw/openclaw/issues/8586) | HIGH | Configurable bypass allows unrestricted command exec | `src/agents/bash-tools.exec.ts:938-947,1278` |
| [#8591](https://github.com/openclaw/openclaw/issues/8591) | HIGH | Env vars exposed via shell commands | `src/agents/bash-tools.exec.ts:972,976-980` |
| [#8590](https://github.com/openclaw/openclaw/issues/8590) | HIGH | Status endpoint exposes sensitive internal info | `src/gateway/server-methods/health.ts:28-31` |
| [#8696](https://github.com/openclaw/openclaw/issues/8696) | HIGH | Playwright download path traversal | `src/browser/pw-tools-core.downloads.ts:20-24` |
| [#8776](https://github.com/openclaw/openclaw/issues/8776) | HIGH | soul-evil hook silently hijacks agent | `src/hooks/soul-evil.ts:217-280` |
| [#9435](https://github.com/openclaw/openclaw/issues/9435) | ~~HIGH~~ FIXED | Gateway auth token exposed in URL query params | Fixed in PR [#9436](https://github.com/openclaw/openclaw/pull/9436) — query token acceptance removed from `src/gateway/hooks.ts`, dashboard URL no longer passes `?token=` |
| [#9512](https://github.com/openclaw/openclaw/issues/9512) | HIGH | Skill download archive path traversal | `src/agents/skills-install.ts:267,274` |
| [#9517](https://github.com/openclaw/openclaw/issues/9517) | ~~HIGH~~ FIXED | Gateway canvas host auth bypass | Fixed in PR [#9518](https://github.com/openclaw/openclaw/pull/9518) — new `authorizeCanvasRequest()` at `src/gateway/server-http.ts:92-126` |
| [#9627](https://github.com/openclaw/openclaw/issues/9627) | HIGH | Config secrets exposed in JSON after update/doctor | `src/config/io.ts:482-539` — partially mitigated by `redactConfigSnapshot()` (PR #9858) |
| [#9813](https://github.com/openclaw/openclaw/issues/9813) | HIGH (DUPLICATE #9627) | Gateway expands ${ENV_VAR} on meta writeback | `src/config/io.ts:498` — partially mitigated by `redactConfigSnapshot()` (PR #9858) |
| [#11126](https://github.com/openclaw/openclaw/issues/11126) | HIGH (DUPLICATE #9627) | Config write paths resolve ${VAR} to cleartext | Same as #9627/#9813 — `src/config/io.ts:482-539`; partially mitigated by `redactConfigSnapshot()` (PR #9858) |
| [#9795](https://github.com/openclaw/openclaw/issues/9795) | LOW | sanitizeMimeType regex not end-anchored (by design) | `src/media-understanding/apply.ts:96-106` |
| [#9792](https://github.com/openclaw/openclaw/issues/9792) | INVALID | validateHostEnv skips baseEnv (by design) | `src/agents/bash-tools.exec.ts:972-980` |
| [#9791](https://github.com/openclaw/openclaw/issues/9791) | INVALID | Fullwidth marker bypass (fold is length-preserving) | `src/security/external-content.ts:110-148` |
| [#9667](https://github.com/openclaw/openclaw/issues/9667) | INVALID | JWT verification in nonexistent file | `src/auth/jwt.ts` (does not exist) |
| [#4940](https://github.com/openclaw/openclaw/issues/4940) | MEDIUM | commands.restart bypass via exec tool | `src/agents/bash-tools.exec.ts` (no commands.restart check) |
| [#5120](https://github.com/openclaw/openclaw/issues/5120) | ~~MEDIUM~~ FIXED | Webhook token accepted via query parameters | Fixed in PR [#9436](https://github.com/openclaw/openclaw/pull/9436) — query token extraction removed from `src/gateway/hooks.ts` (note: upstream issue still OPEN) |
| [#5122](https://github.com/openclaw/openclaw/issues/5122) | MEDIUM | readJsonBody() Slowloris DoS (no read timeout) | `src/gateway/hooks.ts:65-111` |
| [#5123](https://github.com/openclaw/openclaw/issues/5123) | MEDIUM | ReDoS in session filter regex | `src/infra/exec-approval-forwarder.ts:70-77` |
| [#5124](https://github.com/openclaw/openclaw/issues/5124) | MEDIUM | ReDoS in log redaction patterns | `src/logging/redact.ts:47-61` |
| [#6021](https://github.com/openclaw/openclaw/issues/6021) | MEDIUM | Timing attack in non-gateway token comparisons | `src/gateway/server-http.ts:160`, `src/infra/node-pairing.ts:277` |
| [#7862](https://github.com/openclaw/openclaw/issues/7862) | MEDIUM | Session transcripts 644 instead of 600 | `src/auto-reply/reply/session.ts:87` |
| [#8027](https://github.com/openclaw/openclaw/issues/8027) | MEDIUM | web_fetch hidden text prompt injection | `src/agents/tools/web-fetch-utils.ts:31-34` |
| [#8592](https://github.com/openclaw/openclaw/issues/8592) | MEDIUM | No detection of encoded/obfuscated commands | `src/infra/exec-safety.ts:1-44` |
| [#8588](https://github.com/openclaw/openclaw/issues/8588) | MEDIUM | Sensitive config files accessible when sandbox is home dir | `src/agents/sandbox/context.ts:39-46` (workspaceAccess=rw) |
| [#8589](https://github.com/openclaw/openclaw/issues/8589) | LOW | Sandbox file read lacks content filtering | `src/agents/pi-tools.read.ts:286-302` (no redaction on read) |
| [#8593](https://github.com/openclaw/openclaw/issues/8593) | MEDIUM | chat.send handler lacks input length validation | `src/gateway/protocol/schema/logs-chat.ts:34-45` |
| [#8594](https://github.com/openclaw/openclaw/issues/8594) | MEDIUM | No rate limiting on gateway endpoints | `src/gateway/server-constants.ts` (no rate limit controls) |
| [#9007](https://github.com/openclaw/openclaw/issues/9007) | LOW | Google Places URL path interpolation (skill, not core) | `skills/local-places/src/local_places/google_places.py:238` |
| [#9065](https://github.com/openclaw/openclaw/issues/9065) | LOW | ~/.openclaw group-writable after sudo install | Operational - code uses `0o700` but sudo bypasses |
| [#10324](https://github.com/openclaw/openclaw/issues/10324) | MEDIUM | Memory index multi-write lacks transactions | `src/memory/manager.ts:2257-2354` (DELETEs+INSERTs without BEGIN/COMMIT) |
| [#10326](https://github.com/openclaw/openclaw/issues/10326) | MEDIUM | Child process stop() lacks SIGKILL escalation | `src/imessage/client.ts:110-131`, `src/signal/daemon.ts:96-100` |
| [#10330](https://github.com/openclaw/openclaw/issues/10330) | MEDIUM | TOCTOU race in device auth token storage | `src/infra/device-auth-store.ts:92-119` (read+write with no lock) |
| [#10331](https://github.com/openclaw/openclaw/issues/10331) | MEDIUM | Session store stale cache inside write lock | `src/config/sessions/store.ts:364,422` (missing `skipCache: true`) |
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
| [#10521](https://github.com/openclaw/openclaw/issues/10521) | INVALID | Security audit flags claude-opus-4-6 as below 4.5 | `src/security/audit-extra.ts:321` regex correctly matches `claude-opus-4-6` in current code (2026.2.6) |
| [#10033](https://github.com/openclaw/openclaw/issues/10033) | ENHANCEMENT | Feature: secrets management integration | Enhancement request; current state: plaintext creds with 0o600 perms |
| [#10927](https://github.com/openclaw/openclaw/issues/10927) | ENHANCEMENT | Random IDs for external content wrapper tags | `src/security/external-content.ts:47-48` — static tags; `replaceMarkers()` at `:110-150` already sanitizes injected markers |
| [#10890](https://github.com/openclaw/openclaw/issues/10890) | ENHANCEMENT | RFC: Skill Security Framework (manifests, signing, sandboxing) | Comprehensive proposal for phased skill security; relates to #9512 (skill path traversal) |
| [#11437](https://github.com/openclaw/openclaw/issues/11437) | CRITICAL | CWD .env → config path override → plugin code exec via jiti | `src/infra/dotenv.ts:10`, `src/config/paths.ts:87-105`, `src/plugins/config-state.ts:73,194` |
| [#11434](https://github.com/openclaw/openclaw/issues/11434) | CRITICAL | CWD .env → arbitrary dynamic import via OPENCLAW_BROWSER_CONTROL_MODULE | `src/gateway/server-browser.ts:13-14` — raw `await import(override)` |
| [#11431](https://github.com/openclaw/openclaw/issues/11431) | CRITICAL | Hook/plugin npm install runs lifecycle scripts (no --ignore-scripts) | `src/hooks/install.ts:237`, `src/plugins/install.ts:281` |
| [#11023](https://github.com/openclaw/openclaw/issues/11023) | HIGH | Sandbox browser bridge started without auth token | `src/agents/sandbox/browser.ts:192` — no `authToken` passed; relates to #6609 |
| [#11945](https://github.com/openclaw/openclaw/issues/11945) | HIGH | config.patch bypasses commands.restart restriction | `src/gateway/server-methods/config.ts:330` — `scheduleGatewaySigusr1Restart()` with no `commands.restart` check; contrast `gateway-tool.ts:78` |
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

**Affected code:** `src/config/io.ts:317-321`

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
- Plugin HTTP dispatch at `server-http.ts:332` occurs without auth check

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
- `src/gateway/server-http.ts:356-376` - Canvas/A2UI handler dispatch (now auth-wrapped via `authorizeCanvasRequest()` at `:92-126`, PR #9518)
- `src/gateway/server-http.ts:418-440` - WebSocket upgrade for canvas (now auth-wrapped via `authorizeCanvasRequest()` at `:425`, PR #9518)

**Verification:**
- No `authorizeGatewayConnect` call before `canvasHost.handleHttpRequest(req, res)`
- Other endpoints in the same file DO call authorization methods

### #5120: Webhook Token Accepted via Query Parameters

**Status: FIXED** in PR [#9436](https://github.com/openclaw/openclaw/pull/9436)

**Severity:** ~~MEDIUM~~ FIXED
**CWE:** CWE-598 (Sensitive Query Strings)

**Vulnerability:** Webhook endpoint accepted authentication tokens via URL query parameters, causing credential leakage through logs, browser history, and Referer headers.

**Fix:** Query token extraction removed entirely from `src/gateway/hooks.ts`. `extractHookToken()` now only accepts `Authorization: Bearer` header and `X-OpenClaw-Token` header. Server returns HTTP 400 when `?token=` is present (`src/gateway/server-http.ts:150-157`).

### #4949: Browser Control Server DNS Rebinding

**Severity:** HIGH
**CWE:** CWE-350 (Reliance on Reverse DNS Resolution)

**Vulnerability:** Browser control server binds to `127.0.0.1` but performs no Host header validation. DNS rebinding attacks can bypass localhost restriction to reach browser automation endpoints from a remote origin.

**Affected code:**
- `src/browser/server.ts:36` - binds to `"127.0.0.1"` but no Host header check
- `src/browser/server.ts:26-41` - no `isLocalDirectRequest` or origin validation

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

**Affected code:** `src/gateway/hooks.ts:65-111` - size limit present, timeout absent.

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

**Severity:** MEDIUM
**CWE:** CWE-208 (Observable Timing Discrepancy)

**Vulnerability:** Gateway auth correctly uses `safeEqual` (timing-safe), but hook tokens, node pairing, and device pairing use direct `===`/`!==` comparisons vulnerable to timing attacks.

**Affected code:**
- `src/gateway/auth.ts:35-40` - `safeEqual` uses `timingSafeEqual` (correct)
- `src/gateway/server-http.ts:160` - hook token uses direct `!==` (vulnerable)
- `src/infra/node-pairing.ts:277` - node token uses direct `===` (vulnerable)
- `src/infra/device-pairing.ts:434` - device token uses direct `!==` (vulnerable)

### #6606: Telegram Webhook Binds to 0.0.0.0 with Optional Secret

**Severity:** HIGH
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
- `src/agents/bash-tools.exec.ts:938-940,945-947,951-953` - `elevatedMode=full` sets security to "full", ask to "off"
- `src/agents/bash-tools.exec.ts:1278` - `bypassApprovals` skips all approval checks

### #8590: Status Endpoint Exposes Sensitive Internal Info

**Severity:** HIGH
**CWE:** CWE-200 (Exposure of Sensitive Information)

**Vulnerability:** Gateway status/health endpoint returns unredacted internal information including file paths, session IDs, agent IDs, and model configuration to any connected client.

**Affected code:**
- `src/gateway/server-methods/health.ts:28-31` - returns full unredacted `getStatusSummary()`
- `src/commands/status.summary.ts:134-195` - exposes paths, session IDs, agent IDs, model configs

### #8591: Env Vars Exposed via Shell Commands

**Severity:** HIGH
**CWE:** CWE-526 (Exposure of Sensitive Information Through Environmental Variables)

**Vulnerability:** Full `process.env` is passed as the base environment to child processes. An agent can run `env` or `printenv` to dump all environment variables, including API keys and secrets.

**Affected code:**
- `src/agents/bash-tools.exec.ts:972,976-980` - full `process.env` passed to child spawn
- `src/agents/bash-tools.exec.ts:61-78` - `DANGEROUS_HOST_ENV_VARS` blocks injection INTO env, but doesn't filter what children can READ

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

**Severity:** HIGH
**CWE:** CWE-506 (Embedded Malicious Code)

**Vulnerability:** The `soul-evil` hook ships bundled with OpenClaw and can silently replace the agent's SOUL.md (system prompt) content. When activated, it overrides the agent's personality and behavior without explicit user notification.

**Affected code:**
- `src/hooks/bundled/soul-evil/handler.ts:8-49` - bundled hook handler
- `src/hooks/soul-evil.ts:217-280` - `applySoulEvilOverride` swaps SOUL.md content
- `src/hooks/soul-evil.ts:187-216` - `decideSoulEvil` activation logic

**Note:** May be an intentional Easter egg feature, but the security impact (silent system prompt replacement) is real.

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

**Vulnerability:** Code correctly uses `mode: 0o700` for directory creation (`src/config/io.ts:497`), but when installed via `sudo`, the directory inherits root ownership. Subsequent user-space operations may create group-writable files.

**Note:** This is an operational issue (sudo usage), not a code bug. `src/security/audit.ts:159-176` already detects group-writable state directories.

### #9435: Gateway Auth Token Exposed in URL Query Params

**Status: FIXED** in PR [#9436](https://github.com/openclaw/openclaw/pull/9436)

**Severity:** ~~HIGH~~ FIXED
**CWE:** CWE-598 (Sensitive Query Strings)

**Vulnerability:** Gateway authentication tokens were passed via URL query parameters (`?token=...`) in dashboard and onboarding flows, exposing credentials through logs, browser history, and Referer headers.

**Fix:** Query token acceptance completely removed. `extractHookToken()` in `src/gateway/hooks.ts:46-63` no longer reads `url.searchParams`. `src/commands/dashboard.ts` no longer constructs `?token=` URLs. `src/commands/onboard-helpers.ts` no longer passes token in URL. Server now returns HTTP 400 when `?token=` is present (`src/gateway/server-http.ts:150-157`).

### #9627: Config Secrets Exposed in JSON After Update/Doctor

**Severity:** HIGH
**CWE:** CWE-312 (Cleartext Storage of Sensitive Information)

**Vulnerability:** When `openclaw doctor` or `openclaw config set` writes the config file, environment variable references (`${VAR}`) are resolved to plaintext values. The write-back serializes resolved secrets to disk in cleartext JSON, destroying the original `${VAR}` references.

**Affected code:**
- `src/config/io.ts:482-539` - `writeConfigFile` serializes resolved config without restoring `${VAR}` references
- `src/config/env-substitution.ts:83-89` - `substituteString` is a one-way transformation
- `src/commands/doctor.ts:285` - `writeConfigFile(cfg)` writes env-resolved config back to disk

### #9813: Gateway Expands ${ENV_VAR} on Meta Writeback (DUPLICATE of #9627)

**Severity:** HIGH (DUPLICATE)
**CWE:** CWE-312 (Cleartext Storage of Sensitive Information)

**Vulnerability:** Same root cause as #9627. When the gateway updates `meta.lastTouchedAt` in the config file, the writeback path resolves `${ENV_VAR}` references to plaintext values, destroying the original references.

**Affected code:**
- `src/config/io.ts:498` - `JSON.stringify` serializes resolved config during meta writeback

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
- `src/agents/bash-tools.exec.ts:972-980` - validation scoped to `params.env` only

**Our analysis:** `baseEnv = coerceEnv(process.env)` is the **host's own environment**, not untrusted input. The code comment at line 969 states: "We validate BEFORE merging to prevent any dangerous vars from entering the stream." Validating `baseEnv` would **break the gateway** — the host always has `PATH` set, which `validateHostEnv` explicitly rejects (it's designed to block agents from injecting `PATH` overrides). The validation boundary is intentionally scoped to untrusted agent-supplied variables. This is a Qodo AI automated finding.

### #9791: Fullwidth Character Markers Bypass (Fold Is Length-Preserving)

**Severity:** INVALID
**CWE:** N/A

**Vulnerability claimed:** Fullwidth Unicode characters (e.g., `＜` U+FF1C) could bypass marker detection in `replaceMarkers`, causing index misalignment when mapping between folded and original strings.

**Affected code:**
- `src/security/external-content.ts:110-148` - `replaceMarkers` and `foldMarkerChar`

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

**Severity:** MEDIUM
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
- `src/memory/manager.ts:2257-2354` — `indexFile()` performs 5+ SQL operations (DELETE vector :2273, DELETE FTS :2281, DELETE chunks :2287, INSERT loop :2290-2343, UPSERT files :2344-2353) with **no BEGIN/COMMIT**
- Contrast: `src/memory/manager.ts:740,752` correctly uses `BEGIN`/`COMMIT` for similar batch operations

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
- `src/config/sessions/store.ts:364` — `updateSessionEntry()` inside `withSessionStoreLock()` calls `loadSessionStore(storePath)` **without `skipCache: true`**
- `src/config/sessions/store.ts:422` — `updateLastRoute()` same pattern
- Correct usage at :272 passes `{ skipCache: true }` when reading inside a write path

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

**Current defense:** `replaceMarkers()` at `src/security/external-content.ts:110-150` already sanitizes injected `<<<EXTERNAL_UNTRUSTED_CONTENT>>>` tags (case-insensitive, including fullwidth Unicode variants) to `[[MARKER_SANITIZED]]`. The existing defense is functional; random IDs would add defense-in-depth and improve content correlation for debugging.

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
- `src/agents/pi-embedded-runner/run/attempt.ts:211-215` — `execOverrides` passed to tool creation, but populated only from directives/session

**Impact:** If an operator configures per-agent exec restrictions (e.g., `agents.mybot.tools.exec.host = "docker"` for sandboxed execution), those restrictions are silently ignored. The agent runs with global exec defaults. Global config still applies; only per-agent overrides are lost.

### #11945: config.patch Bypasses commands.restart Restriction

**Severity:** HIGH
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
- `src/cli/gateway-cli/run-loop.ts:76-77` — `consumeGatewaySigusr1RestartAuthorization()` returns true (pre-authorized), bypassing `isGatewaySigusr1RestartExternallyAllowed()`
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

### Notable Non-Core Issues

#### #9860: System Prompt Hijacking in google-antigravity Provider

**Not a core OpenClaw vulnerability.** This is a dependency issue in the `@mariozechner/pi-ai` library used by the google-antigravity provider. The attack uses triple identity injection combined with role corruption (`system` → `user` role remapping) to hijack the system prompt. Affects users of the Gemini provider through OpenClaw. Worth monitoring but the fix must come from the upstream dependency.

#### #9828: Config Schema Injected Into Every Session (~100-150k Tokens)

**Design concern, not a vulnerability.** The full config schema is injected into every agent session's system prompt, consuming an estimated 50-75% of the 200k context window. This creates both an information disclosure vector (config schema reveals internal structure) and a resource waste problem (reduced effective context for actual conversation). A potential optimization would be to inject only relevant config keys per session.
