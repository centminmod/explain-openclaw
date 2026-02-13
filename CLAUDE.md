# explain-clawdbot Directory

This directory contains synthesized documentation for OpenClaw, combining AI-generated analyses with source-code verification.

## Purpose

1. **Beginner-friendly documentation** - Plain English explanations for non-technical users
2. **Technical deep-dives** - Architecture, code mapping, and deployment guides
3. **Security analysis** - Verified audits, threat models, worst-case scenarios
4. **Multi-model synthesis** - Reconciles analyses from Opus 4.5, GPT-5.2, GLM 4.7, Gemini 3.0 Pro, Kimi K2.5

## Key Principle

> **Repo docs + code win.** Model summaries are supporting material.

When AI-generated content conflicts with source code, trust the code.

## Directory Structure

| Path | Purpose |
|------|---------|
| `README.md` | Main entry point with TOC, FAQ, worst-case scenarios (security content extracted to `08-security-analysis/`) |
| `01-plain-english/` | Beginner explanations (`what-is-clawdbot.md`), glossary |
| `02-technical/` | Architecture diagrams, repo map |
| `03-deploy/` | Mac mini, VPS, Cloudflare runbooks |
| `04-privacy-safety/` | Threat model, hardening checklist |
| `05-worst-case-security/` | Attack scenarios, prompt injection examples, misconfiguration guides |
| `08-security-analysis/` | Extracted security audits, CVEs, hardening log, upstream issues, ecosystem threats, model comparison |
| `99-reference/` | Commands, troubleshooting |

## Maintenance Workflow

- After completing documentation edits, always verify ALL files referenced in the plan were actually updated. Cross-check the plan's file list against git diff before reporting completion.
- Never report findings as 'new' or 'valid' without first verifying them against the actual codebase. If you discover something that seems noteworthy, grep the codebase and existing docs to confirm it's not already covered or fabricated.

### After upstream commits merge

1. Run `/sync-explain-docs-with-upstream` skill
2. Update security audit sections if commits touch security controls
3. Verify line number references still match source code
4. Add "Post-Merge Hardening" entry to `08-security-analysis/post-merge-hardening/` and update TOC in `post-merge-hardening.md`

### After doc sync or multi-file update

Run the sync-public-repo pipeline to push changes to the public repo unless told otherwise.

### Adding new security findings

1. Verify claims against source code (file:line references required)
2. Add to appropriate file in `08-security-analysis/` or create new file in `05-worst-case-security/`
3. Update TOC in README.md
4. Cross-reference with existing audits (Issue #1796, Medium article)

### CLI command verification

When verifying CLI commands or tool flags in documentation, always check platform compatibility (Linux vs macOS vs BSD). Flag any Linux-only commands (e.g., `iostat -x`, `top -p`) and provide cross-platform alternatives.

### Adding ecosystem threats

1. Verify threat with reputable security sources (not just social media claims)
2. Add to `08-security-analysis/ecosystem-security-threats.md`
3. Include: description, mechanism, real-world examples, mitigations
4. Update the "Threat Summary" table

### Rebrand handling

The project has been renamed: Clawdbot -> Moltbot -> OpenClaw. When updating:

- Use "OpenClaw" for current references
- Keep historical references accurate (e.g., "Issue #1796" from Clawdbot era)
- CLI command is `openclaw`

## Security Audit Sections

Security audit analyses are in `08-security-analysis/`:

### Issue #1796 (Argus Security Platform)

- 512 findings, 8 CRITICAL claims
- **Verdict: 0/8 exploitable as described**
- All claims verified against source code with file/line references

### Medium Article (Saad Khalid)

- 8 claimed zero-day vulnerabilities
- **Verdict: 0/8 exploitable as described**
- Three legitimate defense-in-depth gaps identified (still open)

### Ecosystem Threats Section

- 5 supply chain/social engineering threats
- Not codebase vulnerabilities; user education focus
- Prompted by community advisory from @edgeaiplanet

## Source AI Analyses

Individual model analyses are stored in sibling directories:

- `../explain-clawdbot-opus-4.5/` - Most rigorous security verification
- `../explain-clawdbot-copilot-gpt-5.2/` - Best contextual framing
- `../explain-clawdbot-glm-4.7/` - Clear side-by-side tables
- `../explain-clawdbot-gemini-3.0-pro/` - Concise (verify claims)
- `../explain-clawdbot-kilocode-kimi-k2.5/` - Best beginner analogies

## Git Exclusions

- **Never commit `.private/` or `.claude/` files.** Both directories are excluded via `.git/info/exclude` and must not be force-added or staged. `.private/` contains local-only reports; `.claude/` contains local skills/settings. Neither should reach the remote.

## Related Resources

- Official docs: <https://docs.openclaw.ai>
- Security docs: <https://docs.openclaw.ai/gateway/security>
- Security policy: `../SECURITY.md`
- Source code: `../src/`
- Canonical docs: `../docs/`

<claude-mem-context>
# Recent Activity

### Feb 5, 2026

| ID | Time | T | Title | Read |
|----|------|---|-------|------|
| #1 | 11:51 PM | 🔵 | Comprehensive OpenClaw Security Documentation Repository | ~925 |

</claude-mem-context>
