> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Open PRs](./open-upstream-prs.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## SecurityScorecard STRIKE Report Analysis (Feb 2026)

> **Source:** [Beyond the Hype: Moltbot's Real Risk Is Exposed Infrastructure, Not AI Superintelligence](https://securityscorecard.com/blog/beyond-the-hype-moltbots-real-risk-is-exposed-infrastructure-not-ai-superintelligence/) — SecurityScorecard STRIKE Team, published 9 February 2026
>
> **Live dashboard:** [declawed.io](https://declawed.io) (updated every 15 minutes)
>
> **Audited against:** OpenClaw v2026.2.6-3 (`package.json:2-3`)

---

### What the Report Found

The SecurityScorecard STRIKE team performed internet-wide scanning to identify publicly exposed OpenClaw (formerly Moltbot/Clawdbot) instances.

**Key statistics:**

| Metric | Value |
|--------|-------|
| Unique IPs with exposed control panels | 28,663 |
| Total exposed instances | 40,214 |
| Countries | 76 |
| Instances identified as "vulnerable to RCE" | 12,812 |
| Instances correlating with prior breach activity | 549 |

**Version fragmentation** reveals that the vast majority of exposed instances are running outdated, pre-rename versions:

| Version era | Share | Implication |
|-------------|-------|-------------|
| **Clawdbot** Control | 39.5% | Oldest — predates Moltbot rename |
| **Moltbot** Control | 38.5% | Pre-OpenClaw rename |
| **OpenClaw** Control | 22.0% | Current branding (may still be outdated builds) |

This means **78% of discovered instances are running pre-rename versions** that likely lack the security hardening commits from January 2026.

**Scanning methodology:**
- Favicon hash fingerprinting (3 MD5 hashes across the project's branding eras)
- SSL certificate CN analysis (284 domain patterns)
- Reverse DNS and certificate transparency logs (416 associated domains)
- SecurityScorecard proprietary breach/threat actor correlation

**Favicon hash verification** — 2 of 3 hashes confirmed against current source (`ui/public/`):

| Report hash | File | Verified? |
|-------------|------|-----------|
| `f58854f6450618729679ad33622bebaf` | `favicon.ico` | **Yes** — matches current codebase |
| `75661cbf8fd50009be0c46a9ca8b3180` | `favicon.svg` (OpenClaw) | **Yes** — matches current codebase |
| `087361307be2b2789849d4fd58170cca` | `favicon.svg` (Moltbot) | Predates current repo history — from Moltbot branding era |

**Geographic/infrastructure distribution:** 45% hosted on Alibaba Cloud, 37% located in China, 24.5% on DigitalOcean.

---

### CVEs Cited (All Patched)

The report cites 3 of the 5 official CVEs/GHSAs. All were patched before publication.

| CVE / GHSA | Severity | Summary | Patched in | Cited in report? |
|------------|----------|---------|------------|-----------------|
| CVE-2026-24763 | HIGH | Docker PATH command injection | v2026.1.29 | Yes |
| GHSA-g8p2-7wf7-98mq (CVE-2026-25253) | HIGH | 1-click RCE via gatewayUrl token exfiltration | v2026.1.29 | Yes |
| GHSA-q284-4pvr-m585 (CVE-2026-25157) | HIGH | OS command injection via sshNodeCommand | v2026.1.29 | Yes |
| CVE-2026-25593 | HIGH | Unauthenticated local RCE via WebSocket config.apply | v2026.1.20 | No |
| CVE-2026-25475 | MEDIUM | Local file inclusion via MEDIA: path extraction | v2026.1.30 | No |

**Current version:** v2026.2.6-3. All CVEs have been patched for weeks.

Full details: [Official Security Advisories](./official-security-advisories.md)

---

### Additional Vulnerability Claims (declawed.io FAQ)

The STRIKE team's live dashboard at declawed.io lists 8 vulnerabilities in its FAQ. Three are the CVEs above. The remaining 5 were cross-referenced against the current codebase:

| declawed.io Claim | Code-verified Status | Why |
|-------------------|---------------------|-----|
| **Token leakage via browser history** | NOT VULNERABLE | Tokens are passed via `--token` CLI flag or `OPENCLAW_GATEWAY_TOKEN` env var, not URL query params. The `openclaw dashboard` command prints a tokenized URL for local convenience; the token is in the fragment, not persisted in server logs. |
| **API key in client JS** | NOT VULNERABLE | API keys are read from env vars server-side (`process.env.OPENAI_API_KEY`). The UI fetches config from the gateway API; no keys are bundled into client JavaScript. |
| **SSRF via webhook bypass** | NOT EXPLOITABLE | `extensions/voice-call/src/webhook-security.ts:456,480` has a `skipVerification` parameter, but it is explicitly a dev-only feature. The production path enforces Twilio signature verification with `crypto.timingSafeEqual()`. Previously audited in [Issue #1796](./issue-1796-argus-audit.md). |
| **Symlink privilege escalation** | NOT VULNERABLE | `src/infra/fs-safe.ts:57-58` opens files with `O_NOFOLLOW` flag. Lines 74-76 check `isSymbolicLink()`. Lines 89-90 verify inode/device match. Added in post-merge hardening (commit `5eee991`). |
| **Unauthenticated localhost admin** | DESIGN CHOICE | Loopback binding (`127.0.0.1`) intentionally skips auth because the threat model trusts the local machine. Any non-loopback binding requires auth (`run.ts:250-261`). The SSRF escalation scenario requires a separate SSRF vulnerability on the same host, which is not an OpenClaw bug. |

**Score: 0/5 exploitable on current codebase.**

---

### What the Report Gets Right

1. **Real CVEs exist.** Five official CVEs/GHSAs were disclosed and patched. The report cites 3 with public exploit code.

2. **Docker-compose defaults to LAN binding.** This is true and is a reasonable concern for users who deploy via Docker without reviewing the compose file.

3. **Misconfiguration is a real risk.** VPS deployments with public binding and outdated versions are genuinely dangerous.

4. **The favicon-hash methodology is sound.** Scanning for known favicon hashes reliably identifies exposed instances. The version fragmentation data is useful for understanding update adoption.

5. **The hardening recommendations are good.** The report's advice (bind localhost, use Tailscale/VPN, rotate tokens, run security audit) aligns with our existing documentation.

6. **The title framing is honest.** "Beyond the Hype: Moltbot's Real Risk Is Exposed Infrastructure, Not AI Superintelligence" correctly identifies the issue as infrastructure misconfiguration, not AI-specific vulnerability.

---

### What the Report Omits

The report describes real risks but omits critical security controls that exist in the current codebase:

| Omitted control | What it does | Code reference |
|-----------------|-------------|----------------|
| **Loopback default** | Native CLI binds to `127.0.0.1` by default, not `0.0.0.0` | `run.ts:181` — `const bindRaw = toOptionString(opts.bind) ?? cfg.gateway?.bind ?? "loopback"` |
| **Auth enforcement at startup** | Gateway refuses to start on non-loopback without a configured token or password | `run.ts:250-261` — `if (bind !== "loopback" && !hasSharedSecret) { ... exit(1) }` |
| **Exec approval system** | Tool execution requires human confirmation by default; an exposed Gateway does not mean automatic command execution | Default tool policies require approval |
| **Env var blocklist** | 16 dangerous env vars blocked (`LD_PRELOAD`, `DYLD_INSERT_LIBRARIES`, `NODE_OPTIONS`, `BASH_ENV`, etc.) plus prefix-based blocking | `bash-tools.exec-runtime.ts:34-52` |
| **Docker sandbox isolation** | Containerized deployments have filesystem and process isolation limiting blast radius | Docker-compose default configuration |
| **Two prior independent audits** | Both found 0 exploitable claims: [Issue #1796 (Argus)](./issue-1796-argus-audit.md) — 0/8, [Medium article](./medium-article-audit.md) — 0/8 | See linked analyses |

The report's claim that OpenClaw "binds to `0.0.0.0` out of the box" conflates Docker-compose defaults (which require `0.0.0.0` for container networking) with the native CLI default (which is loopback). The auth enforcement was also strengthened in commit `c4a80f4ed` (Jan 26, 2026), tightening the non-loopback auth check from `resolvedAuthMode === "none"` to `!hasSharedSecret`.

**Important nuance:** The report's version data shows 78% of instances running pre-rename versions. These older versions predate the January 2026 hardening and may have been more permissive. The report's characterizations may be more accurate for those older versions than for the current codebase.

---

### Why This Matters: Update Your Instances

This is the most important section of this analysis.

The SecurityScorecard report, despite its omissions, demonstrates a critical point: **version fragmentation in the OpenClaw ecosystem is severe, and most exposed instances are running dangerously outdated software.**

**The numbers tell the story:**
- 78% of discovered instances are running pre-rename versions (Clawdbot or Moltbot era)
- These versions predate the auth hardening commit of January 26, 2026 (`c4a80f4ed`)
- These versions predate all 5 CVE patches (January 20-30, 2026)
- Even the 22% running "OpenClaw" branding may be outdated builds

**All 5 CVEs are patched — but only if you update.** The patches do nothing for instances stuck on old versions.

**What you should do right now:**

- [ ] **Update to the latest version:**
  ```bash
  npm install -g openclaw@latest
  openclaw gateway restart
  ```
- [ ] **Verify your binding mode** — should be `loopback` unless you have a specific reason:
  ```bash
  openclaw config get gateway.bind
  # Should output: loopback
  ```
- [ ] **Run the security audit:**
  ```bash
  openclaw security audit --fix
  ```
- [ ] **If using Docker**, pull the latest image and rebuild:
  ```bash
  docker compose pull && docker compose up -d
  ```
- [ ] **Check if your instance is publicly exposed:**
  ```bash
  # Check your public IP on Shodan
  curl -s https://api.shodan.io/shodan/host/$(curl -s ifconfig.me)?key=YOUR_KEY
  ```
- [ ] **Enable auth if binding to non-loopback** — the current version enforces this at startup, but older versions may not:
  ```bash
  openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
  ```
- [ ] **Use SSH tunnels or Tailscale** for remote access instead of public binding:
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
  ```

---

### Audit Scorecard

| Category | Claims | Verdict |
|----------|--------|---------|
| Project naming / history | 1 | ACCURATE |
| Control panel exists | 1 | ACCURATE |
| CVEs with RCE potential | 3 | ACCURATE (all patched before publication) |
| Messaging integrations | 1 | ACCURATE |
| Binds to all interfaces "out of the box" | 1 | MISLEADING (native CLI defaults to loopback; Docker-compose defaults to LAN for container networking) |
| "Weak authentication" | 1 | MISLEADING (auth enforced at startup for non-loopback) |
| Agent permissions inheritance | 1 | PARTIALLY ACCURATE (bounded by policy, not unbounded) |
| declawed.io additional vulnerability claims | 5 | 0/5 exploitable on current codebase |
| Statistical claims (28k IPs, 63% exploitable, breach correlation) | 3 | UNVERIFIABLE (no telemetry in codebase; plausible methodology) |

**Running total across all external audits:**

| Audit | Claims tested | Exploitable |
|-------|--------------|-------------|
| [Issue #1796 (Argus)](./issue-1796-argus-audit.md) | 8 | 0 |
| [Medium article (Saad Khalid)](./medium-article-audit.md) | 8 | 0 |
| [SecurityScorecard STRIKE + declawed.io](./securityscorecard-strike-report.md) | 5 | 0 |
| [ZeroLeeks AI Red Team](./zeroleeks-audit.md) | 34 | 0 |
| [Cisco AI Defense blog post](./cisco-ai-defense-skill-scanner.md) | 4 | 0 |
| **Total** | **59** | **0** |

**0/59 third-party vulnerability claims confirmed exploitable on the current codebase.**
