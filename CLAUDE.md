# CLAUDE.md — claude-legal-skill

This file provides guidance to AI assistants working in this repository.

## Project Overview

**claude-legal-skill** (published as `contract-review`) is a single Agent Skills-standard skill for AI-powered legal contract review. It is grounded in the CUAD dataset (41 risk categories from 510 real contracts) and the ContractEval benchmark. It supports Claude Code, OpenAI Codex, Cursor, Gemini CLI, and any Agent Skills-compatible tool.

- **No build step** — single `skill.md` file plus `examples/`
- **License**: MIT
- **Version**: 3.0.0

## Repository Layout

```
claude-legal-skill/
├── skill.md              # Main skill — frontmatter + full review instructions
├── examples/
│   └── demo.png          # Example output screenshot
├── CHANGELOG.md
├── LICENSE
└── README.md
```

## Skill Architecture

`skill.md` is the complete skill. It implements a multi-step review workflow:

1. **Pre-review checklist** — detects blank fields, missing exhibits, signature status
2. **Document type + party identification** — asks which party the user is; adjusts risk framing accordingly
3. **CUAD-grounded analysis** — 41 clause categories, risk severity rating (Critical / Important / Acceptable)
4. **Market standard benchmarks** — compares terms against industry norms with quantitative thresholds
5. **Negotiability ratings** — flags what is and isn't realistically changeable
6. **Redlines** — outputs specific replacement language, not just "negotiate this"
7. **Missing provisions** — suggests language for gaps
8. **Internal consistency check** — broken cross-references, undefined terms

### Frontmatter

```yaml
---
name: contract-review
description: Review legal contracts, NDAs, employment agreements, SaaS terms, and M&A documents. Identifies unfavorable terms, suggests redlines, and compares to market standards. Use for contract analysis, due diligence, or negotiation prep.
version: 3.0.0
---
```

### Activation Triggers

The skill activates when the user:
- Says "review contract", "analyze agreement", "check this contract"
- Uploads or references a PDF/DOCX legal document
- Asks about specific clauses, risks, or terms in a document

### Position-Aware Review

The skill asks which party the user represents (customer, vendor, buyer, seller, receiving party, disclosing party) and adjusts what it flags as risky. Without a stated position, it defaults to the reviewer's perspective.

### Supported Document Types

| Type | Special Handling |
|------|-----------------|
| NDA | Confidentiality term, non-solicitation, standstill, destruction certification |
| SaaS/MSA | SLA, data export, suspension rights, price caps |
| Payment/Merchant | Reserves, chargebacks, network rules, auto-debit |
| M&A | Earnouts, escrow, rep survival, sandbagging |
| Finder/Broker | Fee tails, covered buyer definitions, joint representation |

## Development Workflow

There is **no build step**. All changes are edits to `skill.md`.

### Updating the Skill

1. Edit `skill.md` — the frontmatter `version` must be bumped on any substantive change
2. Update `CHANGELOG.md` with the change description
3. Test by invoking the skill against at least one real contract per changed section
4. Keep the `description` frontmatter trigger-phrase-rich for correct AI tool invocation

### Adding Examples

Place screenshots or sample outputs in `examples/`. Reference them in `README.md` only — not inside `skill.md` itself (keeps the skill body clean and token-efficient).

## Legal Redline Tooling

The skill outputs structured JSON redlines. To convert those into tracked-changes Word docs or PDFs, pair with the external `legal-redline-tools` package (not in this repo):

```bash
pip install git+https://github.com/evolsb/legal-redline-tools.git
legal-redline apply contract.docx redlined.docx \
    --from-json redlines.json \
    --pdf redline.pdf
```

## Key Conventions

- **Output is Markdown** — the skill produces readable Markdown reviews; never use XML tags in output
- **Not legal advice** — the skill body and README must include the disclaimer; do not remove it
- **Structured redlines** — all suggested language changes are in structured JSON, not freeform prose, to support tooling integration
- **US law default** — analysis defaults to US jurisdiction; flag explicitly when provisions vary by state (e.g., non-competes void in CA/ND/OK/MN)
- **Benchmark tables** — market standard thresholds (liability cap, auto-renewal notice, etc.) are maintained in the skill body; update them if industry norms shift and you can cite a source
- **CUAD grounding** — all 41 CUAD risk categories should remain covered; do not remove category coverage without replacement
