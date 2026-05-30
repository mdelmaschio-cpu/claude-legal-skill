# Contract Review Skill

A single-file agent skill for position-aware legal contract review. Compatible with Claude Code, OpenAI Codex, Cursor, GitHub Copilot, Gemini CLI, and 26+ other AI coding tools via the open [Agent Skills standard](https://agentskills.io).

Grounded in CUAD (41 risk categories from 510 real contracts), ContractEval benchmarks, and LegalBench.

## Repository Structure

```
claude-legal-skill/
├── skill.md           # The complete skill — all instructions, checklists, benchmarks
├── README.md          # Installation, usage, features, accuracy notes
├── CHANGELOG.md       # Version history
├── LICENSE            # MIT
└── examples/          # Sample review outputs (demo.png and example contracts)
```

## The Skill: `skill.md`

This single file IS the entire skill. When Claude reviews a contract, it reads `skill.md` and follows its instructions. The file contains:

- **YAML frontmatter** — `name: contract-review`, trigger description, `version: 3.0.0`
- **Pre-review checklist** — blank fields, missing exhibits, execution status check
- **Position identification** — user's party role changes what is flagged as risky
- **Output format specification** — structured markdown template with risk levels, key terms table, red flags scan, redline suggestions
- **Red flags quick scan** — 12 danger signs to check before deep analysis
- **Document type checklists** — NDA, SaaS/MSA, Payment/Merchant, M&A, Finder/Broker
- **CUAD 41 risk categories + extensions** — complete taxonomy of contract risk
- **Market standard benchmarks** — provision-by-provision comparison table with yellow/red thresholds
- **Negotiability guide** — High/Medium/Low/None ratings with power dynamic factors
- **Jurisdiction notes** — non-competes by state, choice of law, arbitration venues
- **Guardrails** — express uncertainty, no hallucination, flag but don't opine on tax

## Installation

```bash
# Claude Code
git clone https://github.com/evolsb/claude-legal-skill ~/.claude/skills/contract-review

# OpenAI Codex
git clone https://github.com/evolsb/claude-legal-skill ~/.codex/skills/contract-review

# Development symlink
git clone https://github.com/evolsb/claude-legal-skill ~/Developer/claude-legal-skill
ln -s ~/Developer/claude-legal-skill ~/.claude/skills/contract-review
```

## Usage

```
Review this NDA - I'm the receiving party
Analyze the indemnification in this MSA - I'm the vendor
Review this acquisition agreement - I'm the seller
Check this merchant agreement - what's my chargeback exposure?
```

## Output Structure (always in this order)

1. Document header (type, party, counterparty, risk level, status)
2. Pre-signing alerts (blank fields, missing exhibits)
3. Executive summary
4. Key terms table
5. Red flags quick scan table
6. Risk analysis: 🔴 Critical → 🟡 Important → 🟢 Acceptable
7. Missing provisions with suggested language
8. Internal consistency issues (broken cross-references, undefined terms)
9. Negotiation priority ranked list

## Key Market Benchmarks

| Provision | Market Standard | Yellow Flag | Red Flag |
|-----------|----------------|-------------|----------|
| Liability cap | 12 months' fees | 6–11 months | <6 months |
| Non-compete duration | 1–2 years | 3–4 years | 5+ years |
| Auto-renewal notice | 90+ days | 60–89 days | <60 days |
| SLA uptime | 99.9% with credits | 99.5% | No SLA |
| Rep survival (M&A) | 12–18 months | 24–30 months | 36+ months |
| NDA confidentiality term | 3 years | 2 years | 5+ years |

## Companion Tool

[legal-redline-tools](https://github.com/evolsb/legal-redline-tools) converts this skill's structured JSON redline output to tracked-changes `.docx`, redline PDFs, and negotiation memos.

```bash
pip install git+https://github.com/evolsb/legal-redline-tools.git
legal-redline apply contract.docx redlined.docx --from-json redlines.json
```

## Accuracy

Claude achieves F1 ~0.62 on ContractEval clause extraction benchmarks. This skill is appropriate for first-pass review and issue identification — not a replacement for attorney review on material deals. Always communicate this limitation.

## AI Assistant Guidelines

- **Edit only `skill.md`** — this is the complete skill; the skill is intentionally single-file by design
- **Guardrails are mandatory** — never remove the "not legal advice" disclaimer or the "no hallucination" guardrail
- **Position-awareness must be preserved** — the skill must ask for the user's party role if not stated; this is what makes the review useful
- **Benchmark tables must stay current** — update market standards when industry norms shift
- **The output format is load-bearing** — the structured markdown output format is consumed by `legal-redline-tools`; do not break the section order or headings
- **Add document types by extending existing checklists** — follow the NDA/SaaS/M&A checklist format; do not create separate files
- **CUAD categories are the taxonomy backbone** — new risk types should be mapped to an existing CUAD category or explicitly added to the extensions section
- **Accuracy disclosure is required** — always surface the F1 ~0.62 limitation note in README and in outputs for material deals
