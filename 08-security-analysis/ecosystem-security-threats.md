> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## Ecosystem Security Threats

> **These are not codebase vulnerabilities.** The threats below target OpenClaw users through supply chain attacks, social engineering, and infrastructure misconfiguration. They apply to any popular self-hosted AI framework, not just OpenClaw.
>
> This section was prompted by a community security advisory from [@edgeaiplanet on Threads](https://www.threads.com/@edgeaiplanet/post/DUNyYMtjnw3).

### Official packages and accounts

Before installing or following any link, verify you are using official sources:

| Platform | Official Source | Impostor Red Flags |
|----------|----------------|-------------------|
| npm | `openclaw` | Typos (`opennclaw`, `open-claw`), extra characters, different org |
| GitHub | `openclaw/openclaw` | Forked repos, similar usernames (`openclaw-bot`, `openclawai`) |
| X/Twitter | Verify on [docs.openclaw.ai](https://docs.openclaw.ai) | Old handles, recently created accounts |
| Docs | `docs.openclaw.ai` | Lookalike domains (`docs-openclaw.ai`, `openclaw-docs.com`) |

### 1. npm Typosquatting Honeypots

**What it is:** Attackers publish malicious packages with names similar to legitimate ones, hoping developers will mistype or copy the wrong name.

**How it works:**
- Package named `opennclaw` or `openclaw-bot` contains malicious `postinstall` script
- Script runs automatically during `npm install`
- Exfiltrates `.env` files, API keys, SSH keys, or cryptocurrency wallets

**Real-world examples:**
- 3,180+ malicious npm packages detected in 2025 alone ([The Hacker News](https://thehackernews.com/2025/10/10-npm-packages-caught-stealing.html))
- "lotusbail" package (56,000 downloads) stole WhatsApp credentials ([CSO Online](https://www.csoonline.com/article/4117139/from-typos-to-takeovers-inside-the-industrialization-of-npm-supply-chain-attacks.html))

**Mitigations:**
- Always verify the exact package name: `npm view openclaw`
- Check package metadata: author, repository link, download count, publish history
- Use `npm install --ignore-scripts` when evaluating unfamiliar packages
- Review `package.json` scripts before running `npm install` on cloned repos

### 2. The Rebrand Trap (Handle Sniping)

**What it is:** When projects rename (Clawdbot to Moltbot to OpenClaw), attackers register the abandoned usernames to impersonate official accounts.

**How it works:**
- Attacker monitors popular projects for rebrand announcements
- Registers old handles within seconds of abandonment
- Posts "official" announcements with phishing links or malicious downloads

**Real-world examples:**
- Scammers sniped the old Clawdbot handle within seconds of the rebrand ([DEV.to](https://dev.to/sivarampg/from-clawdbot-to-moltbot-how-a-cd-crypto-scammers-and-10-seconds-of-chaos-took-down-the-4eck))
- GitHub discussion on username squatting after renames ([GitHub Community](https://github.com/orgs/community/discussions/83803))

**Mitigations:**
- Only use links from [docs.openclaw.ai](https://docs.openclaw.ai) or the official GitHub repo
- Be suspicious of announcements on old handles
- Check account creation dates and posting history
- Verify announcements through multiple official channels

### 3. Session Token Stealing

**What it is:** Malware or malicious plugins steal session files that bypass 2FA, allowing attackers to impersonate you on messaging platforms.

**How it works:**
- Infostealers search for Telegram `tdata/` directories and WhatsApp session files
- Malicious "plugins" request permissions to access session storage
- Stolen sessions let attackers read messages and pair new devices without your credentials

**Real-world examples:**
- PupkinStealer and Raven Stealer target Telegram session files ([Kaspersky](https://www.kaspersky.com/blog/how-to-prevent-whatsapp-telegram-account-hijacking-and-quishing/53012/))
- Malicious npm package stole WhatsApp messages via session hijacking ([The Register](https://www.theregister.com/2025/12/22/whatsapp_npm_package_message_steal/))

**Mitigations:**
- Only install plugins from official OpenClaw sources
- Regularly check linked devices in WhatsApp (Settings > Linked Devices) and Telegram (Settings > Devices)
- Use `openclaw security audit --deep` to check for suspicious access
- Keep session directories (`~/.openclaw/sessions/`) with restrictive permissions
- Enable Telegram's 2FA password (separate from SMS code)
- Keep session directories (`~/.openclaw/sessions/`) with restrictive permissions

### 4. The Shodan Trap (Exposed VPS Instances)

**What it is:** VPS deployments with public binding and no authentication are indexed by search engines like Shodan, exposing credentials and enabling command execution.

**How it works:**
- User deploys Gateway with `gateway.bind: "lan"` or `0.0.0.0` and forgets to configure auth
- Shodan indexes the open port within hours
- Attackers find exposed instances, access the dashboard, and extract API keys or execute commands

**Real-world examples:**
- Researchers found exposed OpenClaw instances with credentials and command execution via Shodan
- Shodan regularly indexes thousands of misconfigured development servers ([Shodan Help](https://help.shodan.io/mastery/vulnerability-assessment))

**Mitigations:**
- **Always** keep Gateway loopback-only: `gateway.bind: "loopback"`
- Use SSH tunnels or Tailscale Serve for remote access (see [Remote Access docs](https://docs.openclaw.ai/gateway/remote))
- If you must bind to LAN, enable authentication: `gateway.auth.enabled: true`
- Use DigitalOcean 1-Click Deploy which pre-configures security hardening
- Run `openclaw security audit --fix` after any configuration change
- Check your public IP on Shodan: `https://www.shodan.io/host/YOUR_IP`

### 5. Fake SaaS / API Key Vacuums

**What it is:** Third-party services offer to "host your bot" or provide "enhanced features" while harvesting your API keys and credentials.

**How it works:**
- Service claims to simplify deployment: "Just paste your Anthropic/OpenAI API key"
- Keys are stored and used for the operator's purposes (crypto mining, resale, abuse)
- Browser extensions impersonate official tools to capture credentials

**Real-world examples:**
- 459+ API keys exfiltrated via fake browser extensions ([Obsidian Security](https://www.obsidiansecurity.com/blog/small-tools-big-risk-when-browser-extensions-start-stealing-api-keys))
- 250+ exposed AI API keys found on GitHub via automated scanning ([DEV.to](https://dev.to/zaim_abbasi/claude-openai-google-api-keys-all-public-this-is-what-i-found-after-scanning-github-at-scale-fj5))

**Mitigations:**
- **Never** share API keys with third-party hosting services
- OpenClaw is self-hosted by design; there is no official managed service
- Use environment variables or credential files with 0600 permissions, not hardcoded keys
- Rotate API keys if you suspect exposure
- Monitor provider dashboards for unusual usage patterns

### Quick Protection Checklist

- [ ] Verify exact package name before `npm install openclaw`
- [ ] Only use official GitHub repo links from [docs.openclaw.ai](https://docs.openclaw.ai)
- [ ] Never share API keys with third-party "hosting" services
- [ ] Keep Gateway loopback-only or behind authentication
- [ ] Regularly check linked devices in WhatsApp/Telegram
- [ ] Run `openclaw security audit --deep` regularly
- [ ] Use encrypted disk (FileVault/LUKS) for credential protection
- [ ] Review installed plugins and their permissions
- [ ] Never run "prerequisite" terminal commands from skill docs without reviewing code
- [ ] Check VirusTotal scan status on ClawHub skill pages before installing
- [ ] Use Koi Security Scanner before installing ClawHub skills
- [ ] Verify npm package names exist on npmjs.com before installing (AI hallucination risk)
- [ ] Never allowlist `npm` or `npx` in shell tool allowlist
- [ ] Check for hidden `.mmd` files in skill directories before enabling
- [ ] Disable `skills.autoDiscover` to prevent automatic skill installation from skills.sh

### Threat Summary

| Threat | Attack Vector | Primary Risk | Detection |
|--------|--------------|--------------|-----------|
| **Typosquatting** | Supply chain | Credential theft, malware | Verify package metadata |
| **Handle sniping** | Social engineering | Phishing, malware distribution | Check account history |
| **Session stealing** | Malware, malicious plugins | Account takeover | Check linked devices |
| **Shodan exposure** | Misconfiguration | Full compromise | Check Shodan, audit config |
| **Fake SaaS** | Social engineering | API key theft | Never share keys externally |
| **ClawHub malicious skills** | Supply chain, social engineering | Credential theft, malware | Check VirusTotal scan status, review local scanner warnings, scan with Koi |
| **NPX/npm hallucination** | AI-recommended fake packages | Code execution, credential theft | Verify package exists on npmjs.com before install |
| **Hidden .mmd payloads** | UI-invisible skill files | Prompt injection, data exfiltration | `ls -laR` skill directory, check for non-.md/.ts files |
| **Skills.sh auto-install** | Unvetted skill distribution | Full Gateway compromise | Disable `skills.autoDiscover`, use ClawHub only |

### 6. ClawHub Malicious Skills (ClawHavoc Campaign)

**What it is:** ClawHub is a third-party skills marketplace for OpenClaw. In February 2026, security researchers discovered **341 malicious skills** (12% of audited packages) designed to steal credentials and install malware.

**How it works:**
- Attackers publish skills with professional-looking documentation
- "Prerequisites" section instructs users to run terminal commands or download files
- Commands fetch Atomic Stealer (macOS) or keyloggers (Windows) from attacker infrastructure
- Malware harvests crypto wallets, browser passwords, SSH keys, and API credentials

**Campaign details:**
- **Scale:** 341 malicious skills out of 2,857 audited (Koi Security)
- **Primary payload:** Atomic Stealer (AMOS) for macOS
- **Disguises:** Crypto tools (Solana trackers, Polymarket bots), YouTube utilities, ClawHub typosquats
- **Attack method:** Social engineering via fake "prerequisites", not code exploits

**Security improvements (Feb 2026):**
- **VirusTotal partnership:** ClawHub now scans all published skills through a 6-step pipeline (deterministic packaging, SHA-256 hashing, VirusTotal analysis with 70+ AV engines, Gemini LLM code review, auto-approval/blocking). Previously approved skills are rescanned daily.
- **OpenClaw local scanner:** Built-in pattern-based static analysis runs at install time, detecting dangerous code patterns (shell exec, eval, crypto mining, credential harvesting). See `src/security/skill-scanner.ts`.
- **Limitations:** Neither layer can detect social engineering (the ClawHavoc attack vector), prompt injection, or zero-day threats. A clean scan is not a guarantee of safety.

**Real-world sources:**
- [OpenClaw Blog - VirusTotal Partnership](https://openclaw.ai/blog/virustotal-partnership)
- [Koi Security ClawHavoc Report](https://www.koi.ai/blog/clawhavoc-341-malicious-clawedbot-skills-found-by-the-bot-they-were-targeting)
- [The Hacker News](https://thehackernews.com/2026/02/researchers-find-341-malicious-clawhub.html)
- [BleepingComputer](https://www.bleepingcomputer.com/news/security/malicious-moltbot-skills-used-to-push-password-stealing-malware)

**Mitigations:**
- **Never run prerequisite commands** without reading the code first
- Check VirusTotal scan status on the ClawHub skill page before installing
- Review local scanner warnings shown during skill installation
- Avoid skills less than 30 days old or from unknown publishers
- Use [Koi Security Scanner](https://koi.ai/clawhub-scanner) to check skills before installing
- Inspect skill code in `~/.openclaw/skills/` before enabling
- Be extremely suspicious of crypto-related skills
- Run OpenClaw in a VM/container for skill testing

For detailed analysis, see: [ClawHub Marketplace Risks](../05-worst-case-security/clawhub-marketplace-risks.md)

For detailed hardening guidance, see:
- [Hardening checklist](../04-privacy-safety/hardening-checklist.md)
- [Threat model](../04-privacy-safety/threat-model.md)
- Official security docs: https://docs.openclaw.ai/gateway/security
