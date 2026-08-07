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

The `description` field drives auto-discovery. The `name` field must match the directory name where users install the skill.

**Skill body** (~430 lines of structured markdown) covering:
- When to activate (trigger phrases)
- Step 1: Pre-review checklist
- Step 2: Position identification (customer/vendor/buyer/seller/etc.)
- Output format spec with a full annotated example
- Red flags quick scan table (12 danger signs)
- Document-type checklists: NDA, SaaS/MSA, Payment/Merchant, M&A, Finder/Broker
- Risk categories: 41 original CUAD categories + ~43 extensions (84 total)
- Market standard benchmarks table with numeric thresholds
- Negotiability guide (High/Medium/Low/None ratings)
- Jurisdiction notes
- Guardrails (not legal advice, no hallucination, always show acceptable terms)

### README.md — User-Facing Documentation

Installation guide, feature overview, usage examples. The README version badge must always match the version in `skill.md` YAML frontmatter — they are coupled and must be updated together.

### examples/ — Quality Baseline

Four full sample outputs serving as the authoritative reference for correct skill behavior:

| File | Contract Type | User Position | Risk Level | Key Purpose |
|------|--------------|---------------|------------|-------------|
| `nda-review.md` | NDA | Receiving party | Medium | Standard NDA analysis |
| `saas-agreement-review.md` | SaaS | Customer | High | High-risk vendor agreement |
| `ma-agreement-review.md` | M&A acquisition | Seller | High | Complex deal review |
| `balanced-agreement.md` | SaaS | Customer | Low | **Demonstrates affirming acceptable terms** |

The `balanced-agreement.md` example is the most important one to preserve — it demonstrates the mandatory "Reviewed & Acceptable" section.

## Development Environment

No setup required. This is a pure markdown repository with no runtime dependencies, no package manager, no virtual environment, and no build step.

The `.gitignore` excludes:
- `.venv/`, `venv/`, `env/` — Python environments (for local tooling only)
- `.zdai/` — Zuva AI config containing API tokens
- `contracts/`, `*.pdf`, `*.docx` — actual contract documents (privacy)

**Never commit actual contract documents.**

## Commands

There are no build, test, lint, or run commands.

**Installation (end users):**
```bash
# Claude Code
git clone https://github.com/evolsb/claude-legal-skill ~/.claude/skills/contract-review

# OpenAI Codex
git clone https://github.com/evolsb/claude-legal-skill ~/.codex/skills/contract-review
```

**Quality validation (manual — the only quality gate):**
```
1. Paste a real contract into Claude Code
2. Trigger the skill: "Review this NDA - I'm the receiving party"
3. Compare output structure against examples/ folder
4. Verify risk ratings match the market benchmarks in skill.md
5. Confirm a "Reviewed & Acceptable" section appears
```

## Key Conventions

### Output Format (strictly enforced)

Every contract review must follow this exact sequence:
1. Header block (document type, party position, counterparty, overall risk, document status)
2. Pre-Signing Alerts
3. Executive Summary
4. Key Terms table (with section references)
5. Red Flags Quick Scan table
6. Risk Analysis sections: Critical / Important / Acceptable
7. Missing Provisions
8. Negotiation Priority matrix

The "Reviewed & Acceptable" section under Risk Analysis is mandatory.

### Position-Aware Analysis (core differentiator)

Before reviewing, the skill asks which party the user represents. Risk assessment is always relative to that position.

### Market Benchmark Thresholds (do not change without strong justification)

| Provision | Standard | Yellow Flag | Red Flag |
|-----------|----------|-------------|----------|
| Liability cap | 12 months' fees | 6-11 months | <6 months |
| Non-compete duration | 1-2 years | 3-4 years | 5+ years |
| Auto-renewal notice | 90+ days | 60-89 days | <60 days |
| SLA uptime | 99.9% with credits | 99.5% | No SLA |
| Data export | 90 days, standard format | 30 days | None |

If you change thresholds in `skill.md`, update the example files accordingly.

### Versioning

When bumping the version:
1. Update `version:` in `skill.md` YAML frontmatter
2. Update the version badge in `README.md` — these two must match
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
- `README.md` — to update docs or version badge
- `CHANGELOG.md` — to document every change
- `examples/*.md` — to update sample outputs when behavior or thresholds change

**When editing skill.md:**
- The `description` field must remain specific enough for AI tools to auto-discover the skill
- Market benchmark thresholds are intentional and calibrated — do not change them casually
- The guardrails section is non-negotiable: never give legal advice, never hallucinate, always include a "Reviewed & Acceptable" section

**Quality validation is always manual.** There is no automated test suite.

**Companion project distinction.** The `legal-redline-tools` repo contains Python tooling for generating Word/PDF deliverables from this skill's output. It is a separate repository — do not add code to this repository.
