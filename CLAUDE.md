# CLAUDE.md — claude-legal-skill

## Repository Overview

This repository is an **Agent Skills-compatible contract review skill** for AI coding assistants (Claude Code, OpenAI Codex, Cursor, GitHub Copilot, Gemini CLI, and 26+ others). It provides AI-powered legal contract analysis grounded in the [CUAD dataset](https://github.com/TheAtticusProject/cuad) (41 risk categories from 510 real contracts), ContractEval benchmarks, and LegalBench.

**What it does:** Analyzes legal contracts (NDAs, SaaS agreements, M&A docs, merchant agreements, finder/broker agreements) and produces structured reviews with risk ratings, market benchmarks, specific redline language, and negotiation priorities.

**What it is not:** It is not a code project. There are no scripts, tests, or build systems. The entire skill is defined in a single Markdown file (`skill.md`) that AI assistants read and follow as instructions.

---

## Directory Structure

```
claude-legal-skill/
├── skill.md              # The skill itself — AI assistant instructions
├── README.md             # User-facing documentation and install guide
├── CHANGELOG.md          # Version history (current: 3.0.0)
├── LICENSE               # MIT
├── .gitignore            # Excludes contracts/, *.pdf, *.docx, .env, etc.
└── examples/
    ├── nda-review.md           # NDA example (receiving party perspective)
    ├── saas-agreement-review.md # SaaS example (customer, high-risk agreement)
    ├── ma-agreement-review.md   # M&A example (seller perspective)
    ├── balanced-agreement.md    # Low-risk SaaS example (how to report good terms)
    └── demo.png                 # Screenshot used in README
```

---

## Key Files

### `skill.md` — The Core Skill

This is the only functional file in the repository. It contains:

1. **YAML front matter** — name, description, version (consumed by Agent Skills-compatible tools for discovery)
2. **Activation triggers** — phrases that should invoke the skill
3. **Step 1: Pre-Review Checklist** — blank fields, missing exhibits, signature status, page truncation
4. **Step 2: Document type and position identification** — prompts AI to ask "which party are you?"
5. **Output format specification** — Markdown only, no XML tags; includes a complete annotated example output
6. **Red Flags Quick Scan table** — 12 instant danger signs to check first
7. **Document Type Checklists** — NDA, SaaS/MSA, Payment/Merchant, M&A, Finder/Broker
8. **Risk Categories (CUAD 41 + extensions)** — full taxonomy organized by domain
9. **Market Standard Benchmarks** — numeric thresholds with Standard / Yellow Flag / Red Flag columns
10. **Negotiability Guide** — High/Medium/Low/None ratings with power dynamic factors
11. **Jurisdiction Notes** — non-compete state law, choice of law implications, arbitration venues
12. **Guardrails** — no hallucination, not legal advice, always include "Reviewed & Acceptable" section

### `examples/` — Reference Outputs

Each example file follows the same structure: a `## Prompt` block showing what the user typed, an `## Example Output` block showing ideal skill output, and a `## [Type] Red Flags` or lesson section with takeaways.

| File | Document Type | User Position | Risk Level |
|------|--------------|---------------|------------|
| `nda-review.md` | Mutual NDA | Receiving party | Medium |
| `saas-agreement-review.md` | SaaS subscription | Customer | High |
| `ma-agreement-review.md` | Asset purchase | Seller | High |
| `balanced-agreement.md` | SaaS subscription | Customer | Low (balanced) |

The `balanced-agreement.md` example is particularly important — it shows that the skill should **not** over-flag acceptable terms and should present a meaningful "Reviewed & Acceptable" table.

---

## Skill Conventions and Patterns

### Output Structure (Always in This Order)

```markdown
# Contract Review: [Document Name]

**Document Type:** ...
**Your Position:** ...
**Counterparty:** ...
**Risk Level:** [🔴 High | 🟡 Medium | 🟢 Low]
**Document Status:** Draft / Executed

## ⚠️ Pre-Signing Alerts       # Only if blank fields or missing exhibits
## Executive Summary            # 2-3 sentence overview
## Key Terms                    # Table: Term | Value | Location
## Red Flags (Quick Scan)       # Table: Flag | Found | Location
## Risk Analysis                # Grouped 🔴 Critical / 🟡 Important / 🟢 Minor
## Reviewed & Acceptable        # Table of terms that are fine
## Missing Provisions           # Table + suggested language
## Internal Consistency Issues  # Cross-references, undefined terms
## Negotiation Priority         # Ranked table: # | Issue | Ask | Negotiability
```

### Severity Ratings

- **🔴 Critical** — Must address before signing (e.g., liability cap < 6 months, uncapped indemnification)
- **🟡 Important** — Should negotiate (e.g., asymmetric termination, missing SLA)
- **🟢 Acceptable / Minor** — Acceptable as-is or low-priority optional ask

### Redline Format

Every flagged issue should include:
- Quoted original contract language
- Issue explanation
- **Redline:** specific replacement text (e.g., `"three (3) months" → "twelve (12) months"`)
- **Fallback:** compromise position when full ask is unlikely

### Position-Aware Analysis

The skill adjusts what it flags based on user position:
- Customer reviewing vendor agreement → flag vendor-favorable terms
- Vendor reviewing own template → flag customer-favorable terms
- Buyer in M&A → flag seller-favorable terms
- Seller in M&A → flag buyer-favorable terms
- Receiving party in NDA → flag disclosing-party-favorable terms

If position is not stated, the skill should ask before proceeding.

### Market Benchmarks (Key Thresholds)

| Provision | Standard | Yellow Flag | Red Flag |
|-----------|----------|-------------|----------|
| Liability cap | 12 months' fees | 6-11 months | <6 months |
| Auto-renewal notice | 90+ days | 60-89 days | <60 days |
| Non-compete duration | 1-2 years | 3-4 years | 5+ years |
| Rep survival (M&A) | 12-18 months | 24-30 months | 36+ months |
| Data export window | 90 days | 30 days | None |
| Cure period | 30 days | 15 days | None |

---

## Development Workflow

There is no build system, test suite, or CI pipeline. The workflow is entirely content-based:

### Making Changes to the Skill

1. Edit `skill.md` directly — this is the sole source of truth for skill behavior
2. Test by running the skill in Claude Code or another compatible assistant against a sample contract
3. Update `CHANGELOG.md` when making significant changes
4. Bump the version in `skill.md` front matter to match

### Adding Examples

1. Create a new `.md` file in `examples/`
2. Follow the format: `## Prompt`, `## Example Output` (with fenced markdown block), and a `## Lessons` or checklist section
3. Reference the example in `README.md` if appropriate

### Installation for Development

```bash
git clone https://github.com/evolsb/claude-legal-skill ~/Developer/claude-legal-skill
ln -s ~/Developer/claude-legal-skill ~/.claude/skills/contract-review
```

Changes to `skill.md` take effect immediately in the next Claude Code session (no restart needed).

### Git Conventions

- Commits follow the pattern: short imperative subject, e.g., `Add legal-redline-tools integration (#10)`
- Commit messages reference GitHub issue numbers when applicable
- No pre-commit hooks or linters

---

## Integration: legal-redline-tools

The skill outputs structured JSON redlines. To convert these into lawyer-ready Word documents and PDFs, pair with the companion project:

```bash
pip install git+https://github.com/evolsb/legal-redline-tools.git
legal-redline apply contract.docx redlined.docx \
    --from-json redlines.json \
    --pdf redline.pdf \
    --memo-pdf internal-memo.pdf
```

---

## Notes for AI Assistants

- **This repo has no code to run.** Do not attempt to execute, test, or lint anything.
- **`skill.md` is the product.** All substantive changes belong there. README.md and examples are supporting documentation.
- **`.gitignore` excludes contract files** (`contracts/`, `*.pdf`, `*.docx`). Never commit actual contract documents.
- **The skill must avoid hallucination.** A core guardrail in `skill.md` is "only reference text actually in the document." When improving the skill, preserve this constraint explicitly.
- **Always include "Reviewed & Acceptable."** The balanced-agreement example illustrates why: the skill loses credibility if it flags everything. Preserving this section is important.
- **Position matters.** Changes to risk logic must be validated against both sides of a transaction type (e.g., if adjusting indemnification guidance, check both customer and vendor perspectives).
- **Market benchmark numbers are deliberate.** The thresholds in the benchmarks table (12-month liability cap, 90-day auto-renewal notice, etc.) were derived from real negotiation outcomes. Don't change them without a documented rationale.
- **Version is in two places:** `skill.md` YAML front matter and `CHANGELOG.md`. Keep them in sync.
- **Agent Skills compatibility** requires the YAML front matter in `skill.md` (`name`, `description`, `version`) to remain valid and accurate for tool discovery across 26+ platforms.
