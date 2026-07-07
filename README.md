> [查看中文文档](README_zh.md)

# Full-Stack Cybersecurity Expert

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![SKILL.md](https://img.shields.io/badge/Agent%20Skill-v1.0.0-blue)](SKILL.md)

> A battle-tested AI Agent cybersecurity skill pack. Covering Red/Blue Team operations, penetration testing, incident response, code audit, threat intelligence, and security assessment.
> **Claude Code / Codex CLI / ChatGPT** — multi-platform compatible.
>
> **A protocol, not a prompt** — Mode switching, state management (with resume), token economy, anti-hallucination checks, and persistent rollback auditing. Engineered for real operations.

---

## Overview

This skill pack provides a standardized cybersecurity operations framework for AI agents like **Claude Code / Codex CLI / ChatGPT**. It is not a simple "you are a security expert" prompt — it is an executable **protocol-level specification**.

### Four Operational Modes

| Mode | Capability | Typical Scenario |
|------|-----------|-----------------|
| 🔴 **Red Team** | Kill chain exploitation, manual foothold, on-the-fly PoC/EXP crafting | Penetration testing, vulnerability exploitation, full-spectrum attack simulation |
| 🔵 **Blue Team IR** | High-volume log triage, attack chain traceback | Incident response, alert analysis, intrusion investigation, malware analysis |
| 🟢 **Code Review** | Cross-file taint tracking, defense layer evaluation | Code audit, security hardening, vulnerability verification |
| ⚪ **Advisory** | Standard-based assessment, quantified compliance scoring | Security assessment, threat intelligence, compliance, security training |

### Core Design Philosophy

This skill addresses several pain points unique to AI agents performing long-running security tasks:

1. **Context Drift** — State management ensures the agent remembers progress across dozens of turns
2. **Token Waste** — Token economy rules force the agent to optimize input, saving resources for analysis
3. **Hallucination Control** — Three-layer defense: CVE verification, confidence tagging, and PoC self-check
4. **Operational Safety** — Human-in-the-Loop red lines and mandatory rollback auditing prevent leftover backdoors

---

## File Structure

```
full-stack-cybersecurity-expert/
├── SKILL.md                     # 🎯 Agent instructions — Chinese
├── SKILL_en.md                  # 🎯 Agent instructions — English
├── README.md                    # 👤 English docs (GitHub default)
├── README_zh.md                 # 👤 Chinese docs
├── LICENSE                      # MIT License
├── SECURITY.md                  # Security policy
├── CONTRIBUTING.md              # Contribution guidelines
├── .gitignore                   # Git ignore rules
│
├── references/                  # On-demand deep-dive materials
│   ├── mode-definitions.md      #   Complete phase flows + vuln submission format
│   ├── state-persistence.md     #   State JSON format + resume protocol
│   ├── code-audit-deep-dive.md  #   Code audit spec (Source/Sink tables, etc.)
│   ├── blocking-rules.md        #   Production middleware blocking rules (Nginx/Apache/IIS)
│   ├── giant-log-protocol.md    #   >1GB log slicing protocol
│   ├── anti-hallucination.md    #   Anti-hallucination checklist
│   ├── ad-security.md           #   AD security (Kerberos attacks, ACL abuse, AD CS, etc.)
│   ├── cloud-security.md        #   Cloud security (AWS/GCP/Azure, K8s, etc.)
│   ├── mobile-security.md       #   Mobile security (Android/iOS pentest, OWASP Mobile Top 10 2024)
│   ├── attack_framework.md      #   MITRE ATT&CK technique mappings
│   ├── owasp_top10.md           #   OWASP Top 10 (2021)
│   ├── commands_cheatsheet_offensive.md  #   Pentest command cheatsheet (recon/web/intranet/crack/proxy)
│   ├── commands_cheatsheet_defensive.md  #   IR command cheatsheet (Linux/Windows/Webshell/Event Log)
│   └── report_template.md       #   Security assessment report template
│
├── examples/                    # Usage examples
│   ├── log-analysis-example.md  #   Log analysis walkthrough
│   └── pentest-workflow-example.md # Pentest workflow walkthrough
│
└── scripts/                     # Dynamic weapon forge (agent writes PoCs here at runtime)
```

---

## Usage

### With Claude Code / Other Agents

```bash
# Place the directory under your agent's skills/ path
# Claude Code: ~/.claude/skills/full-stack-cybersecurity-expert/
# Generic:     $AGENT_HOME/skills/full-stack-cybersecurity-expert/
```

### Other Platforms

```bash
# Codex CLI / ChatGPT: inject SKILL.md content as system prompt
```

> **Tip**: The core of this project is `SKILL.md`. Any AI tool that supports system prompt injection can use its content directly.

### Trigger Examples

| You say | Agent responds with |
|---------|-------------------|
| "Run a web pentest" | 🔴 Red Team |
| "Write an SSRF PoC" | 🔴 Red Team |
| "Audit this AD domain" / "Internal network pentest" | 🔴 Red Team — AD |
| "Security test this cloud app" | 🔴 Red Team — Cloud |
| "Pentest this mobile app" | 🔴 Red Team — Mobile |
| "Triage this WAF alert" | 🔵 Blue Team IR |
| "Server got hacked — IR now" | 🔵 Blue Team IR |
| "Analyze this malware sample" | 🔵 Blue Team IR |
| "Code review this Java class" | 🟢 Code Review |
| "Design a CIS benchmark checklist" | ⚪ Advisory |
| "Is this IP malicious?" | ⚪ Advisory |
| "Explain SQL injection and mitigations" | ⚪ Advisory |

### Task Execution Flow

Every task follows a five-step protocol:

```
Step 1: Confirm objective + authorization → Step 2: Mode switch + planning → Step 3: Iterative execution
→ Step 4: Findings + remediation → Step 5: Report + cleanup + rollback audit
```

---

## Design Philosophy

### Why not just "you are a security expert"?

Security operations demand more than a simple role preset:

- **Divergent Thinking vs. Convergent Thinking** — Pentesting requires creative attack chains; incident response demands disciplined evidence chains. Forcing both into one reasoning model degrades performance.
- **Long Task Spans** — A full pentest may require dozens of turns. Without state management, the agent forgets.
- **High Stakes** — A wrong command can trigger WAF alerts, corrupt production data, or leave backdoors behind.
- **Rigorous Output** — CVE IDs cannot be fabricated. Exploit chains must close. Fixes must be runnable.

This skill solves these problems through **protocol engineering**, not prompt engineering.

### Key Mechanisms

| Mechanism | Problem Solved |
|-----------|---------------|
| **Mode Switching** | Different tasks use different reasoning models — no cognitive collision |
| **State Management** | Long tasks survive across turns; resume after interruption |
| **Token Economy** | Hard limits optimize context usage; more resources for real analysis |
| **Anti-Hallucination** | CVEs must be verified, PoCs must self-check — think before output |
| **Rollback Audit** | Mandatory cleanup checklist after every pentest — no forgotten backdoors |

---

## License

MIT License — see [LICENSE](LICENSE)

## Disclaimer

This skill pack is designed for authorized security testing and research. All technical information is for legitimate and compliant use only. Users must confirm they have written authorization for the target system. The author assumes no liability for misuse.
