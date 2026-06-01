# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

**claude-legal-skill** is a single-skill Claude Code package for contract review. It uses CUAD (Contract Understanding Atticus Dataset) risk detection, market benchmarks, and generates lawyer-ready redlines from any contract. Compatible with 26+ agent tools.

This is a minimal repository — one skill, no build system, no test suite, no application code.

## Repository Layout

```
claude-legal-skill/
├── llm-council/
│   └── SKILL.md          # The skill itself
├── examples/             # Sample outputs and demos
├── README.md
├── LICENSE
├── CHANGELOG.md
└── .gitignore
```

Wait — the skill directory is named `llm-council/` in the filesystem but is the legal skill. The primary skill file is at `llm-council/SKILL.md`.

## Installation

```bash
# Clone and install
git clone https://github.com/<author>/claude-legal-skill
cp -r claude-legal-skill ~/.claude/skills/
```

Or install via Claude Code plugin marketplace if published there.

## What the Skill Does

- **CUAD risk detection**: Identifies high-risk clauses from the 41 CUAD categories (uncapped liability, auto-renewal traps, IP assignment, etc.)
- **Market benchmarking**: Compares clause terms against market norms for the contract type
- **Redlines**: Generates specific replacement language for problematic clauses
- **Lawyer-ready output**: Structured output format suitable for review by counsel

## Key Conventions

- This is a single-skill package — do not add more skills to the root directory
- The skill is self-contained in `SKILL.md`; all logic is in the prompt, not in external scripts
- `examples/` contains sample inputs and outputs — update these when the skill behavior changes
- No dependencies required beyond Claude Code itself
- Always include a disclaimer that output is not legal advice and should be reviewed by a licensed attorney

## CHANGELOG

Update `CHANGELOG.md` when making changes to the skill's behavior or output format.
