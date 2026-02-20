> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Open PRs](./open-upstream-prs.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Hudson Rock](./hudson-rock-infostealer-analysis.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## `openclaw security audit` command reference

> **Source:** `src/cli/security-cli.ts:45-51`, `src/security/audit.ts:657-742`, `src/security/fix.ts:387-473`, `src/security/audit-extra.sync.ts`, `src/security/audit-extra.async.ts`, `src/security/audit-channel.ts`
>
> The built-in security audit scans your local config, filesystem permissions, and channel policies for common misconfigurations. It does **not** scan source code for vulnerabilities.

### Command modes

| Command | Behavior |
| --------- | ---------- |
| `openclaw security audit` | Read-only scan: 16 collector functions check config, filesystem, channels, models, plugins, hooks, gateway, browser. No network calls. |
| `openclaw security audit --deep` | Everything above + static code-safety scan of installed plugins and skills (`collectPluginsCodeSafetyFindings`, `collectInstalledSkillsCodeSafetyFindings`); also `maybeProbeGateway()` connects to gateway WebSocket (5 s timeout), verifies auth, adds `gateway.probe_failed` if unreachable. |
| `openclaw security audit --fix` | Runs `fixSecurityFootguns()` first, then full audit. Report reflects post-fix state. Also accepts `--deep`. |
| `openclaw security audit --json` | Any mode above with JSON output instead of formatted text. |

### Check categories (70+ check IDs)

| # | Category | Check ID prefix | Severities | What it checks |
| --- | ---------- | ---------------- | ------------ | --------------- |
| 1 | Attack surface summary | `summary.attack_surface` | info | Counts open groups, elevated tools, hooks (reports `hooks.webhooks` and `hooks.internal` as separate lines), browser control |
| 2 | Synced folders | `fs.synced_dir` | warn | State/config in iCloud, Dropbox, OneDrive, Google Drive |
| 3 | Filesystem permissions | `fs.state_dir.*`, `fs.config.*`, `fs.credentials_dir.*`, `fs.auth_profiles.*`, `fs.sessions_store.*`, `fs.log_file.*` | critical/warn | World/group-writable dirs, world/group-readable config/credentials, symlink detection |
| 4 | Config include files | `fs.config_include.*` | critical/warn | Permissions on included config files |
| 5 | Gateway configuration | `gateway.bind_no_auth`, `gateway.loopback_no_auth`, `gateway.tailscale_funnel`, `gateway.tailscale_serve`, `gateway.control_ui.*`, `gateway.token_too_short`, `gateway.trusted_proxies_missing`, `gateway.trusted_proxy_auth`, `gateway.trusted_proxy_no_proxies`, `gateway.trusted_proxy_no_user_header`, `gateway.trusted_proxy_no_allowlist`, `gateway.auth_no_rate_limit`, `gateway.http.no_auth`, `gateway.http.session_key_override_enabled`, `gateway.tools_invoke_http.dangerous_allow` | critical/warn/info | Non-loopback bind without auth, Tailscale Funnel public exposure, Control UI insecure auth/device auth, token length, missing trusted proxy, trusted-proxy mode hardening checks (missing proxy IPs/user-header/allowlist), auth brute-force protection (`gateway.auth_no_rate_limit`), and HTTP endpoint exposure: `gateway.http.no_auth`=warn/critical when `auth.mode="none"` (lists exposed HTTP endpoints; critical if remotely exposed); `gateway.http.session_key_override_enabled`=info; `gateway.tools_invoke_http.dangerous_allow`=warn/critical when `gateway.tools.allow` re-enables default-denied HTTP tools (sessions_spawn, sessions_send, gateway, whatsapp_login) |
| 5a | Gateway nodes | `gateway.nodes.deny_commands_ineffective` | warn | `denyCommands` has pattern-like or unknown entries that won't match (exact command-name matching only) |
| 6 | Browser control | `browser.remote_cdp_http`, `browser.control_invalid_config`, `browser.control_no_auth` | critical/warn | Remote CDP over plain HTTP, invalid CDP config, and browser control server with auth disabled |
| 7 | Logging | `logging.redact_off` | warn | `redactSensitive="off"` leaks secrets in tool summaries |
| 8 | Elevated tools | `tools.elevated.allowFrom.*.wildcard`, `tools.elevated.allowFrom.*.large` | critical/warn | Wildcard `"*"` in elevated allowlist, oversized allowlist (>25 entries) |
| 8a | Exec runtime / sandboxing | `tools.exec.host_sandbox_no_sandbox_defaults`, `tools.exec.host_sandbox_no_sandbox_agents` | warn | Exec host sandbox not applied to default or agent profiles |
| 8b | Sandbox config | `sandbox.docker_config_mode_off`, `sandbox.dangerous_bind_mount`, `sandbox.dangerous_network_mode`, `sandbox.dangerous_seccomp_profile`, `sandbox.dangerous_apparmor_profile`, `sandbox.bind_mount_non_absolute` | critical/warn | Docker config present but `sandbox.mode=off`; dangerous bind mounts, network modes, seccomp/apparmor profiles (critical); non-absolute bind mount path (warn) |
| 8c | Tools profile | `tools.profile_minimal_overridden` | warn | Global `tools.profile=minimal` overridden by an agent-level profile |
| 9 | Hooks hardening | `hooks.path_root`, `hooks.token_too_short`, `hooks.token_reuse_gateway_token`, `hooks.default_session_key_unset`, `hooks.request_session_key_enabled`, `hooks.request_session_key_prefixes_missing` | critical/warn | Hooks base path is `"/"`, short token (<24 chars), token reuses gateway token (**critical** as of Feb 19 2026 sync); session key unset (warn); request session key enabled or prefixes missing (critical if remotely exposed, warn otherwise) |
| 10 | Model hygiene | `models.legacy`, `models.weak_tier`, `models.small_params` | critical/warn | Legacy models (GPT-3.5, Claude 2), weak tier (Haiku, pre-GPT-5), small models (<=300B params) without sandboxing exposed to web tools |
| 11 | Config secrets | `config.secrets.gateway_password_in_config`, `config.secrets.hooks_token_in_config` | warn/info | Secrets stored in config file instead of env vars |
| 12 | Plugins/extensions | `plugins.extensions_no_allowlist`, `plugins.tools_reachable_permissive_policy`, `plugins.installs_unpinned_npm_specs`, `plugins.installs_missing_integrity`, `plugins.installs_version_drift`, `hooks.installs_unpinned_npm_specs`, `hooks.installs_missing_integrity`, `hooks.installs_version_drift` | critical/warn | Extensions present but `plugins.allow` not configured; permissive tool reachability policy; unpinned/missing-integrity/version-drifted npm specs in plugin or hook installs |
| 13 | Channel security | `channels.discord.*`, `channels.slack.*`, `channels.telegram.*`, `channels.*.dm.*` | critical/warn/info | DM policies (open/disabled/scoped), group policies, slash command restrictions, sender allowlists, multi-user DM session isolation |
| 14 | Exposure matrix | `security.exposure.open_groups_with_elevated` | critical | Dangerous combination: open `groupPolicy` + elevated tools enabled |
| — | Deep probe | `gateway.probe_failed` | warn | `--deep` only: gateway WebSocket unreachable or auth failed |
| — | Plugin/skill code safety | `plugins.code_safety.*`, `skills.code_safety.*` | critical/warn | `--deep` only: static pattern scan of installed plugin/skill JS/TS/JSON for dangerous patterns; `entry_escape`=critical, `entry_path`/`scan_failed`=warn |

### What `--fix` applies (`src/security/fix.ts:387-473`)

**Config changes** (`applyConfigFixes`, line 276):

- `logging.redactSensitive`: `"off"` → `"tools"` (prevents secrets in tool summaries)
- `groupPolicy`: `"open"` → `"allowlist"` for all 7 supported channels (telegram, whatsapp, discord, signal, imessage, slack, msteams), including per-account overrides
- WhatsApp `groupAllowFrom`: populated from pairing store when policy flipped (so existing paired contacts still work)

**Filesystem hardening** (chmod/icacls):

| Target | Mode | Purpose |
| -------- | ------ | --------- |
| `~/.openclaw/` (state dir) | `700` | User-only access to all state |
| Config file | `600` | User-only read/write (contains tokens) |
| Config include files | `600` | Same protection for split configs |
| `~/.openclaw/credentials/` | `700` | OAuth credential directory |
| `credentials/*.json` | `600` | Individual credential files |
| `agents/<id>/agent/` | `700` | Per-agent directory |
| `agents/<id>/agent/auth-profiles.json` | `600` | API keys and tokens |
| `agents/<id>/sessions/` | `700` | Session transcript directory |
| `agents/<id>/sessions/sessions.json` | `600` | Session metadata |
| `agents/<id>/sessions/*.jsonl` | `600` | Session transcript files |

On Windows: uses `icacls` ACL reset instead of `chmod`.

`--fix` skips symlinks, missing paths, and already-correct permissions (safe + idempotent).

### Coverage vs documented security issues

The audit is a **configuration and filesystem hardening tool**. It detects misconfigurations but not code-level vulnerabilities.

#### Issues the audit detects or mitigates

| Issue | Severity | What the audit catches | Check ID |
| ------- | ---------- | ---------------------- | ---------- |
| [#9065](https://github.com/openclaw/openclaw/issues/9065) | LOW | `~/.openclaw` group-writable after sudo install | `fs.state_dir.perms_group_writable` (`--fix` applies chmod 700) |
| [#7862](https://github.com/openclaw/openclaw/issues/7862) | MEDIUM | Session transcripts 644 instead of 600 | `fs.sessions_store.perms_readable` (`--fix` applies chmod 600) |
| [#9627](https://github.com/openclaw/openclaw/issues/9627) | HIGH | Config secrets exposed in JSON | `config.secrets.gateway_password_in_config` (warns; recommends env var) |
| [#6609](https://github.com/openclaw/openclaw/issues/6609) | HIGH | Browser bridge server optional auth | `browser.control_invalid_config`, `browser.control_no_auth` |
| General gateway exposure | — | Non-loopback bind, missing auth, Tailscale Funnel | `gateway.bind_no_auth`, `gateway.loopback_no_auth`, `gateway.tailscale_funnel` |
| Channel misconfiguration | — | Open DMs, open groups, missing allowlists | All `channels.*` checks + `security.exposure.open_groups_with_elevated` |

**Note:** The audit does not scan workspace bootstrap `.md` files (MEMORY.md, SOUL.md, AGENTS.md, TOOLS.md, IDENTITY.md, USER.md, HEARTBEAT.md, BOOTSTRAP.md, memory.md) or `memory/*.md` for prompt injection content. The bootstrap files are injected into the system prompt as trusted context with no content validation. Memory directory files (`memory/*.md`) go through a separate tool-call pipeline but are also unscanned. See [Cisco AI Defense gap analysis](./cisco-ai-defense-skill-scanner.md#beyond-skillmd-all-persistent-md-files-are-unscanned).

#### Issues the audit cannot detect (code-level bugs)

| Issue | Severity | Why the audit cannot detect it |
| ------- | ---------- | ------------------------------- |
| [#8512](https://github.com/openclaw/openclaw/issues/8512) | CRITICAL | Plugin HTTP routes bypass — code-level auth gap in `src/gateway/server/plugins-http.ts` |
| [#3277](https://github.com/openclaw/openclaw/issues/3277) | HIGH | Path traversal via `startsWith` — code-level validation bug |
| [#4950](https://github.com/openclaw/openclaw/issues/4950) | HIGH | Browser evaluate default on — hardcoded constant, not configurable |
| [#5052](https://github.com/openclaw/openclaw/issues/5052) | HIGH | Config validation fail-open — code-level bug in `src/config/io.ts` |
| [#5255](https://github.com/openclaw/openclaw/issues/5255) | HIGH | Browser file upload arbitrary read — code-level |
| [#8516](https://github.com/openclaw/openclaw/issues/8516) | HIGH | Browser download/trace arbitrary file write — code-level |
| [#8586](https://github.com/openclaw/openclaw/issues/8586) | HIGH | Configurable exec bypass — code-level allowlist gap |
| [#8590](https://github.com/openclaw/openclaw/issues/8590) | HIGH | Status endpoint info leak — code-level |
| [#8591](https://github.com/openclaw/openclaw/issues/8591) | HIGH | Env vars exposed via shell — code-level |
| [#8776](https://github.com/openclaw/openclaw/issues/8776) | HIGH | soul-evil hook hijacking — code-level |
| [#9435](https://github.com/openclaw/openclaw/issues/9435) | ~~HIGH~~ FIXED | Gateway token in URL query params — fixed in PR #9436 |
| [#9512](https://github.com/openclaw/openclaw/issues/9512) | HIGH | Skill archive path traversal (Zip Slip) — code-level |
| [#9517](https://github.com/openclaw/openclaw/issues/9517) | HIGH | Canvas host auth bypass — code-level |
| [#8696](https://github.com/openclaw/openclaw/issues/8696) | HIGH | Playwright download path traversal — code-level |
| [#4949](https://github.com/openclaw/openclaw/issues/4949) | HIGH | Browser DNS rebinding — code-level (no Host header validation) |
| [#4995](https://github.com/openclaw/openclaw/issues/4995) | HIGH | iMessage DM auto-responds with pairing codes — code-level |
| [#5995](https://github.com/openclaw/openclaw/issues/5995) | HIGH | Secrets in session transcripts — by design |
| [#6606](https://github.com/openclaw/openclaw/issues/6606) | HIGH | Telegram webhook binds 0.0.0.0 — code-level |
| [#8054](https://github.com/openclaw/openclaw/issues/8054) | HIGH | Type coercion "undefined" credentials — code-level |

#### Verdict

> **The audit catches ~6 of 37 documented issues** (config/filesystem misconfigurations). The expanded check set (70+ check IDs) adds sandbox config, exec runtime sandboxing, gateway HTTP auth gaps, plugin/hook supply-chain checks, and `--deep` plugin/skill code-safety scanning. The remaining ~31 issues are code-level vulnerabilities that require upstream patches. For defense-in-depth: run `openclaw security audit --fix` **and** monitor the [open upstream security issues](./open-upstream-issues.md) list.
