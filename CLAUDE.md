# CLAUDE.md — claude-legal-skill

This file provides guidance to Claude Code when working in this repository.

## Project Purpose

**Contract Review** is an agent skill for legal analysis of contracts and legal documents. It provides AI-powered, position-aware contract review grounded in the [CUAD dataset](https://github.com/TheAtticusProject/cuad) (41 legal risk categories from 510 real contracts), ContractEval benchmarks, and LegalBench. It works with Claude Code, OpenAI Codex, Cursor, GitHub Copilot, Gemini CLI, and any [Agent Skills](https://agentskills.io)-compatible tool.

The skill analyzes legal contracts and outputs risk assessments with severity ratings, red flags, key terms, market standard benchmarks, negotiability ratings, specific redline language, missing provisions, and internal consistency checks. It is position-aware — the risk profile changes depending on whether the user is a customer, vendor, buyer, seller, or other party.

This is a **single-skill repository** with one primary file (`skill.md`), example outputs, and a README. There is no build process or installer — install by cloning to the tool's skills directory.

## Repository Structure

```
claude-legal-skill/
├── README.md           # Installation, usage examples, features, credits, accuracy notes
├── CHANGELOG.md        # Version history
├── LICENSE             # MIT
├── skill.md            # The primary skill file — this is the entire deliverable
│
└── examples/           # Sample outputs for different contract types
    ├── demo.png                  # Screenshot of demo output
    ├── nda-review.md             # Example NDA review output
    ├── saas-agreement-review.md  # Example SaaS MSA review output
    ├── ma-agreement-review.md    # Example M&A agreement review
    └── balanced-agreement.md     # Example of a well-balanced contract
```

## The Primary Artifact: skill.md

`skill.md` is the entire skill. It uses a YAML frontmatter block followed by the complete skill instructions:

```yaml
---
name: contract-review
description: Review legal contracts, NDAs, employment agreements, SaaS terms,
  and M&A documents. Identifies unfavorable terms, suggests redlines, and
  compares to market standards. Use for contract analysis, due diligence,
  or negotiation prep.
version: 3.0.0
---
```

The skill body contains:
- Pre-review checklist (blank fields, missing exhibits, document status)
- Position identification workflow
- Complete output format with example
- Red flags quick scan table (12 danger signs)
- Document-type checklists (NDA, SaaS/MSA, Payment/Merchant, M&A, Finder/Broker)
- CUAD 41 risk categories + extensions
- Market standard benchmarks with numeric thresholds
- Negotiability guide
- Jurisdiction notes
- Guardrails

## Installation

```bash
# Claude Code
git clone https://github.com/evolsb/claude-legal-skill ~/.claude/skills/contract-review

# OpenAI Codex
git clone https://github.com/evolsb/claude-legal-skill ~/.codex/skills/contract-review

# Development symlink (recommended for contributors)
git clone https://github.com/evolsb/claude-legal-skill ~/Developer/claude-legal-skill
ln -s ~/Developer/claude-legal-skill ~/.claude/skills/contract-review
```

No build step, no dependencies, no installer required.

## Core Skill Features

### Position-Aware Analysis

The skill's most important behavior: always determine which party the user represents before analyzing risk. The same clause is favorable or unfavorable depending on position:

- Customer reviewing vendor MSA → flag vendor-favorable terms
- Vendor reviewing own template → flag customer-favorable terms
- Buyer in M&A → flag seller-favorable terms
- Receiving party in NDA → flag disclosing-party-favorable terms

**Always ask if position is unclear before analyzing.**

### Document Type Checklists

Specialized checklists are built into the skill for:
- **NDA**: confidentiality term, standstill, residuals, destruction certification
- **SaaS/MSA**: SLA, data export, suspension rights, price caps, auto-renewal notice
- **Payment/Merchant**: reserves, chargebacks, network rules, auto-debit
- **M&A**: earnouts, escrow, rep survival, sandbagging, working capital
- **Finder/Broker**: fee tails, covered buyer definitions, joint representation

### Market Standard Benchmarks

The skill includes specific numeric thresholds for common terms — apply these consistently:

| Provision | Standard | Yellow | Red |
|-----------|----------|--------|-----|
| Liability cap | 12 months | 6-11 months | <6 months |
| Non-compete | 1-2 years | 3-4 years | 5+ years |
| Auto-renewal notice | 90+ days | 60-89 days | <60 days |
| NDA confidentiality | 3 years | 2 years | 5+ years |
| Fee tail (broker) | 12-18 months | 24 months | Perpetual |
| Data export | 90 days, standard format | 30 days | None |

## Output Format

The skill produces **structured markdown only** — no XML tags. The standard structure (in order):

1. Document metadata header (type, position, counterparty, risk level, status)
2. Pre-signing alerts (blank fields, missing exhibits) — if any
3. Executive summary (1–2 paragraphs)
4. Key terms table (term, value, location)
5. Red flags quick scan table
6. Risk analysis: Critical / Important / Acceptable
7. Missing provisions table with suggested language
8. Internal consistency issues
9. Negotiation priority table (ranked by negotiability and importance)
10. Legal disclaimer

For each Critical and Important item, the format is:
```
**[Provision Name]** (Section X.Y)
> [Exact quoted text from contract]

- **Issue:** [What's wrong and why it matters]
- **Market Standard:** [What's typical]
- **Negotiability:** [High/Medium/Low — why]
- **Redline:** [Exact replacement language]
- **Fallback:** [Acceptable compromise]
```

Redlines must be **specific** — exact replacement language, not "negotiate this."

## Guardrails (Non-Negotiable)

These constraints are built into the skill and must not be removed or weakened:

1. **Legal disclaimer** — Every output ends with: "This review is for informational purposes only. Material terms should be reviewed by qualified legal counsel."
2. **No hallucination** — Only reference text actually in the document. Never invent clauses.
3. **Express uncertainty** — When a clause is genuinely ambiguous, say so and recommend attorney review.
4. **Show acceptable terms** — Always include a "Reviewed & Acceptable" section.
5. **Document status** — Note if a contract is already executed (review is informational only).
6. **Jurisdiction flags** — Note when governing law affects enforceability (especially non-competes and arbitration).

## Versioning

The `version` field in `skill.md` frontmatter follows semver. Bump on any change to skill behavior:
- **Patch** (x.x.Z): Wording improvements, clarifications that don't change behavior
- **Minor** (x.Y.0): New risk categories, new document type checklists, enhanced output
- **Major** (X.0.0): Breaking changes to output format or fundamental workflow changes

Update CHANGELOG.md with every version bump.

## Development Workflow

### Editing the Skill

All editing happens in `skill.md`. When editing:
- Keep the output format structure intact — users and downstream tools depend on it
- Test against the example contracts in `examples/` after any significant change
- Verify market standard benchmarks are accurate before updating
- Do not add references to external tools or services users need to set up separately

### Adding a New Document Type

1. Add a new checklist section in `skill.md` under "Document Type Checklists"
2. Include: key provisions, standard terms, common traps for each party
3. Add market standard benchmarks for the new type
4. Create an example output in `examples/` and reference it in README.md

### Pairing with legal-redline-tools

The skill can output structured JSON redlines when specifically requested. These can be processed by [legal-redline-tools](https://github.com/evolsb/legal-redline-tools) to produce tracked-changes Word documents and redline PDFs. This integration is external — do not manage it in this repo.

## Important Files

| File | Role |
|------|------|
| `skill.md` | The entire skill — workflow, checklists, benchmarks, guardrails |
| `examples/` | Sample outputs for NDA, SaaS, M&A, and balanced agreements |
| `CHANGELOG.md` | Version history — update with every behavioral change |
| `README.md` | Installation, features, usage examples, accuracy notes, credits |

## How AI Assistants Should Behave Here

- **`skill.md` is the entire deliverable** — all editing effort focuses there
- **Never remove the guardrails** — the legal disclaimer and no-hallucination constraints protect users
- **Keep output format stable** — downstream users may be parsing the structured output
- **Test against the examples** — verify output matches the format in `examples/` before shipping changes
- **Benchmarks must be accurate** — market standard numbers should reflect current commercial practice
- **Position-awareness is the differentiator** — preserve and strengthen this feature
- **Negotiability ratings must be realistic** — "High" means the counterparty commonly accepts this change
- **One file, no build step** — resist adding complexity; the skill's value is its simplicity
