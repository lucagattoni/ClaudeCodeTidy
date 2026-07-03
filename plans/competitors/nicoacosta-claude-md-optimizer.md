# NicoAcosta/claude-md-optimizer

**URL:** https://github.com/NicoAcosta/claude-md-optimizer
**Stars (2026-07-03):** 0
**Category:** Direct competitor
**License:** MIT

## What it is

An agent skill (not Claude-Code-specific — distributed across multiple agent platforms) for optimizing bloated CLAUDE.md and AGENTS.md files by extracting content to `docs/`.

## What it does

Analyzes CLAUDE.md/AGENTS.md files and restructures them: keeps only critical content inline, extracts valuable detail to `docs/`, deletes filler outright.

**Goal:** CLAUDE.md under 50 lines (hard max 100). "Every line must earn its token cost."

## Workflow (4 phases)

1. **Discovery** — find all CLAUDE.md/AGENTS.md files, detect repo type
2. **Analysis** — classify sections as KEEP / EXTRACT / DELETE
3. **Proposal** — present optimization plan with before/after line-count projections
4. **Execution** — restructure files, preserve "GSD markers" (undocumented in the public README — likely a project-specific marker/tag convention from the author's own workflow, not explained for external users)

## Distribution surface (broadest of the direct competitors)

- `npx skills add NicoAcosta/claude-md-optimizer` (skills.sh)
- `/plugin marketplace add NicoAcosta/agent-skills` (Claude Code plugin)
- `/add-plugin claude-md-optimizer` (Cursor)
- Codex / OpenCode via `.codex/INSTALL.md` / `.opencode/INSTALL.md`
- `gemini extensions install https://github.com/NicoAcosta/claude-md-optimizer` (Gemini CLI)
- Manual clone + read `skills/claude-md-optimizer/SKILL.md`

## Gaps relative to TidyClaudeMD

This is the least methodologically developed of the four direct CLAUDE.md-optimizer competitors:
- No repo-comprehension gate, no per-line test (True/Live/Consistent/Actionable), no CHALLENGE-equivalent escalation for ambiguous or contradictory content.
- No evidence-gated DELETE — "delete filler" has no stated verification step (no grep-confirmed dead reference requirement).
- No self-improvement loop.
- The undocumented "GSD markers preserved" claim is the one feature this repo doesn't have an obvious equivalent for, but it's unclear from the README what it actually protects — not enough information to adopt or reject.

## What's still worth noting

Its install-surface breadth (5 distinct platforms/marketplaces from one skill definition) is a distribution model TidyClaudeMD does not currently match — this repo is manual-copy-to-`~/.claude/skills/` only, per the main README ("no symlink or sync automation yet").
