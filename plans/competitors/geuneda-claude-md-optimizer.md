# geuneda/claude-md-optimizer

**URL:** https://github.com/geuneda/claude-md-optimizer
**Stars (2026-07-03):** 2
**Category:** Direct competitor
**License:** MIT

## What it is

A Claude Code skill that analyzes and scores CLAUDE.md files, built explicitly on external research (Anthropic best-practices docs, HumanLayer, Arize, Dometrain blog posts) and on findings from a separate MITM-proxy tool, [claude-inspector](https://github.com/kangraemin/claude-inspector), that captured real Claude Code API traffic.

## Problem framing (claude-inspector findings, cited verbatim)

- Every API request includes ALL CLAUDE.md content (~12KB overhead observed).
- Message history accumulates — after 30 turns, overhead exceeds 1MB.
- Injection order: global CLAUDE.md → global rules → project CLAUDE.md → project rules → Memory.
- Non-English content uses 30-50% more tokens than equivalent English.
- Skills persist in context until `/clear`.
- MCP tools are lazy-loaded (minimal cost when unused) — instruction files are not.

## What it does

- **Automated 0-100 scoring** with a detailed per-file breakdown.
- **Anti-pattern regex detection**: linter-territory content, vague directives ("follow best practices"), code bloat (inline snippets >5 lines instead of `file:line` refs), duplicate lines, narrative paragraphs that should be bullet lists, critical instructions buried mid-document (poor attention placement).
- **Progressive disclosure analysis** — checks for sub-doc tables, trigger conditions, content-tier classification (Essential/Reference/Redundant).
- **Attention-placement scoring** — verifies critical content sits at top/bottom, citing LLM "U-shaped attention" (content in the middle of a long context is attended to least).
- **Non-English content detection** — flags CJK content with an estimated token-overhead cost (30-50% savings claimed if converted to English).
- **Cross-file duplicate detection** — finds content repeated between global, project, and `.claude/rules/*.md` files.
- **Session cost estimation** — per-request token cost and cumulative cost over a session (e.g. "after 30 turns: ~216,000 tokens accumulated").
- **Injection-order awareness** — optimizes placement given Claude Code's actual config load order (global → project → rules → Memory), e.g. flags redundant restatement of a global prohibition in a project file that's "already well-attended via injection order."
- **Standalone Python analysis script** (`analyze_claude_md.py`), runnable outside any agent session, with a `--json` machine-readable mode.

## Numeric guardrails it enforces

| File | Max Lines | Optimal |
|---|---|---|
| Project `CLAUDE.md` | 150 | under 100 |
| User `~/.claude/CLAUDE.md` | 50 | under 30 |
| Individual `.claude/rules/*.md` | 30 | under 20 |
| `MEMORY.md` | 200 | under 100 |
| **Total across all sources** | 250 | under 180 |

## 8-priority optimization workflow

0. Language optimization (non-English → English)
1. Remove bloat (defaults, duplicates, linter content, vague directives)
2. Restructure (paragraphs → bullets, imperative form, inline code → file:line refs)
3. Progressive disclosure (Essential/Reference/Redundant tiers, extract with trigger conditions)
4. Attention placement (injection-order-aware)
5. Add essentials (project summary, commands, prohibitions, glossary)
6. Modularize (extract to `.claude/rules/` with glob patterns)
7. Future-proof (info-recording principles, recommend periodic `/clear`)

## Self-aware competitive positioning

Its own README includes a feature-comparison table against two other tools by name — **wrsmith108** and **daymade** — meaning the authors in this niche are already tracking each other. (A `daymade` CLAUDE.md tool could not be located via GitHub search as of 2026-07-03 — possibly renamed, unlisted, or removed since geuneda's README was written.) Claims made in that table: this skill is the only one of the three with automated 0-100 scoring, anti-pattern regex detection, non-English/token-overhead detection, cross-file duplicate detection, session-cost estimation, injection-order awareness, and a standalone analysis script.

## Cited (unverified, third-party) research claims

| Optimization | Claimed impact |
|---|---|
| Well-structured CLAUDE.md | 35% fewer corrections |
| Modular rules (`.claude/rules/`) | 25% less setup time |
| Token optimization | 60% context savings |
| Declared critical paths | 50% less file search time |
| Short code examples (≤5 lines) | 40% fewer corrections vs. long descriptions |
| Validation commands | 30% less back-and-forth |
| Repository-specific tuning | +10.87% accuracy on SWE-Bench |
| Progressive disclosure | 62% line reduction, 0% info loss |
| English conversion (CJK) | 30-50% token reduction/request |

## Install / distribution

Zip download from Releases, `git clone` into `~/.claude/skills/`, or manual copy.

## Gaps relative to TidyClaudeMD

- No repo-comprehension gate — scoring is purely a property of the instruction file's own text (regex/heuristics), never checked against whether a referenced file/command still exists in the repo.
- Content tiers (Essential/Reference/Redundant) are coarser than a per-line True/Live/Consistent/Actionable test; no equivalent of a CHALLENGE verdict escalated to the user for ambiguous cases.
- No self-improvement loop — the tool's own scoring rules don't evolve from accumulated usage evidence.
- No no-loss re-verification pass after applying (relies on "verbatim extraction only" as a stated principle, not a diffed re-check comparable to wrsmith108's explicit validate phase).
