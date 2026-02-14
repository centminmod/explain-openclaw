# Detecting OpenClaw Requests (for hosting services)

## Overview

OpenClaw sets various identifying headers on outgoing HTTP requests across multiple subsystems. This document catalogues every identifiable header, grouped by detectability, with source code references.

**Who this is for:**
- **Hosting services / API providers:** Want to identify, rate-limit, or block OpenClaw traffic
- **OpenClaw users:** Want to understand what your instance reveals about itself in outgoing requests

All file paths are relative to the repository root. Line numbers are verified against the current codebase.

---

## 1. Explicitly identifiable requests

These requests contain headers that directly name OpenClaw. Any service receiving them can trivially identify the source.

### `User-Agent: "OpenClaw-Gateway/1.0"` — Media file fetches

When OpenClaw downloads media files (images, audio, video) attached to incoming messages, the Gateway sets:

```
User-Agent: OpenClaw-Gateway/1.0
```

**Source:** `src/media/input-files.ts:149`
```typescript
init: { headers: { "User-Agent": "OpenClaw-Gateway/1.0" } },
```

**Who sees this:** Any web server hosting media files that users share with OpenClaw.

### `User-Agent: "openclaw"` — GitHub API (signal-cli install)

When auto-installing the Signal CLI dependency, OpenClaw fetches release info from GitHub:

```
User-Agent: openclaw
Accept: application/vnd.github+json
```

**Source:** `src/commands/signal-install.ts:124`
```typescript
headers: {
  "User-Agent": "openclaw",
  Accept: "application/vnd.github+json",
},
```

**Who sees this:** GitHub API servers (only during signal-cli installation).

### `User-Agent: "openclaw"` — Anthropic OAuth API

When checking Anthropic usage via OAuth:

```
User-Agent: openclaw
```

**Source:** `src/infra/provider-usage.fetch.claude.ts:119`
```typescript
"User-Agent": "openclaw",
```

**Who sees this:** Anthropic API servers.

### `HTTP-Referer` + `X-Title` — OpenRouter / Perplexity API

OpenClaw identifies itself to OpenRouter and Perplexity via app attribution headers:

```
HTTP-Referer: https://openclaw.ai
X-Title: OpenClaw
```

**Source:** `src/agents/pi-embedded-runner/extra-params.ts:8-9`
```typescript
const OPENROUTER_APP_HEADERS: Record<string, string> = {
  "HTTP-Referer": "https://openclaw.ai",
  "X-Title": "OpenClaw",
};
```

For Perplexity web search specifically, the title is more descriptive:

```
HTTP-Referer: https://openclaw.ai
X-Title: OpenClaw Web Search
```

**Source:** `src/agents/tools/web-search.ts:481-482`
```typescript
"HTTP-Referer": "https://openclaw.ai",
"X-Title": "OpenClaw Web Search",
```

**Who sees this:** OpenRouter and Perplexity API servers.

### `MM-API-Source: "OpenClaw"` — MiniMax VLM API

OpenClaw uses a custom header when calling MiniMax's vision-language model API:

```
MM-API-Source: OpenClaw
```

**Source:** `src/agents/minimax-vlm.ts:76`
```typescript
"MM-API-Source": "OpenClaw",
```

Also used for MiniMax usage checking:

**Source:** `src/infra/provider-usage.fetch.minimax.ts:322`
```typescript
"MM-API-Source": "OpenClaw",
```

**Who sees this:** MiniMax API servers.

### `clientInfo.name: "openclaw-acp-client"` — ACP protocol

When OpenClaw connects to an ACP (Agent Communication Protocol) server, it identifies itself at the protocol level:

```json
{ "name": "openclaw-acp-client", "version": "1.0.0" }
```

**Source:** `src/acp/client.ts:129`
```typescript
clientInfo: { name: "openclaw-acp-client", version: "1.0.0" },
```

**Who sees this:** Any ACP-compatible agent server that OpenClaw connects to.

---

## 2. Browser-like requests (not easily detectable)

### WebFetch tool — browsing websites

When the WebFetch tool fetches web pages, it uses a Chrome-like User-Agent and browser-typical headers:

```
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36
Accept: */*
Accept-Language: en-US,en;q=0.9
```

**Source:** `src/agents/tools/web-fetch.ts:40-41`
```typescript
const DEFAULT_FETCH_USER_AGENT =
  "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36";
```

**Source:** `src/agents/tools/web-fetch.ts:420-425`
```typescript
headers: {
  Accept: "text/markdown, text/html;q=0.9, */*;q=0.1",
  "User-Agent": params.userAgent,
  "Accept-Language": "en-US,en;q=0.9",
},
```

**Detection difficulty:** Hard. The request looks like a real Chrome browser on macOS. However, subtle fingerprinting hints exist:

- **Missing browser headers:** Real Chrome sends `Sec-Fetch-Mode`, `Sec-Fetch-Site`, `Sec-Fetch-Dest`, `Sec-Ch-Ua`, and `Cookie` headers. WebFetch sends none of these.
- **SSRF guard behavior:** The internal SSRF guard may produce distinctive redirect-following or timing patterns.
- **Static version:** The hardcoded Chrome 122 version string will become increasingly stale over time, making it detectable via version-age analysis.

**Configurable:** Yes — `tools.web.fetch.userAgent` in the config (`src/config/types.tools.ts:381-382`).

---

## 3. Indirect fingerprints (weak signals)

These requests don't name OpenClaw but have characteristics that differ from real browser traffic.

### Brave Search API

```
Accept: application/json
X-Subscription-Token: <api_key>
```

**Source:** `src/agents/tools/web-search.ts:660-663`
```typescript
headers: {
  Accept: "application/json",
  "X-Subscription-Token": params.apiKey,
},
```

No custom User-Agent is set. Node.js `fetch()` uses a default UA string (typically `node` or `undici`) that differs from browsers but doesn't identify OpenClaw specifically.

**Who sees this:** Brave Search API servers.

### xAI Grok API

```
Content-Type: application/json
Authorization: Bearer <api_key>
```

**Source:** `src/agents/tools/web-search.ts:527-535`
```typescript
headers: {
  "Content-Type": "application/json",
  Authorization: `Bearer ${params.apiKey}`,
},
```

No custom User-Agent or referer is set. Same Node.js `fetch()` default applies.

**Who sees this:** xAI API servers.

### Common weak signals across all Node.js fetch requests

When no explicit User-Agent is set, Node.js `fetch()` exposes:
- A non-browser User-Agent string (e.g., `node` or the `undici` library name)
- Missing `Cookie`, `Sec-*`, and `Referer` headers that real browsers always send
- Missing `Accept-Encoding` negotiation that browsers typically include

These signals indicate "automated HTTP client" but not "OpenClaw" specifically.

---

## 4. Provider-specific spoofed identities

These are internal usage-checking calls where OpenClaw impersonates other clients to query provider usage APIs. They are **not user-facing tool requests** — they only fire when checking account usage/quotas.

### GitHub Copilot usage check

```
User-Agent: GitHubCopilotChat/0.26.7
Editor-Version: vscode/1.96.2
X-Github-Api-Version: 2025-04-01
```

**Source:** `src/infra/provider-usage.fetch.copilot.ts:24`
```typescript
"User-Agent": "GitHubCopilotChat/0.26.7",
```

### OpenAI/Codex usage check

```
User-Agent: CodexBar
```

**Source:** `src/infra/provider-usage.fetch.codex.ts:30`
```typescript
"User-Agent": "CodexBar",
```

### Google/Antigravity usage check

```
User-Agent: antigravity
X-Goog-Api-Client: google-cloud-sdk vscode_cloudshelleditor/0.1
```

**Source:** `src/infra/provider-usage.fetch.antigravity.ts:205-206`
```typescript
"User-Agent": "antigravity",
"X-Goog-Api-Client": "google-cloud-sdk vscode_cloudshelleditor/0.1",
```

**Note:** These spoofed identities are relevant to the privacy fingerprint discussion because they show OpenClaw sending headers that claim to be other software. The respective providers could detect this mismatch if they correlate User-Agent strings with expected client behavior.

---

## 5. Gateway client identifiers (internal only)

The Gateway uses WebSocket-based client identification for its internal control protocol. These identifiers are **not sent over external HTTP** — they exist only in the Gateway ↔ client communication channel.

**Source:** `src/gateway/protocol/client-info.ts:1-14`
```typescript
export const GATEWAY_CLIENT_IDS = {
  WEBCHAT_UI: "webchat-ui",
  CONTROL_UI: "openclaw-control-ui",
  WEBCHAT: "webchat",
  CLI: "cli",
  GATEWAY_CLIENT: "gateway-client",
  MACOS_APP: "openclaw-macos",
  IOS_APP: "openclaw-ios",
  ANDROID_APP: "openclaw-android",
  NODE_HOST: "node-host",
  TEST: "test",
  FINGERPRINT: "fingerprint",
  PROBE: "openclaw-probe",
};
```

Each client also sends metadata:

**Source:** `src/gateway/protocol/client-info.ts:34-43`
```typescript
export type GatewayClientInfo = {
  id: GatewayClientId;
  displayName?: string;
  version: string;
  platform: string;
  deviceFamily?: string;
  modelIdentifier?: string;
  mode: GatewayClientMode;
  instanceId?: string;
};
```

**Who sees this:** Only the Gateway process itself. Not visible to external services unless the Gateway is exposed on a network and an attacker intercepts WebSocket traffic.

---

## 6. For hosting services: detection tips

### Match rules (ordered by coverage)

1. **User-Agent contains "OpenClaw" or "openclaw"** — catches media fetches (`OpenClaw-Gateway/1.0`), GitHub API calls, and Anthropic OAuth calls
2. **HTTP-Referer = `https://openclaw.ai`** — catches all OpenRouter and Perplexity-routed requests
3. **X-Title contains "OpenClaw"** — catches OpenRouter/Perplexity requests (redundant with referer but more specific)
4. **MM-API-Source = "OpenClaw"** — catches MiniMax API calls

### What you won't catch

- **WebFetch browsing traffic** — intentionally uses a Chrome-like User-Agent and standard browser headers. You would need deeper behavioral analysis (missing `Sec-*` headers, stale Chrome version, no cookies) to flag this traffic.
- **Brave Search / xAI Grok API calls** — no OpenClaw-specific headers; only detectable via generic Node.js fetch fingerprinting.

---

## 7. For OpenClaw users: controlling your fingerprint

### What you can configure

| Setting | Config path | Effect |
|---|---|---|
| WebFetch User-Agent | `tools.web.fetch.userAgent` | Overrides the Chrome-like default UA for web browsing |

### What you cannot configure (hardcoded)

| Header | Where it's set | Notes |
|---|---|---|
| `User-Agent: OpenClaw-Gateway/1.0` | `src/media/input-files.ts:149` | Media file downloads |
| `User-Agent: openclaw` | `src/commands/signal-install.ts:124` | Signal CLI installation |
| `User-Agent: openclaw` | `src/infra/provider-usage.fetch.claude.ts:119` | Anthropic usage check |
| `HTTP-Referer: https://openclaw.ai` | `src/agents/pi-embedded-runner/extra-params.ts:8` | OpenRouter/Perplexity |
| `X-Title: OpenClaw` | `src/agents/pi-embedded-runner/extra-params.ts:9` | OpenRouter/Perplexity |
| `X-Title: OpenClaw Web Search` | `src/agents/tools/web-search.ts:482` | Perplexity search |
| `MM-API-Source: OpenClaw` | `src/agents/minimax-vlm.ts:76` | MiniMax VLM |

### Recommendation

If request fingerprinting matters to your use case:
1. Review which tools and providers you have enabled — each carries its own identifying headers
2. Use `tools.web.fetch.userAgent` to set a current, realistic browser string for web browsing
3. Be aware that OpenRouter/Perplexity attribution headers are hardcoded and will always identify your traffic as OpenClaw
4. Media fetch headers are hardcoded — any server hosting files you share with OpenClaw will see `OpenClaw-Gateway/1.0`

---

## 8. Cloudflare WAF rule examples

Ready-to-use Cloudflare WAF custom rule expressions for detecting, challenging, or blocking OpenClaw traffic. These rules use [Cloudflare's Rules Language](https://developers.cloudflare.com/ruleset-engine/rules-language/) and work across all plan tiers (unless noted).

### Key expression fields

| Field | Type | Purpose |
|---|---|---|
| `http.user_agent` | String | The User-Agent request header |
| `http.referer` | String | The Referer request header |
| `http.request.headers["name"]` | Map\<Array\<String\>\> | Access any header by name (**must be lowercase**) |
| `lower()` | Function | Case-insensitive matching |
| `any()` | Function | Check if any array element matches |
| `len()` | Function | Check if header exists (`len(...) > 0`) |
| `contains` | Operator | Substring match (**case-sensitive**) |
| `matches` | Operator | Regex match — **Business and Enterprise only** ([Rust regex engine](https://docs.rs/regex/latest/regex/#syntax)) |

### Action availability by plan

| Action | API value | Plan required | Use case |
|---|---|---|---|
| Block | `block` | Free+ | Hard block OpenClaw requests |
| Managed Challenge | `managed_challenge` | Free+ | Present adaptive challenge (recommended first step) |
| JS Challenge | `js_challenge` | Free+ | Present JavaScript challenge |
| Interactive Challenge | `challenge` | Free+ | Present CAPTCHA-style challenge |
| Log | `log` | **Enterprise only** | Log without blocking — validate rule impact first |
| Skip | `skip` | Free+ | Exempt known-good OpenClaw traffic from other rules |

### Rule 1: Catch all explicitly identifiable OpenClaw requests (broad match)

```
(lower(http.user_agent) contains "openclaw") or
(http.referer contains "openclaw.ai") or
(any(http.request.headers["x-title"][*] contains "OpenClaw")) or
(any(http.request.headers["mm-api-source"][*] eq "OpenClaw"))
```

**Recommended action:** `managed_challenge` for initial deployment, then switch to `block` after reviewing Security Events.

### Rule 2: Match only User-Agent identifiers (simplest)

```
(lower(http.user_agent) contains "openclaw")
```

Catches: `OpenClaw-Gateway/1.0`, `openclaw`.

### Rule 3: Match OpenRouter/Perplexity referer attribution

```
(http.referer eq "https://openclaw.ai")
```

Catches: All requests routed through OpenRouter or Perplexity with OpenClaw attribution.

### Rule 4: Match custom header identifiers

```
(len(http.request.headers["x-title"]) > 0 and any(http.request.headers["x-title"][*] contains "OpenClaw")) or
(len(http.request.headers["mm-api-source"]) > 0 and any(http.request.headers["mm-api-source"][*] eq "OpenClaw"))
```

### Rule 5: Enterprise Log-first approach (validate before blocking)

```
(lower(http.user_agent) contains "openclaw") or
(http.referer contains "openclaw.ai") or
(any(http.request.headers["x-title"][*] contains "OpenClaw"))
```

**Action:** `log` — Enterprise plans only. Deploy as Log first, review Security Events, then switch to Block or Managed Challenge.

### Rule 6: Skip rule — allow known-good OpenClaw traffic

```
(lower(http.user_agent) contains "openclaw") and (ip.src in {203.0.113.0/24})
```

**Action:** `skip` > All remaining custom rules. Place **before** block rules in rule order. Replace the example IP range with your actual allowed IPs.

### Regex rules (Business and Enterprise only)

The `matches` operator uses the [Rust regular expression engine](https://docs.rs/regex/latest/regex/#syntax) and is available on **Business and Enterprise plans**. It can match partial values (unlike `wildcard` which matches the entire field). Test expressions at [regex101.com (Rust flavor)](https://regex101.com/?flavor=rust) or [Rustexp](https://rustexp.lpil.uk/).

> **Tip:** Cloudflare supports two string formats for regex — quoted strings (`"..."`) requiring `\"` and `\\` escaping, and raw strings (`r#"..."#`) which avoid most escaping. The examples below use quoted strings for dashboard compatibility.

#### Rule 7: Regex — match all OpenClaw User-Agent variants in one expression

```
(http.user_agent matches "(?i)openclaw")
```

The `(?i)` flag makes the match case-insensitive inline, catching `OpenClaw-Gateway/1.0`, `openclaw`, `OPENCLAW`, and any future casing variants — without needing the `lower()` wrapper.

#### Rule 8: Regex — match User-Agent with version extraction pattern

```
(http.user_agent matches "OpenClaw-Gateway/[0-9]+\\.[0-9]+")
```

Matches the versioned Gateway UA (`OpenClaw-Gateway/1.0`, `OpenClaw-Gateway/2.1`, etc.) while ignoring unrelated strings that might coincidentally contain "openclaw".

#### Rule 9: Regex — match all OpenClaw fingerprints in a single expression

```
(http.user_agent matches "(?i)openclaw") or
(http.referer matches "^https?://openclaw\\.ai") or
(any(http.request.headers["x-title"][*] matches "(?i)^openclaw"))
```

Combines case-insensitive regex across all key fields. More precise than the `contains`-based Rule 1 because:
- `(?i)openclaw` catches any casing variant without `lower()`
- `^https?://openclaw\\.ai` anchors the referer to the domain start, avoiding false positives from URLs that merely contain the string
- `(?i)^openclaw` on `X-Title` ensures the header starts with "OpenClaw" (not just contains it)

#### Rule 10: Regex — detect stale Chrome User-Agent (WebFetch fingerprint)

```
(http.user_agent matches "Chrome/1[0-1][0-9]\\." or http.user_agent matches "Chrome/12[0-2]\\.") and
(not len(http.request.headers["sec-ch-ua"]) > 0)
```

This is a **heuristic** rule — not specific to OpenClaw, but useful for detecting its WebFetch tool. OpenClaw's default WebFetch UA hardcodes Chrome 122, which will become increasingly outdated. This rule flags requests claiming to be Chrome 100-122 while **lacking the `sec-ch-ua` header** that real Chrome always sends. The `len(...) > 0` check tests for header presence since `sec-ch-ua` is a separate HTTP header, not a substring within User-Agent. Expect false positives from other automated tools using stale Chrome strings.

**Note:** This rule will NOT match if the OpenClaw user has overridden their WebFetch UA via `tools.web.fetch.userAgent` to a current Chrome version.

### Dashboard setup

1. Navigate to **Security > WAF > Custom rules** (or **Security > Security rules** in the new dashboard)
2. Click **Create rule**
3. Enter a rule name (e.g., "Detect OpenClaw Traffic")
4. Under **When incoming requests match**, click **Edit expression** and paste the expression
5. Under **Then take action**, select the desired action
6. Click **Deploy** (or **Save as Draft** to review first)

### API example

```bash
curl "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/rulesets/$RULESET_ID/rules" \
  --request POST \
  --header "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  --header "Content-Type: application/json" \
  --data '{
    "action": "managed_challenge",
    "expression": "(lower(http.user_agent) contains \"openclaw\") or (http.referer contains \"openclaw.ai\") or (any(http.request.headers[\"x-title\"][*] contains \"OpenClaw\"))",
    "description": "Challenge OpenClaw automated requests"
  }'
```

### Important notes

- `contains` is **case-sensitive** in Cloudflare expressions — use the `lower()` wrapper for case-insensitive matching on `http.user_agent`
- The `matches` (regex) operator is **Business and Enterprise only** — Free and Pro plans must use `contains`, `eq`, or `wildcard` operators instead
- The `matches` operator uses the **Rust regex engine** — test patterns at [regex101.com (Rust flavor)](https://regex101.com/?flavor=rust) before deploying
- Custom header names in `http.request.headers["..."]` **must be lowercase** per Cloudflare docs
- The `log` action is **Enterprise-only** — use Managed Challenge as the starting action for Free/Pro/Business plans
- WAF custom rule limits vary by plan: Free (5), Pro (20), Business (100), Enterprise (1000+)
- Deploy as Managed Challenge first, review Security Events, then switch to Block if appropriate
- **WebFetch browsing traffic uses a Chrome-like User-Agent and is intentionally hard to distinguish** — Rules 1-9 will NOT catch WebFetch browsing traffic; Rule 10 provides a heuristic approach

---

## 9. Protecting an OpenClaw Gateway behind Cloudflare (inbound)

The previous sections focused on detecting OpenClaw's **outbound** requests. This section covers the reverse: using Cloudflare as a **reverse proxy in front of your OpenClaw Gateway** to protect it from unauthorized access, DDoS, and header spoofing on **inbound** requests.

### Can Cloudflare proxy an OpenClaw Gateway?

**Yes.** The OpenClaw Gateway serves HTTP and WebSocket on a single port (default 18789). Cloudflare supports both:

- **HTTP proxying** — all Gateway HTTP endpoints (`/v1/chat/completions`, `/v1/responses`, `/tools/invoke`, `/hooks/*`, Control UI) work through Cloudflare's reverse proxy
- **WebSocket proxying** — the Gateway's control-plane WebSocket upgrade on `/` is [supported by Cloudflare](https://developers.cloudflare.com/network/websockets/) without additional configuration (enable WebSockets in **Network** settings)
- **TLS termination** — Cloudflare terminates TLS at the edge; the Gateway can run plain HTTP behind it

### Gateway configuration for Cloudflare

When placing Cloudflare in front of the Gateway, configure these settings:

| Setting | Config path | Required value | Why |
|---|---|---|---|
| Bind mode | `gateway.bind` | `"lan"` or `gateway.customBindHost` | Must not be `"loopback"` — Cloudflare needs to reach the Gateway's port |
| Auth | `gateway.auth.token` or `gateway.auth.password` | Must be set | Gateway **refuses to start** on non-loopback without auth (`src/gateway/server-runtime-config.ts:97-101`) |
| Trusted proxies | `gateway.trustedProxies` | Cloudflare IP ranges | Gateway trusts `X-Forwarded-For` / `X-Real-IP` from these IPs for client IP resolution (`src/gateway/net.ts:150-164`) |

**Source:** `src/config/types.gateway.ts:243-247`
```typescript
/**
 * IPs of trusted reverse proxies (e.g. Traefik, nginx). When a connection
 * arrives from one of these IPs, the Gateway trusts `x-forwarded-for` (or
 * `x-real-ip`) to determine the client IP for local pairing and HTTP checks.
 */
trustedProxies?: string[];
```

Example config snippet:

```yaml
gateway:
  bind: "lan"
  auth:
    token: "<your-secret-token>"
  trustedProxies:
    - "173.245.48.0/20"
    - "103.21.244.0/22"
    - "103.22.200.0/22"
    - "103.31.4.0/22"
    - "141.101.64.0/18"
    - "108.162.192.0/18"
    - "190.93.240.0/20"
    - "188.114.96.0/20"
    - "197.234.240.0/22"
    - "198.41.128.0/17"
    - "162.158.0.0/15"
    - "104.16.0.0/13"
    - "104.24.0.0/14"
    - "172.64.0.0/13"
    - "131.0.72.0/22"
```

> **Tip:** Get the current Cloudflare IP ranges at [cloudflare.com/ips](https://www.cloudflare.com/ips/). These change occasionally — consider automating the update.

### Inbound `x-openclaw-*` headers

OpenClaw's HTTP API endpoints read several custom headers from inbound requests. When the Gateway sits behind Cloudflare, these headers pass through transparently and can be inspected by WAF rules.

| Header | Read at | Purpose | HTTP endpoint(s) |
|---|---|---|---|
| `x-openclaw-agent-id` | `src/gateway/http-utils.ts:27` | Agent routing — selects which named agent handles the request | `/v1/chat/completions`, `/v1/responses` |
| `x-openclaw-agent` | `src/gateway/http-utils.ts:28` | Agent routing (fallback alias for `x-openclaw-agent-id`) | Same as above |
| `x-openclaw-session-key` | `src/gateway/http-utils.ts:71` | Session pinning — pins request to a specific named session | `/v1/chat/completions`, `/v1/responses` |
| `x-openclaw-token` | `src/gateway/hooks.ts:168-169` | Webhook authentication — alternative to `Authorization: Bearer` | `/hooks/*` |
| `x-openclaw-message-channel` | `src/gateway/tools-invoke-http.ts:173` | Tool policy routing — specifies channel context (e.g., `"discord"`, `"slack"`) | `/tools/invoke` |
| `x-openclaw-account-id` | `src/gateway/tools-invoke-http.ts:175` | Account-level tool policy routing | `/tools/invoke` |
| `x-openclaw-relay-token` | `src/browser/extension-relay.ts:80` | Browser extension CDP relay auth | Separate loopback-only server (NOT on main Gateway port) |

> **Note:** `x-openclaw-relay-token` is on a separate server that binds exclusively to loopback — it is **never** accessible through Cloudflare and is listed here only for completeness.

### WAF rules for inbound Gateway protection

#### Rule 11: Block requests with spoofed `x-openclaw-*` routing headers from untrusted sources

```
(len(http.request.headers["x-openclaw-agent-id"]) > 0 or
len(http.request.headers["x-openclaw-agent"]) > 0 or
len(http.request.headers["x-openclaw-session-key"]) > 0 or
len(http.request.headers["x-openclaw-account-id"]) > 0) and
(not ip.src in {<YOUR_TRUSTED_IPS>})
```

**Action:** `block`. Prevents external attackers from injecting routing headers to target specific agents, hijack sessions, or escalate account-level tool access. Replace `<YOUR_TRUSTED_IPS>` with your known client IP ranges.

#### Rule 12: Block unauthorized webhook token injection

```
(len(http.request.headers["x-openclaw-token"]) > 0) and
(not ip.src in {<WEBHOOK_SOURCE_IPS>})
```

**Action:** `block`. Only allow the `x-openclaw-token` auth header from known webhook sources (e.g., your messaging platform's outbound IPs). This adds a network-layer check before the Gateway even sees the request.

#### Rule 13: Rate limit the API endpoints

```
(http.request.uri.path eq "/v1/chat/completions" or
http.request.uri.path eq "/v1/responses" or
http.request.uri.path eq "/tools/invoke")
```

**Action:** Use as a [Rate Limiting rule](https://developers.cloudflare.com/waf/rate-limiting-rules/) expression rather than a custom WAF rule. Recommended: 60 requests/minute per IP for the chat completions endpoint, 30/minute for tools invoke.

#### Rule 14: Require authentication header

```
(http.request.uri.path matches "^/v1/" or http.request.uri.path eq "/tools/invoke") and
(not len(http.request.headers["authorization"]) > 0)
```

**Action:** `block`. Reject requests to API endpoints that lack an `Authorization` header before they reach the Gateway. The Gateway enforces auth itself, but this reduces load from unauthenticated scanners.

#### Rule 15 (Business/Enterprise): Regex match all `x-openclaw-*` headers at once

```
(any(http.request.headers.names[*] matches "(?i)^x-openclaw-")) and
(not ip.src in {<YOUR_TRUSTED_IPS>})
```

**Action:** `block`. Uses `http.request.headers.names` (the array of all header names in the request) with regex to catch **any** current or future `x-openclaw-*` header from untrusted sources in a single rule. Requires the `matches` operator (Business/Enterprise only).

### Transform rules: strip headers from untrusted sources

Cloudflare [HTTP Request Header Modification rules](https://developers.cloudflare.com/rules/transform/request-header-modification/) (available on all plans) can **remove** headers before they reach your origin. This is useful to strip `x-openclaw-*` routing headers that external clients should never set:

**Dashboard:** Rules > Transform Rules > HTTP Request Header Modification

| Action | Header name | When |
|---|---|---|
| Remove | `x-openclaw-agent-id` | `not ip.src in {<TRUSTED_IPS>}` |
| Remove | `x-openclaw-agent` | `not ip.src in {<TRUSTED_IPS>}` |
| Remove | `x-openclaw-session-key` | `not ip.src in {<TRUSTED_IPS>}` |
| Remove | `x-openclaw-account-id` | `not ip.src in {<TRUSTED_IPS>}` |
| Remove | `x-openclaw-message-channel` | `not ip.src in {<TRUSTED_IPS>}` |

This is a defense-in-depth approach: even if an attacker bypasses the WAF block rule, the routing headers are stripped before reaching the Gateway.

### What Cloudflare does NOT protect

- **WebSocket message content** — Cloudflare proxies WebSocket frames but does not inspect their content. After the WS upgrade, all Gateway control-plane messages (including authentication, agent commands, tool invocations) pass through opaquely.
- **Browser extension relay** — The CDP relay server binds to loopback only and is not exposed through Cloudflare.
- **Gateway-to-provider traffic** — Outbound requests from the Gateway to AI providers (the subject of sections 1-8) are not routed through Cloudflare unless you also configure an outbound proxy.
- **Origin IP exposure** — If your Gateway's origin IP is discovered (DNS history, email headers, etc.), attackers can bypass Cloudflare entirely. Use firewall rules on the host to only accept connections from [Cloudflare IP ranges](https://www.cloudflare.com/ips/).
