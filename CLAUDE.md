# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **documentation-only AI skill** — no source code, no build system, no dependencies. It defines a contract review skill for Claude Code and 26+ compatible AI tools (Cursor, GitHub Copilot, Gemini CLI, OpenAI Codex, etc.).

The skill is distributed by users cloning it into their tool's skills directory (e.g., `~/.claude/skills/contract-review`). The AI tool reads `skill.md` at runtime and follows its instructions when the user shares a contract for review.

This repository follows the open [Agent Skills standard](https://agentskills.io), not the superpowers-skills convention. The skill manifest uses a `description` field (not `when_to_use`) for auto-discovery by AI tools.

## Repository Structure

```
claude-legal-skill/
├── CLAUDE.md              # This file — AI assistant guidance
├── CHANGELOG.md           # Version history (semver)
├── LICENSE                # MIT
├── README.md              # User-facing docs: installation, usage, features, benchmarks
├── skill.md               # The authoritative skill manifest (YAML frontmatter + ~430 lines)
├── .gitignore             # Excludes .venv, IDE files, .zdai/, contracts/, *.pdf, *.docx
└── examples/
    ├── demo.png                    # Screenshot used in README
    ├── balanced-agreement.md       # SaaS, customer, LOW risk — affirms acceptable terms
    ├── ma-agreement-review.md      # M&A, seller position, HIGH risk
    ├── nda-review.md               # NDA, receiving party, MEDIUM risk
    └── saas-agreement-review.md    # SaaS, customer position, HIGH risk
```

## Key Files

### skill.md — The Runtime Skill Manifest

This is the only file the AI tool loads at runtime. It has two parts:

**YAML frontmatter** (lines 1-5):
```yaml
---
name: contract-review
description: Review legal contracts, NDAs, employment agreements, SaaS terms, and M&A documents. Identifies unfavorable terms, suggests redlines, and compares to market standards. Use for contract analysis, due diligence, or negotiation prep.
version: 3.0.0
---
```

The `description` field drives auto-discovery — AI tools match user requests against it. Keep it specific and third-person. The `name` field must match the directory name where users install the skill.

**Skill body** (~430 lines of structured markdown) covering:
- When to activate (trigger phrases)
- Step 1: Pre-review checklist (blank fields, missing exhibits, signature status)
- Step 2: Position identification (customer/vendor/buyer/seller/etc.)
- Output format spec with a full annotated example
- Red flags quick scan table (12 danger signs)
- Document-type checklists: NDA, SaaS/MSA, Payment/Merchant, M&A, Finder/Broker
- Risk categories: 41 original CUAD categories + ~43 extensions (84 total)
- Market standard benchmarks table with numeric thresholds
- Negotiability guide (High/Medium/Low/None ratings)
- Jurisdiction notes (non-competes, governing law, arbitration venues)
- Guardrails (not legal advice, no hallucination, always show acceptable terms)

### README.md — User-Facing Documentation

Installation guide, feature overview, usage examples, and links to the companion project `legal-redline-tools`. The README version badge must always match the version in `skill.md` YAML frontmatter — they are coupled and must be updated together.

The README references `legal-redline-tools` (https://github.com/evolsb/legal-redline-tools) as a companion Python project for generating tracked-changes Word docs and redline PDFs from the skill's output. That project lives in a separate repository — do not conflate the two.

### examples/ — Quality Baseline

Four full sample outputs that serve as the authoritative reference for correct skill behavior:

| File | Contract Type | User Position | Risk Level | Key Purpose |
|------|--------------|---------------|------------|-------------|
| `nda-review.md` | NDA | Receiving party | Medium | Standard NDA analysis |
| `saas-agreement-review.md` | SaaS | Customer | High | High-risk vendor agreement |
| `ma-agreement-review.md` | M&A acquisition | Seller | High | Complex deal review |
| `balanced-agreement.md` | SaaS | Customer | Low | **Demonstrates affirming acceptable terms** |

The `balanced-agreement.md` example is the most important one to preserve. It exists specifically to demonstrate that the skill must include a "Reviewed & Acceptable" section — it cannot just surface problems. Every review output must affirm acceptable terms, not only flag risks.

### CHANGELOG.md — Version History

All four versions (1.0.0 through 3.0.0) were released on 2026-01-26 during rapid iteration. Follow semver. Every change must be documented here.

## Development Environment

No setup required. This is a pure markdown repository with no runtime dependencies, no package manager, no virtual environment, and no build step.

The `.gitignore` excludes:
- `.venv/`, `venv/`, `env/` — Python environments (for local tooling only)
- `.zdai/` — Zuva AI config containing API tokens
- `contracts/`, `*.pdf`, `*.docx` — actual contract documents (privacy)
- `.env`, `.env.local` — environment files

**Never commit actual contract documents.** The gitignore exclusions for `contracts/` and `*.pdf/*.docx` are intentional privacy safeguards.

## Commands

There are no build, test, lint, or run commands.

**Installation (end users):**
```bash
# Claude Code
git clone https://github.com/evolsb/claude-legal-skill ~/.claude/skills/contract-review

# OpenAI Codex
git clone https://github.com/evolsb/claude-legal-skill ~/.codex/skills/contract-review

# Other Agent Skills-compatible tools — clone to your tool's skills directory
```

**Development (symlink for local editing):**
```bash
git clone https://github.com/evolsb/claude-legal-skill ~/Developer/claude-legal-skill
ln -s ~/Developer/claude-legal-skill ~/.claude/skills/contract-review
```

**Quality validation (manual — the only quality gate):**
```
1. Paste a real contract into Claude Code
2. Trigger the skill: "Review this NDA - I'm the receiving party"
3. Compare output structure against examples/ folder
4. Verify risk ratings match the market benchmarks in skill.md
5. Confirm a "Reviewed & Acceptable" section appears
```

## Architecture

The skill follows a **progressive disclosure** loading model:

1. **Metadata** (`name` + `description` from YAML frontmatter) — always in the AI tool's context, approximately 100 tokens
2. **skill.md body** — loaded when the tool determines the skill is relevant to the user's request
3. **examples/** — not loaded automatically; consulted by humans as a quality reference

The skill's output is structured markdown only — no XML tags, no JSON in the contract review itself. The companion project `legal-redline-tools` can consume the skill's redline suggestions to produce Word/PDF deliverables.

## Key Conventions

### Output Format (strictly enforced)

Every contract review must follow this exact sequence:
1. Header block (document type, party position, counterparty, overall risk, document status)
2. Pre-Signing Alerts (blank fields, missing exhibits)
3. Executive Summary
4. Key Terms table (with section references)
5. Red Flags Quick Scan table
6. Risk Analysis sections: Critical / Important / Acceptable
7. Missing Provisions (with suggested language)
8. Internal Consistency Issues (optional, when present)
9. Negotiation Priority matrix

The "Reviewed & Acceptable" section under Risk Analysis is mandatory — every review must affirm what is acceptable, not only flag problems.

### Position-Aware Analysis (core differentiator)

Before reviewing, the skill asks which party the user represents. Risk assessment is always relative to that position:
- Customer reviewing vendor agreement → flags vendor-favorable terms
- Vendor reviewing own template → flags customer-favorable terms
- Buyer in M&A → flags seller-favorable terms
- Seller in M&A → flags buyer-favorable terms
- Receiving party in NDA → flags disclosing party-favorable terms

### Market Benchmark Thresholds (do not change without strong justification)

Thresholds are calibrated from real negotiation outcomes. The key ones:

| Provision | Standard | Yellow Flag | Red Flag |
|-----------|----------|-------------|----------|
| Liability cap | 12 months' fees | 6-11 months | <6 months |
| Non-compete duration | 1-2 years | 3-4 years | 5+ years |
| Auto-renewal notice | 90+ days | 60-89 days | <60 days |
| SLA uptime | 99.9% with credits | 99.5% | No SLA |
| Data export | 90 days, standard format | 30 days | None |

If you change thresholds in `skill.md`, update the example files accordingly — they must stay consistent with the thresholds to remain valid as a quality baseline.

### CUAD Category Count

`skill.md` covers 41 original CUAD categories (from the Atticus/NeurIPS dataset) plus ~43 extensions, totaling ~84 risk categories. The README badge states "41 Categories" (the original CUAD count). If the extended count is ever surfaced in marketing copy, update the badge. Do not change the badge to 84 without understanding this distinction.

### Versioning

Follows semver. When bumping the version:
1. Update `version:` in `skill.md` YAML frontmatter
2. Update the version badge in `README.md` — these two are coupled and must match
3. Add an entry to `CHANGELOG.md`

Never update only one of the two version references.

## How AI Assistants Should Work in This Repo

**Do not introduce:**
- Source code of any kind
- Build systems, package managers, or CI pipelines
- Test frameworks or automated test suites
- New files beyond `skill.md`, `README.md`, `CHANGELOG.md`, and `examples/`

**The only files that should ever be edited:**
- `skill.md` — to improve skill behavior, fix instructions, update benchmarks
- `README.md` — to update docs, installation instructions, or version badge
- `CHANGELOG.md` — to document every change
- `examples/*.md` — to update sample outputs when behavior or thresholds change
- `examples/demo.png` — only if a new screenshot is needed

**When editing skill.md:**
- The version field is in the YAML frontmatter at the top of the file (currently `3.0.0`)
- The `description` field must remain specific enough for AI tools to auto-discover the skill
- Market benchmark thresholds are intentional and calibrated — do not change them casually
- The guardrails section at the bottom is non-negotiable: the skill must never give legal advice, must never hallucinate, and must always include a "Reviewed & Acceptable" section

**Quality validation is always manual.** There is no automated test suite. To verify a change works correctly: install the skill, paste a representative contract, invoke the review, and compare the output structure and risk ratings against the `examples/` folder.

**Companion project distinction.** The `legal-redline-tools` repo (https://github.com/evolsb/legal-redline-tools) contains Python tooling for generating Word/PDF deliverables from this skill's output. It is a separate repository. Do not add Python scripts or any executable code to this repository.
