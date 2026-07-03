# alirezarezvani/ClaudeForge

**URL:** https://github.com/alirezarezvani/ClaudeForge
**Stars (2026-07-03):** 397
**Category:** Adjacent (generation + continuous sync, not slimming)
**License:** MIT
**Version at time of review:** 2.0.0

## What it is

A comprehensive toolkit that generates, enhances, and continuously maintains CLAUDE.md files so they stay synchronized with the codebase — installable as a Claude Code plugin.

## What it does (per its "What's New" section)

- **Installable Claude Code plugin** — manifest at `.claude-plugin/plugin.json`; `/plugin marketplace add alirezarezvani/ClaudeForge && /plugin install claudeforge`.
- **Hard 150-line cap per CLAUDE.md**, enforced *deterministically* (not by LLM judgment) via `hooks/hooks.json` firing on `PostToolUse(Edit|Write)` **and** on `InstructionsLoaded` for every `load_reason`. Larger projects spread content across chained sub-files via `@path` imports instead of exceeding the cap.
- **`/sync-claude-md`** — walks every CLAUDE.md in the repo, prunes stale references, splits files that exceed the cap, repairs root↔sub-file import chains.
- **`/sync-claude-md --weekly`** — orchestrates three forked, task-style skills in parallel: `claude-md-drift-audit`, `claude-md-link-check`, `claude-md-dependency-rescan`.
- **Karpathy behavioral guidelines** auto-embedded in every generated CLAUDE.md, and also installed standalone as `~/.claude/skills/karpathy-guidelines/`, scoped to code-file globs.
- **`AGENTS.md` / `.cursorrules` / `.windsurfrules` interop** — `/enhance-claude-md` detects sibling instruction files from other tools and chains them via `@`-imports instead of overwriting them.
- **`CLAUDE.local.md` personal tier** — per-developer overrides exempt from the line cap, auto-gitignored.
- **Layered hook config** — committed defaults (`hooks/hooks-config.json`) plus a presumed personal-override layer (README preview truncated before full detail).

## Core positioning

Explicitly framed as solving *drift* (CLAUDE.md falling out of sync with the actual codebase as it evolves) and *tedium* (manual creation/maintenance), not primarily bloat — though the 150-line hard cap addresses bloat as a side effect of the sync discipline.

## Gaps / differences relative to TidyClaudeMD

- The cap is enforced by a deterministic hook trigger, not by an LLM judging each line — cheap and reliable for "did this cross 150 lines," but blind to *quality* (a file that's exactly 149 lines of dead instructions passes; a genuinely tight 160-line file fails and gets auto-split even if every line earns its place).
- No per-line True/Live/Consistent/Actionable test, no CHALLENGE-equivalent user escalation for ambiguous or contradictory content — sync is drift/link/dependency-focused, not content-quality-focused.
- No self-improvement loop over its own methodology (the parallel weekly audits improve *target* CLAUDE.md files, not ClaudeForge's own rules).

## What's worth borrowing

**A deterministic hook-enforced hard cap as a belt-and-suspenders backstop.** TidyClaudeMD's slimming is entirely judgment-driven (five verdicts, four tests) and only runs when invoked — nothing stops a CLAUDE.md from silently growing past any guardrail between tidy runs. A `PostToolUse` hook that warns (not necessarily blocks) when a CLAUDE.md crosses the global hygiene rule's own ~150-line guardrail would catch drift the moment it happens, rather than waiting for the next manual `/claude-md-tidy` invocation.
