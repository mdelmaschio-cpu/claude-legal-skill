# claude-legal-skill

A single-file, zero-dependency Agent Skill for contract review built on the CUAD dataset (41 legal risk categories) with market benchmarks and position-aware analysis. Compatible with Claude Code, OpenAI Codex, Cursor, GitHub Copilot, and 26+ Agent Skills-compatible tools.

## Repository Purpose

A lightweight, portable contract review skill that works anywhere Agent Skills are supported. Unlike `ai-legal-claude`, this repo is a single `skill.md` file — no scripts, no installation steps, no dependencies.

## Structure

```
claude-legal-skill/
├── README.md
├── skill.md                   # Main skill definition (432 lines)
├── CHANGELOG.md               # Version history (current: v3.0.0)
├── LICENSE                    # MIT
└── examples/
    ├── demo.png
    ├── nda-review.md
    ├── saas-agreement-review.md
    ├── ma-agreement-review.md
    └── balanced-agreement.md
```

## How the Skill Works

`skill.md` is the entire skill. It is a structured markdown prompt that instructs the AI model to perform multi-phase contract review. There is no code to run.

**Phases:**
1. **Pre-review checklist** — detect blank fields, missing exhibits, signature status
2. **Document type classification** — NDA, SaaS/MSA, Payment, M&A, Finder, Employment, etc.
3. **Power dynamic assessment** — startup vs. enterprise, standard vs. negotiated form, regulated industry?
4. **Position-aware risk analysis** — 41 CUAD categories applied from the reviewer's position
5. **Market benchmark comparison** — flag terms that deviate from market standard
6. **Missing provisions** — identify absent protections with suggested language
7. **Internal consistency check** — catch broken cross-references and undefined terms

## CUAD-Based Risk Categories

The skill applies all 41 categories from the Contract Understanding Atticus Dataset (CUAD):
liability caps, auto-renewal, non-compete scope/duration, IP assignment, termination for convenience, governing law, indemnification breadth, audit rights, data handling, and more.

Each finding includes:
- **Negotiability rating**: High / Medium / Low
- **Market standard**: what's typical for this contract type
- **Current position**: what the contract actually says
- **Recommended language**: specific redline suggestion when negotiability is High

## Position Awareness

Specify your position for accurate risk assessment:

```
Review this contract from the perspective of the customer / buyer / employee / licensee
```

Without a specified position, the skill infers it from context clues (which party is named first, which party's obligations dominate, etc.). Risk levels flip based on position — an indemnification clause high-risk for one party is low-risk for the other.

## Trigger Phrases

The model invokes this skill when it sees:
- "review contract", "analyze agreement", "check this contract"
- "upload PDF/DOCX", "paste contract"
- "what are the risks in this agreement"
- "is this NDA standard", "is this SaaS agreement fair"

## Installation

```bash
# Claude Code
git clone https://github.com/mdelmaschio-cpu/claude-legal-skill ~/.claude/skills/claude-legal-skill

# Other tools: copy skill.md to the tool's skills directory
```

## Development Workflow

`skill.md` is the only file that matters. Edit it directly.

**Testing changes:**
1. Apply the updated skill to one of the example contracts in `examples/`
2. Compare output against the reference reviews (`nda-review.md`, `saas-agreement-review.md`, etc.)
3. Verify all 41 CUAD categories are covered in the output
4. Check that position-aware logic produces different risk assessments for customer vs. vendor

No build step, no test runner, no CI/CD. The examples directory is the regression suite — new features should produce output consistent with examples, and breaking changes should update the examples.

**Versioning:** Update `CHANGELOG.md` and the version in the `skill.md` frontmatter together. Use semantic versioning:
- **Patch**: Improved wording, additional trigger phrases, minor clarifications
- **Minor**: New document type checklist, new CUAD category coverage, new output section
- **Major**: Structural changes to output format, removal of features, breaking compatibility

## Key Conventions

- **Single file is the constraint**: Do not add scripts, dependencies, or multi-file structure. If the feature requires code, it belongs in `ai-legal-claude` instead.
- **CUAD completeness**: All 41 CUAD categories must be considered for every review (some will be N/A).
- **Market benchmarks must be grounded**: Only state "market standard is X" for provisions with well-established norms. Avoid invented benchmarks.
- **Limitations section**: `skill.md` includes an explicit limitations section — do not remove it. It must state: not legal advice, US law focus, context window constraints, no real-time data.
- **Markdown output only**: Output is formatted markdown, not XML or JSON. The skill is designed to pair with legal-redline-tools for tracked-changes .docx export.
- **Examples are living docs**: When updating the skill's output structure, update the example files to match.

## Scope vs. ai-legal-claude

| Feature | claude-legal-skill | ai-legal-claude |
|---------|-------------------|-----------------|
| Dependencies | None | Python + ReportLab |
| Installation | Single file | Bash installer |
| Skills | 1 | 14 |
| Agents | 0 | 5 parallel |
| PDF output | No | Yes |
| Document generation | No | Yes (NDA, ToS, etc.) |
| Tool compatibility | 26+ tools | Claude Code only |
| Best for | Quick review anywhere | Deep analysis + docs |
