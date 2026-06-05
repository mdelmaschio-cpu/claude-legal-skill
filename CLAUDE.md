# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **documentation-only AI skill** — no source code, no build system, no dependencies. It defines a contract review skill for Claude Code and 26+ compatible AI tools (Cursor, GitHub Copilot, Gemini CLI, etc.).

The skill is distributed by users cloning it into their tool's skills directory (e.g., `~/.claude/skills/contract-review`). Claude reads `skill.md` at runtime and follows its instructions when the user uploads a contract.

## Architecture

There are two files that define behavior:

- **`skill.md`** — The authoritative instruction set. YAML front matter (`name`, `description`, `version`) followed by ~400 lines of structured markdown covering: activation triggers, pre-review checklists, output format spec, red flag patterns, document-type checklists (NDA, SaaS/MSA, Payment, M&A, Finder/Broker), CUAD risk categories, market benchmarks, negotiability ratings, and guardrails.
- **`README.md`** — User-facing marketing and installation docs. Describes what the skill does, installation steps per tool, limitations, and links to the companion project `legal-redline-tools`.

The `examples/` folder contains four full sample outputs showing expected behavior:
- `nda-review.md` — NDA, receiving party, medium risk
- `saas-agreement-review.md` — SaaS, customer position, high risk
- `ma-agreement-review.md` — M&A acquisition, seller position, high risk  
- `balanced-agreement.md` — SaaS, customer position, low risk (demonstrates not over-flagging)

## Key Conventions

**Output format is strictly markdown** — no XML tags, no JSON. Every review must begin with a header block (document type, party position, counterparty, overall risk, document status), then flow through: Pre-Signing Alerts → Executive Summary → Key Terms table → Red Flags Quick Scan → Risk Analysis (🔴 Critical / 🟡 Important / 🟢 Acceptable) → Missing Provisions → Negotiation Priority matrix.

**Position-aware analysis is the core differentiator.** Before reviewing, the skill asks which party the user represents (customer/vendor, buyer/seller, employer/employee, etc.) and tailors every risk assessment to that position. The `balanced-agreement.md` example demonstrates that the skill must affirm acceptable terms — not just surface problems.

**Risk ratings follow market benchmarks** defined in `skill.md` under "Market Standard Benchmarks." Thresholds are specific (e.g., liability cap: Standard = 12 months fees, Yellow = 6–12 months, Red = <6 months). Changes to these thresholds are changes to the skill's judgment, so update the examples accordingly.

**CUAD coverage** is a stated quality claim (41 categories from the Atticus/NeurIPS dataset, extended to ~84 total in `skill.md`). Adding or removing risk categories should be reflected in the README badge count.

## Versioning

Follows semver. Bump `version` in both `skill.md` YAML front matter and `README.md` badge simultaneously. Document every change in `CHANGELOG.md`.

## No Tests or CI

There is no test suite. Quality validation is manual: paste a contract, run the skill, compare output against the example files and market benchmark table in `skill.md`.
