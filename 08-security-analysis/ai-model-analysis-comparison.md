> **Navigation:** [Main Guide](../README.md) | [Security Audit Reference](./security-audit-command-reference.md) | [CVEs/GHSAs](./official-security-advisories.md) | [Issue #1796](./issue-1796-argus-audit.md) | [Medium Article](./medium-article-audit.md) | [ZeroLeeks](./zeroleeks-audit.md) | [Post-merge Hardening](./post-merge-hardening.md) | [Open Issues](./open-upstream-issues.md) | [Ecosystem Threats](./ecosystem-security-threats.md) | [SecurityScorecard](./securityscorecard-strike-report.md) | [Cisco AI Defense](./cisco-ai-defense-skill-scanner.md) | [Model Poisoning](./model-poisoning-sleeper-agents.md) | [Model Comparison](./ai-model-analysis-comparison.md)

## AI Model Analysis Comparison

This table summarizes how each AI model performed across the documentation task, helping readers assess source reliability.

### Coverage Comparison

| Topic | Opus 4.5 | Copilot GPT-5.2 | GLM 4.7 | Gemini 3.0 Pro | Kimi K2.5 |
|-------|----------|-----------------|---------|----------------|-----------|
| **Plain English intro** | Excellent | Excellent | Good | Good | Excellent |
| **Technical architecture** | Detailed diagrams | Good diagrams | Basic | Basic | Good diagrams |
| **Installation guide** | Complete | Complete | Complete | Brief | Complete |
| **Mac mini deployment** | Detailed runbook | Detailed runbook | Good | Brief | Good |
| **VPS deployment** | Detailed runbook | Detailed runbook | Good | Brief | Good |
| **Cloudflare Moltworker** | Referenced | Referenced | Referenced | Referenced | Good AI Gateway/D1/KV coverage but **missing Sandbox SDK + Access** |
| **Security/privacy** | Thorough | Thorough | Good | Brief | Detailed but flawed |
| **Configuration reference** | Good | Good | Complete | Partial | Complete |
| **Issue #1796 analysis** | Code-verified | Code-verified | Accurate | Inaccurate | Not verified |
| **Medium article analysis** | Code-verified | Mostly accurate | Hallucinated | Mostly inaccurate | Not verified |
| **ZeroLeeks analysis** | N/A | N/A | N/A | N/A | N/A |

### Security Analysis Accuracy

| Model | Issue #1796 Verdict | Medium Article Verdict | Methodology |
|-------|---------------------|----------------------|-------------|
| **Opus 4.5** | 0/8 exploitable | 0/8 exploitable | Source code verification with file/line references |
| **Copilot GPT-5.2** | 0/8 exploitable | 0/8 exploitable (minor error on claim 3) | Source code verification |
| **GLM 4.7** | 0/8 exploitable | N/A (analyzed wrong claims) | Source code verification |
| **Gemini 3.0 Pro** | Race condition claim accepted | 3 claims accepted at face value | No verification apparent |
| **Kimi K2.5** | 8/8 presented as vulnerabilities | 8/8 presented as vulnerabilities | No verification; accepted audits at face value |

### ZeroLeeks Audit Evaluation (Third Audit)

The [ZeroLeeks AI Red Team audit](https://zeroleaks.ai/reports/openclaw-analysis.pdf) (Jan 2026) was published after the original five documentation models completed their analyses, so none of them covered it. It was independently evaluated using the [`/consult-codex` dual-AI consultation skill](https://github.com/centminmod/my-claude-code-setup/blob/master/.claude/skills/consult-codex/SKILL.md):

| Evaluator | Role | ZeroLeeks Verdict |
|-----------|------|-------------------|
| **Opus 4.6** | Primary evaluator (this repo) | 0/34 exploitable; CRITICAL rating not justified |
| **Codex GPT-5.3** (OpenAI) | Second opinion via `/consult-codex` | CRITICAL rating not justified; "low as a security audit" |
| [**Code-Searcher**](https://github.com/centminmod/my-claude-code-setup/blob/master/.claude/agents/code-searcher.md) (Claude Opus 4.6; default model changed from Sonnet to Opus) | Deep codebase verification | Confirmed no defenses were tested; mapped external content pipeline |

All three evaluators agreed with **High confidence** on all major conclusions. See [ZeroLeeks Audit](./zeroleeks-audit.md) for full details.

### Unique Strengths

| Model | Primary Strength |
|-------|------------------|
| **Opus 4.5** | Most rigorous security verification with code citations |
| **Copilot GPT-5.2** | Best "attacker needs X access" contextual framing |
| **GLM 4.7** | Clear "audit claim vs reality" side-by-side tables |
| **Gemini 3.0 Pro** | Concise summaries (when accurate) |
| **Kimi K2.5** | Best beginner analogies ("think of it as..."); detailed D1/KV/Queues coverage (but missing Sandbox SDK) |

### Recommendation for Readers

- **For security assessment:** Use Opus 4.5 or Copilot GPT-5.2 (code-verified)
- **For Cloudflare deployment:** Use existing `explain-clawdbot/03-deploy/cloudflare-moltworker.md` (covers Sandbox SDK); Kimi K2.5 supplements D1/KV/Queues
- **For quick overview:** Use Gemini 3.0 Pro (verify claims against other sources)
- **For deployment checklists:** Use any model except Gemini 3.0 Pro
- **For beginner concepts:** Kimi K2.5 has excellent plain English analogies
