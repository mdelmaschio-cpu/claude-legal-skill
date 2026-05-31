# CLAUDE.md — claude-legal-skill

This file provides guidance to AI assistants working in this repository.

## Repository Purpose

A single Agent Skills-compatible skill (`contract-review`) for AI-powered legal contract analysis. Version 3.0.0. Grounded in the CUAD dataset (41 risk categories from 510 real contracts), ContractEval benchmarks, and LegalBench.

Compatible with: Claude Code, OpenAI Codex, Cursor, GitHub Copilot, Gemini CLI, and 26+ other tools following the Agent Skills standard.

## Repository Structure

```
claude-legal-skill/
├── README.md           # Full documentation, features, installation, benchmarks
├── CHANGELOG.md        # Version history (v1.0.0 → v3.0.0)
├── LICENSE             # MIT
├── skill.md            # The skill itself — the primary deliverable
└── examples/
    └── demo.png        # Sample output screenshot
```

## The Skill File: `skill.md`

Note: the filename is `skill.md` (lowercase), not `SKILL.md` — this is intentional.

### Frontmatter

```yaml
---
name: contract-review
description: Review legal contracts, NDAs, employment agreements, SaaS terms, and M&A documents...
version: 3.0.0
---
```

### Key Sections in `skill.md`

| Section | Purpose |
|---------|---------|
| Pre-Review Checklist | Catch blank fields, missing exhibits, signature status |
| Document Type + User Position | Adjusts what's flagged based on party (customer/vendor, buyer/seller, etc.) |
| Output Format + Example | Markdown tables, risk ratings, redlines |
| Red Flags Quick Scan | 12 instant danger-sign checks |
| Document Type Checklists | Specialized checklists: NDA, SaaS/MSA, Payment/Merchant, M&A, Finder/Broker |
| CUAD Risk Categories | 41+ categories: basics, term/termination, assignment, financial, liability, IP, dispute resolution |
| Market Standard Benchmarks | Thresholds: standard / yellow flag / red flag per provision |
| Negotiability Guide | High/Medium/Low/None ratings |
| Jurisdiction Notes | Non-compete enforceability, choice of law implications |
| Guardrails | Not legal advice, no hallucination, show what's acceptable |

## Risk Output Format

- 🔴 Critical — immediate attention required
- 🟡 Important — should be negotiated
- 🟢 Reviewed & Acceptable — verified as within market norms

Confidence tagging: 🟢 verified / 🟡 medium / 🔴 assumed

## Installation

```bash
# Claude Code (global)
git clone https://github.com/evolsb/claude-legal-skill ~/.claude/skills/contract-review

# Development (symlink for live edits)
git clone https://github.com/evolsb/claude-legal-skill ~/Developer/claude-legal-skill
ln -s ~/Developer/claude-legal-skill ~/.claude/skills/contract-review

# Other tools — clone to your tool's skills directory
```

## Usage Examples

```
Review this NDA - I'm the receiving party
Analyze the indemnification in this MSA - I'm the vendor
Review this acquisition agreement - I'm the seller
Check this merchant agreement - what's my chargeback exposure?
```

## Paired Tool: legal-redline-tools

This skill outputs structured JSON redlines. To produce tracked-changes `.docx` and redline PDFs:

```bash
pip install git+https://github.com/evolsb/legal-redline-tools.git
legal-redline apply contract.docx redlined.docx \
    --from-json redlines.json \
    --pdf redline.pdf \
    --memo-pdf internal-memo.pdf
```

## Development Conventions

- `skill.md` is the entire deliverable — do not split it into multiple files
- Only `name`, `description`, and `version` in frontmatter — no extra fields
- Output format is markdown (NOT XML tags)
- Always output a "Reviewed & Acceptable" section alongside risk findings
- The disclaimer ("not legal advice") must appear in every output
- Market benchmarks are empirical US-law data — do not change without citing updated market sources

## Accuracy Note

Based on ContractEval benchmarks, clause extraction achieves F1 ~0.62. Best for first-pass review and issue flagging. Recommend attorney review for any material contract terms.

## Important Notes for AI Assistants

- `skill.md` (lowercase) is the skill file — this is intentional, do not rename it
- Position-aware review is the key differentiator: always ask which party the user represents
- The CUAD 41 risk categories are the canonical reference — do not invent new categories
- Market benchmarks (liability cap, non-compete duration, etc.) have specific thresholds — preserve them exactly
- "Reviewed & Acceptable" section is required in every output — do not omit it even when all findings are critical
- ContractEval accuracy (F1 ~0.62) means human attorney review is needed for material deals — always say this
