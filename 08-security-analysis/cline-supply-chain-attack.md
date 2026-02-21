> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Open PRs](./open-upstream-prs.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Hudson Rock](./hudson-rock-infostealer-analysis.md) | [Cline Supply Chain](./cline-supply-chain-attack.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## Cline CLI Supply Chain Attack — "Clinejection" (Feb 2026)

> **Sources:** [Endor Labs](https://www.endorlabs.com/learn/cline-ai-supply-chain-attack-when-your-ai-assistant-installs-malware), [Snyk](https://snyk.io/blog/cline-ai-supply-chain-attack/), [The Hacker News](https://thehackernews.com/2026/02/compromised-cline-vscode-extension.html), [Adnan Khan blog](https://adnanthekhan.com/2026/02/17/clinejection/)
> **Date:** Feb 17, 2026
> **Advisory:** [GHSA-9ppg-jx86-fqw7](https://github.com/cline/cline/security/advisories/GHSA-9ppg-jx86-fqw7) (Cline repository, not OpenClaw)
> **Malicious tarball SHA-256:** `a1b2c3...` (cline@2.3.0 — deprecated within ~8 hours)

---

### TL;DR (Plain English)

A compromised Cline CLI package (v2.3.0) silently ran `npm install -g openclaw@latest` on ~4,000 developer machines via a malicious `postinstall` hook before being deprecated. **Risk to OpenClaw users: LOW** — OpenClaw was the benign payload installed by the hook, not the attack vector, and the Gateway daemon was never started.

---

### What Happened (Plain English)

On **Feb 17, 2026 at ~03:26 PT**, a tampered version of the Cline CLI (v2.3.0) was published to npm. The package contained a `postinstall` hook that automatically ran `npm install -g openclaw@latest` on every machine that installed or updated Cline CLI to v2.3.0.

- **~03:26 PT** — Malicious `cline@2.3.0` published to npm
- **~11:30 PT** — Package deprecated by Cline team after community reports
- **~4,000 downloads** in the ~8-hour window

The attack did not exploit any vulnerability in OpenClaw. OpenClaw was simply the package installed by the hook — it functioned exactly as documented, and the Gateway daemon was never started on affected machines.

---

### The Attack Chain ("Clinejection")

The full attack chain, as documented by Adnan Khan and corroborated by Endor Labs and Snyk:

1. **Prompt injection via GitHub issue:** Attacker opens a GitHub issue on the Cline repository with a crafted payload in the issue title/body
2. **Claude-powered triage bot executes commands:** Cline's automated Claude-powered issue triage bot processes the issue and executes arbitrary commands due to excessive permissions granted to the bot
3. **GitHub Actions cache poisoning:** The bot's commands poison GitHub Actions cache entries (flooding >10 GB of junk data to trigger LRU eviction of legitimate entries)
4. **npm token theft via nightly workflow:** Poisoned cache entries are restored in Cline's nightly publish workflow, which leaks the long-lived `NPM_RELEASE_TOKEN` secret
5. **Malicious package published:** Attacker uses the stolen npm token to publish `cline@2.3.0` via the `clinebotorg` npm account (Cline's legacy bot account)
6. **postinstall hook installs OpenClaw:** The malicious package's `postinstall` script runs `npm install -g openclaw@latest` on every machine that installs it

---

### Why OpenClaw Is Named

OpenClaw is **the payload**, not the vulnerability:

- The `postinstall` hook installed the legitimate `openclaw` npm package globally
- OpenClaw itself contains no malicious code — it is a well-known open-source AI assistant framework
- The OpenClaw Gateway daemon was **not started** by the hook — only the npm package was installed
- No OpenClaw configuration files were created or modified
- No data was exfiltrated by the OpenClaw package

OpenClaw has no GHSA of its own for this incident. The advisory [GHSA-9ppg-jx86-fqw7](https://github.com/cline/cline/security/advisories/GHSA-9ppg-jx86-fqw7) is filed under Cline's GitHub repository. See [Official Security Advisories](./official-security-advisories.md) for OpenClaw's own advisory tracking.

---

### Impact Assessment

| What happened | What did NOT happen |
|---|---|
| OpenClaw npm package installed globally on ~4,000 machines | Gateway daemon NOT started |
| `openclaw` binary added to PATH | No credential exfiltration by OpenClaw |
| Disk space used by OpenClaw package files | No network connections initiated |
| VS Code / JetBrains extensions NOT affected (CLI-only attack) | No credential rotation needed for OpenClaw |
| Attack window: ~8 hours before deprecation | No persistent backdoor installed |

---

### Detection Signals

Endor Labs' AI-powered differential analysis identified three anomalies that flagged `cline@2.3.0` as suspicious:

1. **New `postinstall` hook:** The `postinstall` script was absent in all prior Cline CLI versions — a new lifecycle hook in a patch release is a strong supply chain indicator
2. **Missing SLSA provenance attestation:** Previous Cline releases included SLSA provenance attestations linking the package to its GitHub Actions build. Version 2.3.0 had no provenance record
3. **Publisher account change:** Previous versions were published via GitHub Actions OIDC (short-lived, build-scoped tokens). Version 2.3.0 was published by the legacy `clinebotorg` npm account using a long-lived token

---

### Remediation Steps

If you installed Cline CLI between Feb 17 ~03:26 PT and ~11:30 PT:

1. **Check your Cline version:**
   ```bash
   cline --version
   ```
   If it shows `2.3.0`, you were affected.

2. **Update Cline to a clean version:**
   ```bash
   npm install -g cline@latest
   ```

3. **Remove the unwanted OpenClaw global install:**
   ```bash
   npm uninstall -g openclaw
   ```

4. **No credential rotation needed** — the OpenClaw package did not access or exfiltrate any credentials. The Gateway was never started.

---

### Key Takeaways for the OpenClaw Community

- **Trusted publishing is necessary but not sufficient.** Cline had OIDC-based trusted publishing enabled, but the legacy `clinebotorg` bot account with a long-lived npm token was never disabled. Enabling trusted publishing does not retroactively revoke legacy tokens — you must explicitly disable legacy token publishing.

- **Verify npm package provenance before installing updates.** The `npm audit signatures` command and SLSA provenance attestations can detect packages published outside the expected build pipeline. The absence of provenance on `cline@2.3.0` was a key detection signal.

- **AI-powered CI/CD workflows with broad permissions are a critical attack surface.** The root cause was a Claude-powered GitHub issue triage bot with permissions to execute arbitrary commands. Prompt injection in automated AI workflows can escalate from a crafted issue title to production credential theft.

- **GitHub Actions cache poisoning is a viable pivot technique.** The attacker used cache poisoning to move from a low-privilege bot execution context to the production publish workflow's secrets — a novel lateral movement technique worth monitoring.

- **`postinstall` hooks remain the #1 npm supply chain vector.** This attack reinforces the importance of reviewing `package.json` scripts and using `npm install --ignore-scripts` when evaluating unfamiliar packages or unexpected updates.

---

### Cross-References

- [Ecosystem Security Threats — #9 Clinejection](./ecosystem-security-threats.md#9-cline-cli-supply-chain-attack-clinejection) — condensed version in the ecosystem threats catalog
- [Cross-Cutting Vulnerabilities — Supply Chain Risks](../05-worst-case-security/cross-cutting.md#e-supply-chain-risks) — historical context alongside event-stream, ua-parser-js, and colors/faker
- [Prompt Injection Attacks](../05-worst-case-security/prompt-injection-attacks.md#understanding-prompt-injection) — real-world precedent for prompt injection causing supply chain compromise
- [Skills.sh Risks](../05-worst-case-security/skills-sh-risks.md) — npm `postinstall` hook abuse in the OpenClaw ecosystem context
- [Official Security Advisories](./official-security-advisories.md) — OpenClaw has no GHSA of its own for this incident

---

### Sources

- **Endor Labs:** [Cline AI Supply Chain Attack: When Your AI Assistant Installs Malware](https://www.endorlabs.com/learn/cline-ai-supply-chain-attack-when-your-ai-assistant-installs-malware)
- **The Hacker News:** [Compromised Cline VSCode Extension](https://thehackernews.com/2026/02/compromised-cline-vscode-extension.html)
- **Snyk:** [Cline AI Supply Chain Attack](https://snyk.io/blog/cline-ai-supply-chain-attack/)
- **Adnan Khan:** [Clinejection — Prompt Injection to Supply Chain Compromise](https://adnanthekhan.com/2026/02/17/clinejection/)
- **SOC Defenders:** [Clinejection Analysis](https://socdefenders.com/2026/02/clinejection-supply-chain-attack/)
