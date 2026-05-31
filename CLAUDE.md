# CLAUDE.md — Contract Review Agent Skill

This file provides guidance for AI assistants (Claude and others) working in this repository.

## Overview

`claude-legal-skill` (published as `contract-review`) is an open Agent Skills-compatible skill for AI-powered legal contract review. It is a **prompt-based skill** — a single `skill.md` file containing structured instructions that guide any compatible AI coding assistant (Claude Code, OpenAI Codex, Cursor, GitHub Copilot, Gemini CLI, and 26+ others) to perform sophisticated contract analysis.

The skill is grounded in the [CUAD dataset](https://github.com/TheAtticusProject/cuad) (41 legal risk categories from 510 real contracts), ContractEval benchmarks, and LegalBench. It performs position-aware review (customer vs. vendor, buyer vs. seller, etc.) and outputs market-benchmarked analysis with specific redline language.

**Current version:** 3.0.0  
**License:** MIT

> **Not legal advice**: Always have material terms reviewed by qualified legal counsel. Best used for first-pass review and issue flagging. Claude achieves ~F1 0.62 on CUAD clause extraction benchmarks.

---

## Repository Structure

```
claude-legal-skill/
├── skill.md          # Core skill definition — the primary artifact of this repo
├── examples/
│   ├── nda-review.md             # Sample output: NDA review
│   ├── saas-agreement-review.md  # Sample output: SaaS/MSA review
│   ├── ma-agreement-review.md    # Sample output: M&A agreement review
│   ├── balanced-agreement.md     # Sample output: balanced/acceptable agreement
│   └── demo.png                  # Screenshot of skill output in Claude Code
├── README.md         # Project overview and installation instructions
├── CHANGELOG.md      # Version history
├── LICENSE           # MIT license
└── .gitignore
```

---

## Key Files

### `skill.md`
The single most important file in the repository. This is the complete skill definition consumed by AI coding assistants. It contains:

- **Skill metadata** (YAML front matter): `name`, `description`, `version`
- **Activation triggers**: conditions under which the skill should activate
- **Step 1 — Pre-Review Checklist**: instructs the AI to flag blank fields, missing exhibits, signature status, and page completeness before analysis
- **Step 2 — Document Type & User Position**: instructs the AI to identify who the user is (customer, vendor, buyer, seller, receiving/disclosing party) and adjust risk framing accordingly, accounting for power dynamics
- **Output Format**: full Markdown output specification with an annotated example covering all output sections
- **Red Flags Quick Scan**: 12 danger signs to check first (e.g., liability cap < 6 months, uncapped indemnification, unilateral amendment rights)
- **Document Type Checklists**: specialized checklists for NDA, SaaS/MSA, Payment/Merchant, M&A, and Finder/Broker agreements
- **Risk Categories**: extended CUAD 41-category taxonomy plus additional categories covering suspension rights, price escalation, chargeback liability, unilateral amendment, data export, etc.
- **Market Standard Benchmarks**: quantitative thresholds for key provisions (liability cap, non-compete duration, auto-renewal notice, etc.) with Standard / Yellow Flag / Red Flag tiers
- **Negotiability Guide**: High / Medium / Low / None ratings with power-dynamic factors
- **Jurisdiction Notes**: state-by-state non-compete enforceability, choice-of-law implications, arbitration venue guidance
- **Guardrails**: instructions preventing hallucination, ensuring uncertainty is expressed, and requiring attorney review recommendations

### `examples/`
Four complete sample outputs demonstrating real skill behavior:
- `nda-review.md` — NDA review from the receiving party's perspective
- `saas-agreement-review.md` — SaaS subscription agreement review from the customer's perspective
- `ma-agreement-review.md` — M&A acquisition agreement review from the seller's perspective
- `balanced-agreement.md` — Example of an agreement that reviews as largely acceptable
- `demo.png` — Screenshot showing the skill in action within Claude Code

---

## Installation

This skill follows the open [Agent Skills standard](https://agentskills.io) and installs by cloning to the tool's skills directory.

### Claude Code

```bash
git clone https://github.com/evolsb/claude-legal-skill ~/.claude/skills/contract-review
```

### OpenAI Codex

```bash
git clone https://github.com/evolsb/claude-legal-skill ~/.codex/skills/contract-review
```

### Other Agent Skills-compatible tools

Clone to your tool's configured skills directory.

### Development (Symlink for live editing)

```bash
git clone https://github.com/evolsb/claude-legal-skill ~/Developer/claude-legal-skill
ln -s ~/Developer/claude-legal-skill ~/.claude/skills/contract-review
```

The symlink approach means edits to `skill.md` take effect immediately in Claude Code without re-cloning.

---

## Usage

Once installed, invoke the skill by describing what you want reviewed in natural language. The skill activates when:
- The user mentions "review contract", "analyze agreement", "check this contract"
- The user uploads or references a PDF/DOCX legal document
- The user asks about specific clauses, risks, or terms

### Example Prompts

```
Review this NDA - I'm the receiving party

Analyze the indemnification in this MSA - I'm the vendor

What are the termination provisions? I'm the customer.

Review this acquisition agreement - I'm the seller

Check this merchant agreement - what's my chargeback exposure?
```

### Output Structure

Every review produces a structured Markdown document containing:

1. **Header block** — Document type, user position, counterparty, risk level, document status
2. **Pre-Signing Alerts** — Blank fields, missing exhibits flagged prominently
3. **Executive Summary** — 2–4 sentence overview
4. **Key Terms table** — Term, value, and section reference
5. **Red Flags Quick Scan** — Table of 12 danger signs with found/not-found status
6. **Risk Analysis** — Three tiers:
   - `Critical` (red) — Issues requiring immediate attention with redline language
   - `Important` (yellow) — Issues worth negotiating with negotiability rating
   - `Reviewed & Acceptable` (green) — Items confirmed as acceptable
7. **Missing Provisions** — Provisions absent but recommended, with suggested language
8. **Internal Consistency Issues** — Broken cross-references, undefined terms
9. **Negotiation Priority table** — Ranked list of issues with ask and negotiability
10. **Disclaimer** footer

### Generating Deliverables

The skill outputs structured analysis in Markdown. To produce tracked-changes Word documents and redline PDFs, pair with [legal-redline-tools](https://github.com/evolsb/legal-redline-tools):

```bash
pip install git+https://github.com/evolsb/legal-redline-tools.git

# After skill generates redlines.json:
legal-redline apply contract.docx redlined.docx \
    --from-json redlines.json \
    --pdf redline.pdf \
    --memo-pdf internal-memo.pdf
```

---

## How the Skill Works

### Position-Aware Analysis
The skill's most important feature. Before analyzing content, it asks (if not stated) which party the user represents. This changes what is flagged as risky:
- Customer reviewing vendor agreement → vendor-favorable terms are flagged
- Vendor reviewing own template → customer-favorable terms are flagged
- Buyer in M&A → seller-favorable terms are flagged
- Receiving party in NDA → disclosing party-favorable terms are flagged

Power dynamics also affect the analysis (startup vs. enterprise, standard form vs. negotiated, regulated industry).

### Risk Classification
Risks are classified into three severity levels:
- `Critical` — Issues with material financial or legal exposure
- `Important` — Suboptimal terms worth addressing
- `Acceptable` — Terms confirmed as within market norms

### Market Standard Benchmarks
The skill compares contract terms against quantitative industry thresholds:

| Provision | Standard | Yellow Flag | Red Flag |
|-----------|----------|-------------|----------|
| Liability cap | 12 months' fees | 6–11 months | <6 months |
| Auto-renewal notice | 90+ days | 60–89 days | <60 days |
| Non-compete duration | 1–2 years | 3–4 years | 5+ years |
| Rep survival (M&A) | 12–18 months | 24–30 months | 36+ months |
| SLA uptime | 99.9% with credits | 99.5% | No SLA |
| Data export | 90 days, standard format | 30 days | None |

### Document Type Checklists
Specialized checklists activate based on document type:
- **NDA** — direction, definition scope, term, residuals, standstill, destruction certification
- **SaaS/MSA** — SLA, data export, suspension rights, price caps, auto-renewal notice
- **Payment/Merchant** — reserves, chargebacks, network rules, auto-debit, PCI compliance
- **M&A** — earnouts, escrow, rep survival, sandbagging, working capital, employment comp
- **Finder/Broker** — fee tails, covered buyer definitions, joint representation, exclusivity

### Risk Category Coverage (CUAD 41 + Extensions)
The skill covers all 41 CUAD risk categories plus extensions:
- Document basics (parties, dates, renewal, blank fields)
- Term and termination (cure periods, suspension rights, survival)
- Assignment and control (change of control, asymmetric assignment)
- Financial terms (price escalation, reserves, auto-debit, audit rights)
- Liability and risk (liability cap, uncapped carve-outs, warranty disclaimers, chargeback)
- IP and confidentiality (residuals clause, feedback ownership, non-solicitation)
- Dispute resolution (class action waiver, offshore jurisdiction flags)
- Special provisions (data export, uptime SLA, unilateral amendment rights)

---

## Examples

See the `examples/` directory for full sample outputs:

- **`examples/nda-review.md`** — Demonstrates NDA-specific checklist, one-way vs. mutual analysis, receiving party framing
- **`examples/saas-agreement-review.md`** — Demonstrates SaaS/MSA checklist, liability cap benchmarking, data export gap detection
- **`examples/ma-agreement-review.md`** — Demonstrates M&A checklist, earnout mechanics, rep survival analysis
- **`examples/balanced-agreement.md`** — Demonstrates the `Reviewed & Acceptable` section for a well-balanced agreement
- **`examples/demo.png`** — Screenshot of actual output in Claude Code

---

## Version History

See `CHANGELOG.md` for full details. Key versions:

| Version | Date | Highlights |
|---------|------|------------|
| 3.0.0 | 2026-01-26 | Multi-platform Agent Skills support, complete skill.md rewrite, power-dynamic analysis, full example outputs |
| 2.0.0 | 2026-01-26 | Position-aware review, full CUAD 41-category coverage, document-type checklists, market benchmarks, jurisdiction awareness |
| 1.0.0 | 2026-01-26 | Initial release, basic contract review, CUAD integration |

---

## Development and Contribution

### Making Changes to the Skill

All substantive logic lives in `skill.md`. When editing:
- **Output format changes**: update the example output block in the `## Output Format` section
- **New red flags**: add to the `## Red Flags Quick Scan` table and explain in `## Guardrails`
- **New document type**: add a checklist section under `## Document Type Checklists` and add detection logic to Step 2
- **New risk category**: add to the appropriate group in `## Risk Categories (CUAD 41 + Extensions)`
- **Benchmark changes**: update the `## Market Standard Benchmarks` table with source/rationale

### Testing Changes

There is no automated test suite. Test by:
1. Using the symlink dev setup: `ln -s ~/Developer/claude-legal-skill ~/.claude/skills/contract-review`
2. Running the skill against contracts in `examples/` and comparing output to expected results
3. Checking that edge cases are handled: missing exhibits, already-executed contracts, blank fields, contracts without governing law

### Adding Examples

1. Run the skill against a representative contract type
2. Save the output as `examples/<type>-review.md`
3. Verify the output demonstrates the skill's key features for that contract type
4. Link from `README.md`

### Changelog Convention

Follow the format in `CHANGELOG.md`:
- Group changes under `Added`, `Changed`, `Fixed`, `Removed`
- Use semantic versioning (`MAJOR.MINOR.PATCH`)
- Include date in `[VERSION] - YYYY-MM-DD` format

---

## Conventions

- **The skill is the product**: `skill.md` is the primary deliverable; everything else supports it
- **Markdown only**: skill output must be Markdown — the skill explicitly prohibits XML tags in output
- **No hallucination**: the skill's guardrails require the AI to only reference text actually present in the reviewed document; never invent clauses
- **Always show acceptable items**: every review must include a `Reviewed & Acceptable` section to avoid false-alarm fatigue
- **Always recommend counsel**: every review ends with a disclaimer recommending attorney review for material terms
- **Express uncertainty**: when interpretation is unclear, the skill instructs the AI to say so explicitly
- **Document status matters**: the skill treats already-executed contracts as informational reviews, not negotiation prep
- **US law default**: analysis defaults to US jurisdiction; international variations are flagged but not deeply analyzed

---

## Guardrails (for AI Assistants)

When using this skill, always observe these constraints defined in `skill.md`:

1. **Not legal advice** — Recommend attorney review for any material terms
2. **Not tax advice** — Flag tax implications but do not opine on them
3. **Jurisdiction matters** — Note when enforceability varies by jurisdiction
4. **Express uncertainty** — When clause interpretation is ambiguous, say so
5. **No hallucination** — Only reference text actually present in the document being reviewed
6. **Show acceptable items** — Always include what was reviewed and found acceptable, not just problems
7. **Document status** — If the contract is already executed, frame the review as informational

---

## Related Projects

- [legal-redline-tools](https://github.com/evolsb/legal-redline-tools) — generates tracked-changes `.docx`, redline PDFs, and negotiation memos from skill output
- [ai-legal-claude](https://github.com/mdelmaschio-cpu/ai-legal-claude) — broader legal assistant suite with 14 skills, 5 agents, and PDF report generation for Claude Code
- [CUAD Dataset](https://github.com/TheAtticusProject/cuad) — the academic foundation (Atticus Project, NeurIPS 2021)
- [Agent Skills Standard](https://agentskills.io) — the open standard this skill implements
