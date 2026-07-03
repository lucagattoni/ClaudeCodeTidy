# severity1/claude-code-auto-memory

**URL:** https://github.com/severity1/claude-code-auto-memory
**Stars (2026-07-03):** 151
**Category:** Adjacent (auto-sync, not slimming)
**License:** MIT

## What it is

A Claude Code plugin that watches what Claude Code edits, deletes, and moves during a session, then quietly updates the project's CLAUDE.md ("memory") in the background — "Your CLAUDE.md, always in sync. Minimal tokens. Zero config."

## Problem framing

CLAUDE.md files go stale as codebases evolve: build commands change but memory doesn't, architecture shifts go unrecorded, conventions drift, new team members get incorrect context. Manual maintenance is "tedious and often forgotten."

## How it works

```
Claude Code edits code -> Plugin tracks changes -> Isolated agent updates memory -> Context stays fresh
```

## Features

- **Automatic sync** — tracks Edit/Write/Bash operations and updates CLAUDE.md at end of turn.
- **Bash operation tracking** — detects `rm`, `mv`, `git rm`, `git mv`, `unlink` so file moves/deletes are reflected in memory too.
- **Minimal-token tracking** — the `PostToolUse` hook itself produces no output; only the `Stop` hook triggers the update, and that update runs in an **isolated agent** in a separate context window, so the main conversation's token budget is untouched.
- **Marker-based updates** — only modifies clearly delimited AUTO-MANAGED sections of CLAUDE.md, preserving manually written content around them.
- **Subtree support** — works with hierarchical CLAUDE.md files in monorepos.
- **`/auto-memory:init`** — interactive wizard that analyzes codebase structure and scaffolds an initial CLAUDE.md.

## Gaps / differences relative to TidyClaudeMD

- Purely growth/freshness-oriented — it keeps CLAUDE.md *accurate*, with no slimming, scoring, or bloat-reduction logic at all. A project using this plugin could still end up with an accurate but enormous CLAUDE.md; nothing here fights line count.
- No per-line quality test, no CHALLENGE-equivalent, no user-approved plan-before-edit step for its automatic updates (it runs at end-of-turn without a review gate, by design — a different trust model than TidyClaudeMD's mandatory stop-and-confirm).
- No self-improvement loop.

## What's worth borrowing

**Running maintenance work in an isolated agent/context to avoid spending the main session's tokens.** TidyClaudeMD's own backlog (`v1.3.0-candidate-improvements.md`, item 2) already flags that `claude-md-tidy`'s repo-comprehension survey is unbounded and can cost more attention than the tidy is worth. Delegating that survey (or the reflect skill's evidence-gathering pass) to a forked/isolated sub-agent — the same pattern this plugin uses for its background updates — is a concrete implementation option for containing that cost, distinct from just making the survey claim-driven.
