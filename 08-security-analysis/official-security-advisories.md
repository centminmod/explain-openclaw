> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Open PRs](./open-upstream-prs.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Hudson Rock](./hudson-rock-infostealer-analysis.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## Official Security Advisories (CVEs/GHSAs)

> **Source:** [github.com/openclaw/openclaw/security](https://github.com/openclaw/openclaw/security)
>
> These are officially disclosed vulnerabilities with assigned CVE/GHSA identifiers. Earlier advisories were patched in v2026.1.29-1.30; Feb 14 batch (5 new advisories) patched in v2026.2.1-2.6; Feb 15 batch (26 new advisories, 10 HIGH + 10 MEDIUM + 6 previously tracked) patched in v2026.2.1-2.2+.

### Advisory Summary

| ID | Severity | Summary | CWE | Patched | Credits |
|----|----------|---------|-----|---------|---------|
| [CVE-2026-24763](https://github.com/openclaw/openclaw/security/advisories/GHSA-mc68-q9jw-2h3v) | HIGH | Command Injection via Docker PATH Variable | CWE-78 | v2026.1.29 | @berkdedekarginoglu |
| [GHSA-g8p2-7wf7-98mq](https://github.com/openclaw/openclaw/security/advisories/GHSA-g8p2-7wf7-98mq) | HIGH | 1-Click RCE via gatewayUrl Token Exfiltration | CWE-200 | v2026.1.29 | DepthFirstDisclosures, @0xacb, @mavlevin |
| [GHSA-q284-4pvr-m585](https://github.com/openclaw/openclaw/security/advisories/GHSA-q284-4pvr-m585) | HIGH | OS Command Injection via sshNodeCommand | - | v2026.1.29 | @koko9xxx |
| [CVE-2026-25593](https://github.com/openclaw/openclaw/security/advisories/GHSA-g55j-c2v4-pjcg) | HIGH | Unauthenticated Local RCE via WebSocket config.apply | CWE-20, CWE-78, CWE-306 | v2026.1.20 | @hackerman70000 |
| [CVE-2026-25475](https://github.com/openclaw/openclaw/security/advisories/GHSA-r8g4-86fx-92mq) | MEDIUM | Local File Inclusion via MEDIA: Path Extraction | CWE-22, CWE-200 | v2026.1.30 | @jasonsutter87 |
| [GHSA-gv46-4xfq-jv58](https://github.com/openclaw/openclaw/security/advisories/GHSA-gv46-4xfq-jv58) | CRITICAL | Remote Code Execution via Node Invoke Approval Bypass | - | pending | - |
| [GHSA-943q-mwmv-hhvh](https://github.com/openclaw/openclaw/security/advisories/GHSA-943q-mwmv-hhvh) | HIGH | OC-02: Gateway /tools/invoke Tool Escalation + ACP Auto-Approval | - | pending | - |
| [GHSA-hv93-r4j3-q65f](https://github.com/openclaw/openclaw/security/advisories/GHSA-hv93-r4j3-q65f) | HIGH | Hook Session Key Override Cross-Session Routing | - | pending | - |
| [GHSA-64qx-vpxx-mvqf](https://github.com/openclaw/openclaw/security/advisories/GHSA-64qx-vpxx-mvqf) | HIGH | Arbitrary Transcript Path File Write via sessionFile | - | pending | - |
| [GHSA-56f2-hvwg-5743](https://github.com/openclaw/openclaw/security/advisories/GHSA-56f2-hvwg-5743) | HIGH | SSRF in Image Tool Remote Fetch | - | pending | - |
| [GHSA-xc7w-v5x6-cc87](https://github.com/openclaw/openclaw/security/advisories/GHSA-xc7w-v5x6-cc87) | MEDIUM | Webhook Auth Bypass Behind Reverse Proxy (Loopback Trust) | - | pending | - |
| [GHSA-qw99-grcx-4pvm](https://github.com/openclaw/openclaw/security/advisories/GHSA-qw99-grcx-4pvm) | MEDIUM | Chrome Extension Relay Wildcard Binding | - | pending | - |
| [GHSA-wfp2-v9c7-fh79](https://github.com/openclaw/openclaw/security/advisories/GHSA-wfp2-v9c7-fh79) | MEDIUM | SSRF via Attachment/Media URL Hydration | CWE-918 | < v2026.2.2 | - |
| [CVE-2026-24764](https://github.com/openclaw/openclaw/security/advisories/GHSA-782p-5fr5-7fj8) | LOW | Slack Integration Hardening: Channel Metadata Injection | CWE-74, CWE-94 | < v2026.2.3 | - |
| [GHSA-mv9j-6xhh-g383](https://github.com/openclaw/openclaw/security/advisories/GHSA-mv9j-6xhh-g383) | HIGH | Unauthenticated Nostr Profile HTTP Endpoints | - | pending | - |
| [GHSA-3m3q-x3gj-f79x](https://github.com/openclaw/openclaw/security/advisories/GHSA-3m3q-x3gj-f79x) | MEDIUM | Voice-Call Webhook Verification Bypass Behind Proxy | - | pending | - |
| [GHSA-pchc-86f6-8758](https://github.com/openclaw/openclaw/security/advisories/GHSA-pchc-86f6-8758) | HIGH | BlueBubbles Webhook Auth Bypass via Loopback Proxy Trust | - | v2026.2.13 | @MegaManSec |
| [GHSA-g27f-9qjv-22pm](https://github.com/openclaw/openclaw/security/advisories/GHSA-g27f-9qjv-22pm) | LOW | Log Poisoning via WebSocket Headers | - | pending | - |
| [GHSA-fhvm-j76f-qmjv](https://github.com/openclaw/openclaw/security/advisories/GHSA-fhvm-j76f-qmjv) | HIGH | Access-Group Authorization Bypass if Channel Type Lookup Fails | CWE-285 | >= v2026.2.2 | @simecek |
| [GHSA-rmxw-jxxx-4cpc](https://github.com/openclaw/openclaw/security/advisories/GHSA-rmxw-jxxx-4cpc) | MEDIUM | Matrix Allowlist Bypass via displayName | - | >= v2026.2.2 | @MegaManSec |
| [GHSA-4rj2-gpmh-qq5x](https://github.com/openclaw/openclaw/security/advisories/GHSA-4rj2-gpmh-qq5x) | CRITICAL | Inbound Allowlist Policy Bypass in Voice-Call Extension | - | pending | - |
| [CVE-2026-25474](https://github.com/openclaw/openclaw/security/advisories/GHSA-mp5h-m6qj-6292) | HIGH | Telegram Webhook Secret Verification Bypass | - | pending | - |
| [GHSA-7vwx-582j-j332](https://github.com/openclaw/openclaw/security/advisories/GHSA-7vwx-582j-j332) | HIGH | MS Teams Attachment Downloader Leaks Bearer Tokens | - | pending | - |
| [GHSA-r5h9-vjqc-hq3r](https://github.com/openclaw/openclaw/security/advisories/GHSA-r5h9-vjqc-hq3r) | HIGH | Nextcloud Talk Allowlist Bypass via actor.name Spoofing | - | pending | - |
| [GHSA-qrq5-wjgg-rvqw](https://github.com/openclaw/openclaw/security/advisories/GHSA-qrq5-wjgg-rvqw) | CRITICAL | Path Traversal in Plugin Installation | CWE-22 | >= v2026.2.1 | @logicx24 |
| [GHSA-rv39-79c4-7459](https://github.com/openclaw/openclaw/security/advisories/GHSA-rv39-79c4-7459) | HIGH | Gateway Device Identity Skip | CWE-306 | >= v2026.2.2 | @simecek |
| [GHSA-qj77-c3c8-9c3q](https://github.com/openclaw/openclaw/security/advisories/GHSA-qj77-c3c8-9c3q) | HIGH | Windows cmd.exe Exec Allowlist Bypass | CWE-78 | >= v2026.2.2 | @simecek |
| [GHSA-mqpw-46fh-299h](https://github.com/openclaw/openclaw/security/advisories/GHSA-mqpw-46fh-299h) | HIGH | operator.write Exec Approval Bypass via /approve | CWE-269, CWE-863 | >= v2026.2.2 | @yueyueL |
| [GHSA-33rq-m5x2-fvgf](https://github.com/openclaw/openclaw/security/advisories/GHSA-33rq-m5x2-fvgf) | HIGH | Twitch allowFrom Not Enforced in Optional Plugin | - | pending | - |
| [GHSA-3hcm-ggvf-rch5](https://github.com/openclaw/openclaw/security/advisories/GHSA-3hcm-ggvf-rch5) | HIGH | Exec Allowlist Bypass via Command Substitution/Backticks | CWE-78 | >= v2026.2.2 | @simecek |
| [GHSA-mr32-vwc2-5j6h](https://github.com/openclaw/openclaw/security/advisories/GHSA-mr32-vwc2-5j6h) | HIGH | Browser Relay /cdp WebSocket Missing Auth | - | >= v2026.2.1 | @johnatzeropath, @LeftenantZero, @yueyueL |
| [GHSA-8jpq-5h99-ff5r](https://github.com/openclaw/openclaw/security/advisories/GHSA-8jpq-5h99-ff5r) | HIGH | Local File Disclosure via sendMediaFeishu in Feishu Extension | CWE-22 | pending | - |
| [GHSA-xw4p-pw82-hqr7](https://github.com/openclaw/openclaw/security/advisories/GHSA-xw4p-pw82-hqr7) | HIGH | Sandbox Skill Mirroring Path Traversal | CWE-22 | pending | - |
| [GHSA-7q2j-c4q5-rm27](https://github.com/openclaw/openclaw/security/advisories/GHSA-7q2j-c4q5-rm27) | HIGH | macOS Deep Link Confirmation Truncation | CWE-451 | pending | - |
| [GHSA-j27p-hq53-9wgc](https://github.com/openclaw/openclaw/security/advisories/GHSA-j27p-hq53-9wgc) | HIGH | DoS via Unbounded URL-Backed Media Fetch | CWE-400 | pending | - |
| [GHSA-q447-rj3r-2cgh](https://github.com/openclaw/openclaw/security/advisories/GHSA-q447-rj3r-2cgh) | HIGH | DoS via Unbounded Webhook Request Body Buffering | CWE-400 | pending | - |
| [GHSA-3fqr-4cg8-h96q](https://github.com/openclaw/openclaw/security/advisories/GHSA-3fqr-4cg8-h96q) | HIGH | CSRF Through Loopback Browser Mutation Endpoints | CWE-352 | pending | - |
| [GHSA-rq6g-px6m-c248](https://github.com/openclaw/openclaw/security/advisories/GHSA-rq6g-px6m-c248) | HIGH | Google Chat Webhook Cross-Account Policy Misrouting | CWE-284, CWE-639 | pending | - |
| [GHSA-pv58-549p-qh99](https://github.com/openclaw/openclaw/security/advisories/GHSA-pv58-549p-qh99) | HIGH | Discovery TXT Records Could Steer Routing and TLS Pinning | CWE-345 | pending | - |
| [GHSA-cv7m-c9jx-vg7q](https://github.com/openclaw/openclaw/security/advisories/GHSA-cv7m-c9jx-vg7q) | HIGH | Path Traversal in Browser Upload | CWE-22 | pending | - |
| [GHSA-g6q9-8fvw-f7rf](https://github.com/openclaw/openclaw/security/advisories/GHSA-g6q9-8fvw-f7rf) | HIGH | Gateway Tool Unrestricted gatewayUrl Override | CWE-918 | pending | - |
| [GHSA-4hg8-92x6-h2f3](https://github.com/openclaw/openclaw/security/advisories/GHSA-4hg8-92x6-h2f3) | HIGH | Missing Webhook Auth in Telnyx Provider | CWE-306 | pending | - |
| [GHSA-jmm5-fvh5-gf4p](https://github.com/openclaw/openclaw/security/advisories/GHSA-jmm5-fvh5-gf4p) | MEDIUM | Non-Constant-Time Token Comparison in Hooks Auth | CWE-208 | pending | - |
| [GHSA-47q7-97xp-m272](https://github.com/openclaw/openclaw/security/advisories/GHSA-47q7-97xp-m272) | MEDIUM | Config Writes Persist Resolved ${VAR} Secrets to Disk | - | pending | - |
| [GHSA-w2cg-vxx6-5xjg](https://github.com/openclaw/openclaw/security/advisories/GHSA-w2cg-vxx6-5xjg) | MEDIUM | DoS via Large Base64 Media Buffer Allocation | CWE-400 | pending | - |
| [GHSA-h89v-j3x9-8wqj](https://github.com/openclaw/openclaw/security/advisories/GHSA-h89v-j3x9-8wqj) | MEDIUM | DoS via Unguarded Archive Extraction (ZIP/TAR) | CWE-400 | pending | - |
| [GHSA-mj5r-hh7j-4gxf](https://github.com/openclaw/openclaw/security/advisories/GHSA-mj5r-hh7j-4gxf) | MEDIUM | Telegram Allowlist Accepted Mutable Usernames | CWE-284, CWE-290 | pending | - |
| [GHSA-g34w-4xqq-h79m](https://github.com/openclaw/openclaw/security/advisories/GHSA-g34w-4xqq-h79m) | MEDIUM | iMessage Group Allowlist Cross-Context Authorization | CWE-284, CWE-863 | pending | - |
| [GHSA-8mh7-phf8-xgfm](https://github.com/openclaw/openclaw/security/advisories/GHSA-8mh7-phf8-xgfm) | MEDIUM | skills.status Leaks Secrets to operator.read Clients | CWE-200 | pending | - |
| [GHSA-c37p-4qqg-3p76](https://github.com/openclaw/openclaw/security/advisories/GHSA-c37p-4qqg-3p76) | MEDIUM | Twilio Voice Webhook Auth Bypass via ngrok Loopback | CWE-306 | pending | - |
| [GHSA-pg2v-8xwh-qhcc](https://github.com/openclaw/openclaw/security/advisories/GHSA-pg2v-8xwh-qhcc) | MEDIUM | SSRF in Optional Tlon (Urbit) Extension Auth | CWE-918 | pending | - |

### CVE-2026-24763: Docker PATH Command Injection

**GHSA:** GHSA-mc68-q9jw-2h3v
**Severity:** HIGH (CWE-78: OS Command Injection)
**Affected:** ≤ v2026.1.24
**Credits:** @berkdedekarginoglu

**Description:** Unsafe handling of the PATH environment variable when constructing shell commands in Docker sandbox execution. Authenticated users who could control environment variables could influence command execution within the container context.

**Impact:** Execution of unintended commands inside the container, access to container filesystem and environment variables, exposure of sensitive data.

**Fix:** Commit `771f23d` moved `setupCommand` PATH handling from shell string interpolation to a container env var. See [Post-merge hardening (PR #1)](./post-merge-hardening.md#post-merge-hardening-pr-1-129-upstream-commits).

### GHSA-g8p2-7wf7-98mq: gatewayUrl Token Exfiltration

**Severity:** HIGH (CWE-200: Exposure of Sensitive Information)
**Affected:** ≤ v2026.1.28
**Credits:** DepthFirstDisclosures, @0xacb, @mavlevin

**Description:** The Control UI trusted `gatewayUrl` from query string without validation and auto-connected on load, sending the stored gateway token in the WebSocket connect payload. Clicking a crafted link could send the token to an attacker-controlled server.

**Impact:** Full gateway compromise. The attacker gains operator-level access to the gateway API, enabling arbitrary config changes and code execution. Works even when gateway binds to loopback because the victim's browser acts as the bridge.

**Fix:** Control UI now requires user confirmation before connecting to a new gateway URL (`ui/src/ui/views/gateway-url-confirmation.ts`).

### GHSA-q284-4pvr-m585: sshNodeCommand Injection

**Severity:** HIGH
**Affected:** < v2026.1.29
**Credits:** @koko9xxx

**Description:** Two related vulnerabilities in the macOS app's SSH remote connection handling (`apps/macos/Sources/OpenClaw/CommandResolver.swift`):
1. `sshNodeCommand` constructed shell script without escaping user-supplied project path in error messages
2. `parseSSHTarget` did not validate that SSH targets couldn't begin with a dash

**Impact:** Arbitrary code execution on either the user's local machine or configured remote SSH host.

**Affected component:** macOS menubar application (Remote/SSH mode only). Not affected: CLI, web gateway, iOS/Android apps, Local mode users.

**Fix:** Commit `06289b36d` validates SSH targets and escapes paths.

### CVE-2026-25593: Unauthenticated Local RCE via WebSocket config.apply

**GHSA:** GHSA-g55j-c2v4-pjcg
**Severity:** HIGH (CWE-20: Improper Input Validation, CWE-78: OS Command Injection, CWE-306: Missing Authentication)
**Affected:** < v2026.1.20
**Credits:** @hackerman70000

**Description:** Unauthenticated local WebSocket client could write an unsafe `cliPath` value via the `config.apply` method, enabling command injection. An attacker on the same machine could connect to the gateway's WebSocket endpoint and modify configuration to inject arbitrary commands.

**Impact:** Local privilege escalation and arbitrary code execution as the OpenClaw gateway process user.

**Affected component:** Gateway WebSocket API. Requires local network access to gateway port.

**Fix:** Gateway now validates `cliPath` and requires authentication for config modification methods.

### CVE-2026-25475: Local File Inclusion via MEDIA: Path Extraction

**GHSA:** GHSA-r8g4-86fx-92mq
**Severity:** MEDIUM (CWE-22: Path Traversal, CWE-200: Information Exposure)
**Affected:** < v2026.1.30
**Credits:** @jasonsutter87

**Description:** The `isValidMedia()` function in `src/media/parse.ts` (now at lines 36-64) previously accepted arbitrary paths including system files like `/etc/passwd`, `~/.ssh/id_rsa`, and traversal paths like `../../../etc/passwd`. An attacker could craft a message containing a `MEDIA:` reference pointing to sensitive local files. Fixed: `isValidMedia()` now accepts all path types but `assertLocalMediaAllowed()` (`src/web/media.ts:47-89`) enforces directory root guards at load time.

**Impact:** Exposure of sensitive local files (credentials, configuration, SSH keys) through the media pipeline.

**Affected component:** Media path parsing in `src/media/parse.ts`.

**Fix:** Media path validation now restricts extraction to the media directory and rejects traversal sequences.

### GHSA-wfp2-v9c7-fh79: SSRF via Attachment/Media URL Hydration

**Severity:** MEDIUM (CWE-918: Server-Side Request Forgery)
**Affected:** < v2026.2.2
**Credits:** —

**Description:** Server-side request forgery via attachment or media URL hydration. An attacker could craft media/attachment URLs that, when hydrated by the server, trigger requests to internal services or unintended external endpoints.

**Impact:** Access to internal services, potential information disclosure via SSRF.

**Fix:** Patched in v2026.2.2.

### CVE-2026-24764: Slack Integration Hardening — Channel Metadata Injection

**GHSA:** GHSA-782p-5fr5-7fj8
**Severity:** LOW (CWE-74: Injection, CWE-94: Code Injection)
**Affected:** < v2026.2.3
**Credits:** —

**Description:** Slack channel metadata (topic, purpose, channel name) could influence system prompts, potentially allowing prompt injection via channel configuration fields that are reflected into the agent's context.

**Impact:** Prompt injection via Slack channel metadata. Low severity due to requiring Slack workspace admin access to modify channel metadata.

**Fix:** Patched in v2026.2.3. Channel metadata is now sanitized before inclusion in system prompts.

### GHSA-mv9j-6xhh-g383: Unauthenticated Nostr Profile HTTP Endpoints

**Severity:** HIGH
**Published:** 2026-02-14
**Credits:** —

**Description:** Unauthenticated Nostr profile HTTP endpoints allow remote profile and configuration tampering. An attacker could modify Nostr profile settings without authentication via the HTTP API endpoints.

**Impact:** Remote profile/config tampering via unauthenticated HTTP requests.

**Fix:** Pending. Related hardening: `3e0e78f82` (fix(nostr): guard profile mutations) adds mutation guards in `extensions/nostr/src/nostr-profile-http.ts`. See [Feb 15 sync 5](./post-merge-hardening/2026-02-15-sync-5.md).

### GHSA-3m3q-x3gj-f79x: Voice-Call Webhook Verification Bypass Behind Proxy

**Severity:** MEDIUM
**Published:** 2026-02-14
**Credits:** —

**Description:** The optional voice-call plugin's webhook verification may be bypassed when the gateway is deployed behind certain proxy configurations that alter or strip request headers used for signature validation.

**Impact:** Webhook authentication bypass in proxy configurations, potentially allowing unauthorized voice call events to be processed.

**Fix:** Pending. Related prior hardening: `a749db982` (Feb 4 sync 3) enhanced webhook verification with provider-specific validation for Twilio and Plivo. Further hardened by `29b587e73` (Telnyx fail-closed) and `ff11d8793` (Twilio ngrok signature enforcement) in [Feb 15 sync 6](./post-merge-hardening/2026-02-15-sync-6.md).

### GHSA-g27f-9qjv-22pm: Log Poisoning via WebSocket Headers

**Severity:** LOW
**Published:** 2026-02-14
**Credits:** —

**Description:** OpenClaw log poisoning (indirect prompt injection) via WebSocket headers. Crafted WebSocket connection headers could inject content into gateway logs, potentially influencing agent behavior if logs are consumed by the AI agent.

**Impact:** Indirect prompt injection via log content. Low severity due to requiring WebSocket connection access and the indirect nature of the injection path.

**Fix:** Pending.

### GHSA-pchc-86f6-8758: BlueBubbles Webhook Auth Bypass via Loopback Proxy Trust

**Severity:** HIGH
**Published:** 2026-02-14
**Credits:** @MegaManSec

**Description:** The optional BlueBubbles iMessage channel plugin could accept webhook requests as authenticated based only on the TCP peer address being loopback (`127.0.0.1`, `::1`, `::ffff:127.0.0.1`) even when the configured webhook secret was missing or incorrect. If a deployment exposes the BlueBubbles webhook endpoint through a same-host reverse proxy (or an attacker reaches loopback via SSRF), an unauthenticated party may be able to inject inbound webhook events into the agent pipeline.

**Impact:** Webhook authentication bypass in loopback proxy configurations, potentially allowing unauthorized BlueBubbles events to be processed.

**Fix:** Fixed in v2026.2.13. Commits `f836c385f`, `743f4b284` (defense-in-depth). Related hardening: `71f357d94` (LFI path hardening in BlueBubbles media-send). See [Feb 15 sync 6](./post-merge-hardening/2026-02-15-sync-6.md).

### GHSA-fhvm-j76f-qmjv: Access-Group Authorization Bypass if Channel Type Lookup Fails

**Severity:** HIGH (CWE-285: Improper Authorization)
**Affected:** < v2026.2.2
**Credits:** @simecek

**Description:** Access-group authorization bypass if channel type lookup fails. Originally disclosed as Telegram webhook auth bypass (missing secret), the advisory was updated to reflect a broader access-group authorization issue: if channel type resolution fails during authorization, the authorization check may be bypassed entirely.

**Impact:** Authorization bypass — unauthorized users could interact with the agent when channel type lookup fails.

**Fix:** Commit `ca92597e1`. Authorization handler now fails closed when channel type cannot be resolved.

### GHSA-rmxw-jxxx-4cpc: Matrix Allowlist Bypass via displayName

**Severity:** MEDIUM
**Affected:** < v2026.2.2
**Credits:** @MegaManSec

**Description:** The Matrix channel plugin's allowlist matching accepted display names in addition to full MXIDs (`@user:server`). An attacker could set their display name to match an allowed user's MXID, bypassing the allowlist restriction.

**Impact:** Allowlist bypass in Matrix channel — unauthorized users could interact with the agent by spoofing display names.

**Fix:** Commit `8f3bfbd1c`. Matrix allowlist now requires full MXIDs (`@user:server`) and only accepts single exact matches from directory search. See [Feb 4 sync 1](./post-merge-hardening/2026-02-04-sync-1.md).

### GHSA-4rj2-gpmh-qq5x: Inbound Allowlist Policy Bypass in Voice-Call Extension

**Severity:** CRITICAL (CWE-287: Improper Authentication)
**Affected:** <= v2026.2.1
**Published:** 2026-02-14
**Credits:** @simecek, @MegaManSec (reporters); @stanislavfortaisle (analyst)

**Description:** The optional `voice-call` extension's inbound allowlist check in `extensions/voice-call/src/manager.ts` used suffix-based matching and accepted empty caller IDs after normalization. Two bypasses: (1) missing/empty `from` values normalized to an empty string caused the allowlist predicate to evaluate as allowed; (2) suffix-based matching meant any caller number whose digits ended with an allowlisted number would be accepted.

**Impact:** Unauthorized callers could reach auto-response and tool execution when `inboundPolicy=allowlist` or `pairing` was configured. Only affects deployments with the optional voice-call extension enabled.

**Fix:** Commit `f8dfd034f`. Reject inbound calls when caller ID is missing, require strict equality for normalized caller IDs, add regression tests for missing/anonymous caller ID and suffix-collision cases. Patched in >= v2026.2.2.

### CVE-2026-25474 / GHSA-mp5h-m6qj-6292: Telegram Webhook Secret Verification Bypass

**Severity:** HIGH (CWE-345: Insufficient Verification of Data Authenticity)
**Affected:** <= v2026.1.30
**Published:** 2026-02-14
**Credits:** @yueyueL

**Description:** In Telegram webhook mode, if `channels.telegram.webhookSecret` is not set, OpenClaw accepted webhook HTTP requests without verifying Telegram's `X-Telegram-Bot-Api-Secret-Token` header. An attacker who can reach the webhook endpoint could send forged updates processed as if from Telegram. Note: Telegram webhook mode requires explicit configuration of `channels.telegram.webhookUrl`.

**Impact:** Forged Telegram updates could trigger unintended bot actions, depending on enabled commands/tools and configuration.

**Fix:** Commit `ca92597e1` (config validation: `webhookUrl` requires `webhookSecret`). Defense-in-depth: `5643a934` (default webhook bind to loopback), `3cbcba10` (bound request body size/time), `633fe8b9` (runtime guard: reject webhook startup when secret is missing/empty). Patched in >= v2026.2.1.

### GHSA-7vwx-582j-j332: MS Teams Attachment Downloader Leaks Bearer Tokens

**Severity:** HIGH (CWE-201: Insertion of Sensitive Information Into Sent Data)
**Affected:** <= v2026.1.30
**Published:** 2026-02-14
**Credits:** @yueyueL

**Description:** When downloading inbound MS Teams attachments/inline images, OpenClaw may retry a URL with an `Authorization: Bearer <token>` header after receiving 401/403. Because the default download allowlist uses suffix matching (including some multi-tenant suffix domains), a message referencing an untrusted but allowlisted host could cause the bearer token to be sent to the wrong place. Only affects deployments with the optional MS Teams extension enabled.

**Impact:** Bearer token leakage to attacker-controlled domains matching suffix-based allowlist entries.

**Fix:** Commit `41cc5bcd4`. Patched in >= v2026.2.1. Workaround: ensure auth host allowlist is strict (only Microsoft-owned endpoints) and avoid wildcard/broad suffix entries.

### GHSA-r5h9-vjqc-hq3r: Nextcloud Talk Allowlist Bypass via actor.name Spoofing

**Severity:** HIGH
**Affected:** @openclaw/nextcloud-talk <= v2026.2.2
**Published:** 2026-02-14
**Credits:** @MegaManSec

**Description:** In the optional Nextcloud Talk plugin, allowlist matching accepted equality on the mutable `actor.name` (display name) field from webhook payloads, in addition to the stable `actor.id`. An attacker could change their Nextcloud display name to match an allowlisted user ID and bypass DM or room allowlists. Core OpenClaw is not impacted unless the Nextcloud Talk plugin is installed.

**Impact:** Allowlist bypass — unauthorized users could interact with the agent by spoofing display names in Nextcloud Talk.

**Fix:** Commit `6b4b6049b`. Allowlist matching now uses only `actor.id` (stable identifier). Patched in @openclaw/nextcloud-talk >= v2026.2.6.

### GHSA-33rq-m5x2-fvgf: Twitch allowFrom Not Enforced in Optional Plugin

**Severity:** HIGH (CWE-285: Improper Authorization)
**Affected:** >= v2026.1.29, < v2026.2.1
**Published:** 2026-02-14
**Credits:** @MegaManSec

**Description:** In the optional Twitch channel plugin (`extensions/twitch/src/access-control.ts`), `allowFrom` was documented as a hard allowlist of Twitch user IDs but was not enforced as a hard gate. When `allowedRoles` was unset or empty, the access control path defaulted to allow, so any Twitch user who could mention the bot could reach the agent dispatch pipeline. Only affects deployments with the Twitch plugin enabled.

**Impact:** Authorization bypass — any Twitch user could invoke the bot regardless of `allowFrom` configuration, potentially leading to unintended actions and resource/cost exhaustion.

**Fix:** Commit `8c7901c98`. `checkTwitchAccessControl()` now returns `allowed: false` for non-members when `allowFrom` is configured. Patched in >= v2026.2.1.

### GHSA-qrq5-wjgg-rvqw: Path Traversal in Plugin Installation

**Severity:** CRITICAL (CWE-22: Path Traversal)
**Affected:** < v2026.2.1
**Published:** 2026-02-14
**Credits:** @logicx24

**Description:** During plugin installation, crafted plugin archive paths could traverse outside the intended plugin directory. A malicious plugin package could write files to arbitrary locations on the filesystem via path traversal sequences in archive entry names.

**Impact:** Arbitrary file write on the host filesystem during plugin installation, potentially leading to code execution.

**Fix:** Commit `d03eca84`. Plugin installation now validates archive entry paths and rejects traversal sequences. Patched in >= v2026.2.1.

### GHSA-rv39-79c4-7459: Gateway Device Identity Skip

**Severity:** HIGH (CWE-306: Missing Authentication for Critical Function)
**Affected:** < v2026.2.2
**Published:** 2026-02-14
**Credits:** @simecek

**Description:** The gateway device authentication flow could be bypassed under certain conditions, allowing unauthenticated devices to interact with the gateway API without completing the device identity verification step.

**Impact:** Unauthenticated device access to the gateway API, potentially enabling unauthorized agent interactions and configuration access.

**Fix:** Commit `fe81b1d7`. Gateway now enforces device identity verification on all authenticated endpoints. Patched in >= v2026.2.2.

### GHSA-qj77-c3c8-9c3q: Windows cmd.exe Exec Allowlist Bypass

**Severity:** HIGH (CWE-78: OS Command Injection)
**Affected:** < v2026.2.2
**Published:** 2026-02-14
**Credits:** @simecek

**Description:** On Windows, the exec allowlist could be bypassed by invoking commands through `cmd.exe` with specific argument patterns that the allowlist regex did not account for. This allowed execution of commands that should have been blocked by the configured allowlist.

**Impact:** Execution of blocked commands on Windows deployments, bypassing the exec safety allowlist intended to restrict agent tool use.

**Fix:** Commit `a7f4a53ce`. Windows command parsing now normalizes `cmd.exe` invocations before allowlist evaluation. Patched in >= v2026.2.2.

### GHSA-mqpw-46fh-299h: operator.write Exec Approval Bypass via /approve

**Severity:** HIGH (CWE-269: Improper Privilege Management, CWE-863: Incorrect Authorization)
**Affected:** < v2026.2.2
**Published:** 2026-02-14
**Credits:** @yueyueL

**Description:** An operator with `operator.write` permission could bypass exec approval requirements by using the `/approve` command to auto-approve pending tool executions. The `/approve` endpoint did not verify that the approver had the necessary permission level to approve execution of the specific tool being requested.

**Impact:** Privilege escalation — operators with write-only access could approve arbitrary command executions that should require higher-level approval.

**Fix:** Commit `efe2a464`. The `/approve` endpoint now verifies the approver's permission level against the tool's required approval tier. Patched in >= v2026.2.2.

### GHSA-3hcm-ggvf-rch5: Exec Allowlist Bypass via Command Substitution/Backticks

**Severity:** HIGH (CWE-78: OS Command Injection)
**Affected:** <= v2026.2.1
**Published:** 2026-02-14
**Credits:** @simecek

**Description:** The exec approvals allowlist could be bypassed using unescaped `$()` command substitution or backticks inside double-quoted strings. Only affects setups that explicitly enable the optional exec approvals allowlist feature. Default installs are unaffected.

**Impact:** Execution of blocked commands by embedding command substitution syntax in allowlisted command arguments, bypassing the exec safety allowlist.

**Fix:** Commit `d1ecb4607`. Rejects unescaped `$()` and backticks inside double quotes during allowlist analysis. Patched in >= v2026.2.2.

### GHSA-mr32-vwc2-5j6h: Browser Relay /cdp WebSocket Missing Auth

**Severity:** HIGH
**Affected:** openclaw >= v2026.1.20, < v2026.2.1; moltbot <= v0.1.0
**Published:** 2026-02-14
**Credits:** @johnatzeropath, @LeftenantZero, @yueyueL

**Description:** The Browser Relay `/cdp` WebSocket endpoint at `ws://127.0.0.1:18792/cdp` verified the TCP peer was loopback but did not require a shared secret and did not block browser-initiated cross-origin requests. A website running in the browser could connect to the local relay via loopback WebSocket and use CDP to access cookies from other open tabs and run JavaScript in their context. Requires the Browser Relay extension to be installed and active, and the user to visit an untrusted site.

**Impact:** Disclosure of sensitive information (session cookies from other tabs), JavaScript execution in other tab contexts.

**Fix:** Commit `a1e89afcc`. Now requires per-instance `x-openclaw-relay-token` header, rejects `/cdp` upgrades when Origin is present but not `chrome-extension://`, and refuses connections unless the extension is connected. Patched in >= v2026.2.1. See also [PR #11 hardening](./post-merge-hardening.md#post-merge-hardening-pr-11--21-commits).

### GHSA-8jpq-5h99-ff5r: Local File Disclosure via Feishu Extension

**Severity:** HIGH (CWE-22: Path Traversal)
**Published:** 2026-02-15

**Description:** The `sendMediaFeishu` function in the optional Feishu extension could be abused to read arbitrary local files by manipulating the media path parameter. Requires Feishu extension to be installed and active.

**Impact:** Disclosure of local files accessible to the OpenClaw process.

### GHSA-xw4p-pw82-hqr7: Sandbox Skill Mirroring Path Traversal

**Severity:** HIGH (CWE-22: Path Traversal)
**Published:** 2026-02-15

**Description:** The sandbox skill mirroring feature could be tricked into writing files outside the designated sandbox workspace directory via path traversal in skill file paths.

**Impact:** Arbitrary file write outside sandbox boundaries, potentially overwriting system or configuration files.

### GHSA-7q2j-c4q5-rm27: macOS Deep Link Confirmation Truncation

**Severity:** HIGH (CWE-451: UI Misrepresentation)
**Published:** 2026-02-15

**Description:** The macOS deep link confirmation dialog could truncate the displayed agent message, concealing the actual command or action to be executed. An attacker could craft a message with benign-looking text at the start followed by malicious content beyond the truncation point.

**Impact:** User approves execution of a concealed malicious agent message via truncated deep link dialog.

### GHSA-j27p-hq53-9wgc: DoS via Unbounded URL-Backed Media Fetch

**Severity:** HIGH (CWE-400: Uncontrolled Resource Consumption)
**Published:** 2026-02-15

**Description:** URL-backed media fetch operations did not enforce size limits, allowing an attacker to trigger downloads of arbitrarily large files that exhaust memory or disk.

**Impact:** Denial of service via memory/disk exhaustion through crafted media URLs.

### GHSA-q447-rj3r-2cgh: DoS via Unbounded Webhook Body Buffering

**Severity:** HIGH (CWE-400: Uncontrolled Resource Consumption)
**Published:** 2026-02-15

**Description:** Webhook request body buffering did not enforce size limits, allowing arbitrarily large POST bodies to exhaust server memory.

**Impact:** Denial of service via memory exhaustion through oversized webhook payloads.

### GHSA-3fqr-4cg8-h96q: CSRF Through Loopback Browser Mutation Endpoints

**Severity:** HIGH (CWE-352: Cross-Site Request Forgery)
**Published:** 2026-02-15

**Description:** Browser mutation endpoints accessible via loopback did not validate Origin headers, enabling cross-site request forgery from malicious websites when the user visits them with OpenClaw running locally.

**Impact:** Unauthorized state mutations (config changes, command execution) via CSRF when visiting malicious websites.

### GHSA-rq6g-px6m-c248: Google Chat Webhook Cross-Account Misrouting

**Severity:** HIGH (CWE-284, CWE-639: Authorization Bypass, IDOR)
**Published:** 2026-02-15

**Description:** Google Chat's shared webhook path structure created ambiguity in webhook target resolution, allowing cross-account policy-context misrouting when multiple Google Chat integrations share a deployment.

**Impact:** Webhook messages could be routed to the wrong account's policy context, bypassing access controls.

### GHSA-pv58-549p-qh99: Discovery TXT Records Routing Manipulation

**Severity:** HIGH (CWE-345: Insufficient Verification of Data Authenticity)
**Published:** 2026-02-15

**Description:** Unauthenticated DNS TXT discovery records could be injected to steer gateway routing and TLS pinning decisions, potentially redirecting traffic to attacker-controlled endpoints.

**Impact:** Traffic redirection and TLS downgrade via spoofed discovery records.

### GHSA-cv7m-c9jx-vg7q: Path Traversal in Browser Upload

**Severity:** HIGH (CWE-22: Path Traversal)
**Published:** 2026-02-15

**Description:** The browser upload endpoint did not properly validate file paths, allowing path traversal sequences to read arbitrary local files accessible to the OpenClaw process.

**Impact:** Disclosure of local files via crafted upload path parameters.

### GHSA-g6q9-8fvw-f7rf: Gateway Tool Unrestricted gatewayUrl Override

**Severity:** HIGH (CWE-918: SSRF)
**Published:** 2026-02-15

**Description:** The gateway tool allowed unrestricted `gatewayUrl` override, enabling SSRF by redirecting gateway requests to arbitrary URLs including internal network endpoints.

**Impact:** Server-side request forgery to internal services, potential credential exfiltration via URL parameters.

### GHSA-4hg8-92x6-h2f3: Missing Webhook Auth in Telnyx Provider

**Severity:** HIGH (CWE-306: Missing Authentication)
**Published:** 2026-02-15

**Description:** The Telnyx telephony provider extension did not verify webhook request authenticity, allowing unauthenticated requests to trigger agent actions.

**Impact:** Unauthorized message injection and agent command execution via forged Telnyx webhooks.

### GHSA-jmm5-fvh5-gf4p: Non-Constant-Time Token Comparison in Hooks

**Severity:** MEDIUM (CWE-208: Observable Timing Discrepancy)
**Published:** 2026-02-15

**Description:** Hook authentication token comparison used standard string equality instead of constant-time comparison, potentially leaking token characters through timing side-channel.

**Impact:** Theoretical token recovery via timing attack (requires many requests with precise timing measurement).

### GHSA-47q7-97xp-m272: Config Writes Persist Resolved Secrets

**Severity:** MEDIUM
**Published:** 2026-02-15

**Description:** Config write operations could persist resolved `${VAR}` environment variable values to disk in plaintext, instead of preserving the variable reference syntax.

**Impact:** Secret values from environment variables written to config files in plaintext, surviving beyond the process lifetime.

### GHSA-w2cg-vxx6-5xjg: DoS via Large Base64 Media Buffers

**Severity:** MEDIUM (CWE-400: Uncontrolled Resource Consumption)
**Published:** 2026-02-15

**Description:** Large base64-encoded media files triggered buffer allocation before size limit checks, allowing memory exhaustion.

**Impact:** Denial of service via memory allocation triggered by oversized base64 media payloads.

### GHSA-h89v-j3x9-8wqj: DoS via Unguarded Archive Extraction

**Severity:** MEDIUM (CWE-400: Uncontrolled Resource Consumption)
**Published:** 2026-02-15

**Description:** Archive extraction (ZIP/TAR) did not enforce expansion ratio limits, allowing zip bombs or tar archives with high compression ratios to exhaust disk/memory.

**Impact:** Denial of service via resource exhaustion through crafted archives.

### GHSA-mj5r-hh7j-4gxf: Telegram Allowlist Mutable Username Bypass

**Severity:** MEDIUM (CWE-284, CWE-290: Authorization Bypass, Spoofing)
**Published:** 2026-02-15

**Description:** The Telegram allowlist authorization accepted mutable usernames as identifiers. Since Telegram usernames can be changed, an authorized user could transfer their username to an unauthorized user who would then pass the allowlist check.

**Impact:** Allowlist bypass via Telegram username transfer to unauthorized parties.

### GHSA-g34w-4xqq-h79m: iMessage Group Allowlist Cross-Context Authorization

**Severity:** MEDIUM (CWE-284, CWE-863: Authorization Bypass)
**Published:** 2026-02-15

**Description:** iMessage group allowlist authorization inherited DM pairing-store identities, enabling cross-context command authorization where a user authorized in DMs could execute commands in group contexts they shouldn't have access to.

**Impact:** Command execution in unauthorized group contexts via inherited DM authorization.

### GHSA-8mh7-phf8-xgfm: skills.status Secrets Leak

**Severity:** MEDIUM (CWE-200: Information Disclosure)
**Published:** 2026-02-15

**Description:** The `skills.status` endpoint could leak secret configuration values to clients with only `operator.read` scope, which should not have access to sensitive configuration data.

**Impact:** Disclosure of secret configuration values to lower-privileged gateway clients.

### GHSA-c37p-4qqg-3p76: Twilio Voice Webhook Auth Bypass

**Severity:** MEDIUM (CWE-306: Missing Authentication)
**Published:** 2026-02-15

**Description:** The Twilio voice-call webhook verification could be bypassed when ngrok loopback compatibility mode was enabled, as the loopback detection overrode webhook signature verification.

**Impact:** Unauthenticated voice-call webhook injection when using ngrok loopback mode.

### GHSA-pg2v-8xwh-qhcc: SSRF in Tlon (Urbit) Extension

**Severity:** MEDIUM (CWE-918: SSRF)
**Published:** 2026-02-15

**Description:** The optional Tlon (Urbit) extension authentication flow was vulnerable to SSRF, allowing requests to internal network endpoints through the authentication redirect mechanism.

**Impact:** Server-side request forgery to internal services via Tlon authentication redirect.

### Relationship to Third-Party Audits

These official CVEs are **distinct from** the two third-party security audits documented below:
- [Issue #1796 (Argus)](./issue-1796-argus-audit.md) — Automated scanner report (0/8 exploitable)
- [Medium Article (Saad Khalid)](./medium-article-audit.md) — Manual pentest claims (0/8 exploitable)

The official CVEs were responsibly disclosed through GitHub Security Advisories and patched before public disclosure. The third-party audits contain false positives, design observations, and overstated claims (see analysis sections).
