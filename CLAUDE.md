# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

A standalone contract review skill following the open [Agent Skills standard](https://agentskills.io). Position-aware analysis grounded in the CUAD dataset (41 risk categories from 510 real contracts), ContractEval benchmarks, and LegalBench. Compatible with Claude Code, OpenAI Codex, Cursor, GitHub Copilot, Gemini CLI, and 20+ other tools.

## Structure

```
claude-legal-skill/
├── skill.md          # The entire skill — all instructions, checklists, and benchmarks
├── examples/         # Sample outputs for NDA, M&A, SaaS, and balanced agreements
└── README.md
```

No build step, no scripts, no external dependencies. The entire skill is `skill.md`.

## Installation

```bash
# Claude Code
git clone https://github.com/evolsb/claude-legal-skill ~/.claude/skills/contract-review

# Development (symlink so edits to the repo are reflected immediately)
git clone https://github.com/evolsb/claude-legal-skill ~/Developer/claude-legal-skill
ln -s ~/Developer/claude-legal-skill ~/.claude/skills/contract-review
```

## Skill Design

**Position-aware** — the user must specify which party they are. The skill adjusts what it flags as risky based on that position (e.g., vendor-favorable terms are a risk for the customer, not the vendor). If position isn't provided, the skill asks.

**Document-type dispatch** — the skill detects document type (NDA, SaaS/MSA, Payment/Merchant, M&A, Finder/Broker) and activates the relevant specialized checklist from within `skill.md`. Each checklist covers the key provisions that matter most for that contract type.

**Red flags first** — before deep analysis, the skill scans 12 instant danger signs (uncapped indemnification, liability cap < 6 months, unilateral amendment rights, offshore jurisdiction, etc.).

**Market benchmarks** — risk ratings reference a benchmarks table in `skill.md`. The thresholds are based on real contract norms (e.g., liability cap: 12+ months = standard, 6–11 months = yellow, < 6 months = red).

**Structured output** — the skill produces Markdown in this order: pre-signing alerts → executive summary → key terms table → red flags scan → risk analysis (🔴/🟡/🟢) → missing provisions → internal consistency checks → negotiation priority list.

## Integration with legal-redline-tools

The skill outputs structured JSON redlines. To generate tracked-changes Word documents and PDFs:

```bash
pip install git+https://github.com/evolsb/legal-redline-tools.git
legal-redline apply contract.docx redlined.docx --from-json redlines.json --pdf redline.pdf
```

## Usage Examples

```
Review this NDA - I'm the receiving party
Analyze the indemnification in this MSA - I'm the vendor
Review this acquisition agreement - I'm the seller
Check this merchant agreement - what's my chargeback exposure?
```

## Accuracy Note

Based on ContractEval benchmarks, Claude achieves F1 ~0.62 on clause extraction. Best for first-pass review and issue flagging — not a replacement for attorney review on material deals. The skill always includes a disclaimer to that effect in its output.
