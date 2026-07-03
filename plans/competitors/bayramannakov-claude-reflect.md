# BayramAnnakov/claude-reflect

**URL:** https://github.com/BayramAnnakov/claude-reflect
**Stars (2026-07-03):** 1,137
**Category:** Adjacent (feeds content into CLAUDE.md, doesn't slim it) — conceptual mirror of TidyClaudeMD's own reflect skill
**License:** MIT
**Version at time of review:** 2.6.0, 160 passing tests, cross-platform (macOS/Linux/Windows)

## What it is

A self-learning system for Claude Code that captures user corrections and discovers workflow patterns during real sessions, turning them into permanent CLAUDE.md/AGENTS.md memory and reusable slash commands.

## Feature 1 — Learn from corrections

```
You correct Claude Code (automatic) -> Hook captures to queue (automatic) -> /reflect adds to CLAUDE.md (manual review)
```

When the user corrects Claude ("no, use gpt-5.1 not gpt-5"), a hook captures the correction to a queue automatically; a manual `/reflect` step reviews the queue and writes durable memory into CLAUDE.md/AGENTS.md.

## Feature 2 — Discover workflow patterns (new in v2)

```
Your past sessions -> /reflect-skills finds patterns -> Generates /commands
```

Analyzes session history to find repeating tasks that could become reusable slash commands, then generates them.

## Positioning vs. TidyClaudeMD

This is the **mirror image** of TidyClaudeMD's own `claude-md-tidy-reflect` skill: `claude-reflect` grows CLAUDE.md *content* in from live session feedback (additive, user-facing memory); TidyClaudeMD's reflect skill grows the *tidy skill's own methodology* from recorded tidy-run evidence (self-referential, meta-level), and the target CLAUDE.md itself is only ever shrunk, never grown, by either of TidyClaudeMD's two skills. The two systems solve complementary, non-overlapping problems — one could plausibly run both in the same repo (claude-reflect adding corrections in, `claude-md-tidy` periodically slimming the result).

## Naming collision risk

The word "reflect" for a "learn from corrections and update CLAUDE.md" concept is already an established, 1,000+-star project name (`claude-reflect`, and its `/reflect` command). TidyClaudeMD's own skill is named `claude-md-tidy-reflect` (compound, CLAUDE.md-suite-scoped) rather than a bare `/reflect`, which avoids a direct name clash, but the conceptual space around "a `/reflect`-style command for Claude Code" is not empty.

## Gaps / differences relative to TidyClaudeMD

- No slimming, scoring, or bloat-reduction logic — purely additive.
- No repo-comprehension gate, no per-line True/Live/Consistent/Actionable test on existing content, no CHALLENGE-equivalent.

## What's worth noting (not directly borrowable, but relevant context)

Its automatic-hook-capture + manual-review-command split (capture continuously, apply on demand) is a workable pattern for TidyClaudeMD's own backlog item "no channel for feedback that arrives outside a tidy run" (`v1.3.0-candidate-improvements.md`, item 6) — evidence for `claude-md-tidy-reflect` could similarly be captured passively during ordinary conversations via a hook, rather than requiring the user or a future session to manually re-derive it.
