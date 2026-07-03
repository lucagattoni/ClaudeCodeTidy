# wrsmith108/claude-md-optimizer

**URL:** https://github.com/wrsmith108/claude-md-optimizer
**Stars (2026-07-03):** 19
**Category:** Direct competitor
**License:** MIT

## What it is

A Claude Code skill that optimizes oversized agent instruction files using progressive disclosure. Covers three formats in one tool: **CLAUDE.md** (Claude Code), **AGENTS.md** (OpenAI Codex / GitHub Copilot agents), and **copilot-instructions.md** (GitHub Copilot).

## Problem statement

Instruction files accumulate troubleshooting guides, deployment commands, API references, and config examples over 6+ months of development, commonly reaching 800-1,500+ lines. Since these files load into every AI session as context, oversized files waste the context window — Anthropic warns adherence degrades above 200 lines for CLAUDE.md.

## Supported formats and targets

| Format | Primary Location | Target Size | Native Sub-Doc Mechanism |
|---|---|---|---|
| CLAUDE.md | `./CLAUDE.md` or `./.claude/CLAUDE.md` | <200 lines | `@import` syntax; `.claude/rules/*.md` |
| AGENTS.md | `./AGENTS.md` (hierarchical) | <32 KiB chain (Codex default) | Nested `AGENTS.md` in subdirectories |
| copilot-instructions.md | `.github/copilot-instructions.md` | ~100 lines | `.github/instructions/NAME.instructions.md` |

## Workflow (6 phases)

1. **Detect & Analyze** — scan repo for all three file types, report sizes vs. targets, categorize sections as Essential / Reference / Redundant
2. **Detect Constraints** — check for git-crypt/SOPS/age encryption; identify format-native sub-doc mechanisms; scan for CI scripts that regex-scan the file
3. **Plan** — present categorization table (what stays inline vs. extracted, and where)
4. **Review** — user approves before any changes
5. **Execute** — create sub-documents via the format's native mechanism; rewrite main file with rich inline synopses + progressive disclosure links
6. **Validate** — diff before/after content to confirm zero information loss; verify format target met

## Distinctive features

- **Format-native extraction.** Uses each format's own sub-doc mechanism (`@import` chains, nested AGENTS.md, path-scoped Copilot instructions) instead of generic markdown links.
- **Encryption-awareness.** Detects git-crypt, SOPS, age; places sub-docs only in unencrypted paths; force-classifies encryption-unlock instructions as Essential (a "chicken-and-egg" protection — you can't decrypt the sub-doc that tells you how to decrypt).
- **CI-dependency scanning.** Detects scripts that regex-scan the instruction file (e.g. CI checks that grep CLAUDE.md for a required phrase) and keeps matched content inline so relocation doesn't silently break a pipeline.
- **The "Rich Abstract" pattern, with a measured eval.** Central thesis: *how* you write a reference matters as much as *whether* you extract it. A thin link ("For testing details, see docs/testing.md") gives the agent no way to judge relevance, so an agent facing ambiguity follows every such link to be thorough. A rich abstract — 3-5 sentences with concrete facts (framework, threshold, convention) plus the link — lets the agent answer most questions without opening the file at all.
  - Informal usage eval (4 strategies × focused/ambiguous tasks, counting sub-doc opens): thin inline ref → 2/5 files opened (focused/ambiguous); lazy frontmatter alone → 2/5; **rich abstract + link → 1/1 (focused), 2/5 (ambiguous)**; rich abstract + lazy frontmatter → same as rich-abstract-alone (lazy frontmatter added no measurable reduction on top of a good synopsis).
  - Conclusion cited: instruction-file content loads eagerly at session start on all major platforms (Claude Code, Copilot, Cursor) — `loading_strategy: lazy` in Copilot frontmatter appears unimplemented/aspirational. The only proven platform-level lazy loading is Claude Code's `ToolSearch` for MCP tool schemas, which doesn't apply to instruction files. So savings must come from (a) fewer tokens loaded at startup and (b) fewer sub-doc reads during the session — rich abstracts address both.
- **Zero-loss validation** — aborts if post-optimization diff shows missing content.
- **Refuses to run** if CLAUDE.md is under 200 lines, AGENTS.md chain is within Codex's 32 KiB budget, copilot-instructions.md is under 150 lines, or more than 80% of content is already Essential.

## Install / distribution

Manual clone into `~/.claude/skills/`, or published as a Claude Code plugin via skillsmith.app.

## Gaps relative to TidyClaudeMD

- No repo-comprehension gate — doesn't build a picture of the repo (README, CI, git history) before judging content, just classifies the instruction file's own sections.
- No CHALLENGE-equivalent escalation — the "Plan" phase is a categorization table, not a per-line True/Live/Consistent/Actionable test with contradictions raised to the user individually.
- No self-improvement loop — the skill doesn't record run outcomes or evolve its own methodology over time.
