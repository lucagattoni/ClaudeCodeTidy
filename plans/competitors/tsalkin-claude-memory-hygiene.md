# tsalkin/claude-memory-hygiene

**URL:** https://github.com/tsalkin/claude-memory-hygiene
**Stars (2026-07-03):** 1
**Category:** Adjacent (direct competitor on the *promise*, different artifact)
**License:** MIT
**Language:** Python (stdlib only), EN+RU docs

## What it is

A single-file, zero-dependency (~230 lines) CLI that keeps a Claude Code agent's `MEMORY.md` — the always-loaded memory *index* file, sibling to the per-topic files under `~/.claude/projects/<slug>/memory/` — under its size budget, without deleting knowledge. Not a CLAUDE.md tool: it targets Claude Code's separate memory subsystem (the same one this session's own memory files live in).

## Tagline convergence

*"Keep a Claude Code agent's MEMORY.md memory index under budget — without losing knowledge."* This is strikingly close to TidyClaudeMD's own pitch ("keep every repo's CLAUDE.md slim without losing information") — independently arrived at, for an adjacent-but-distinct artifact.

## Problem framing

`MEMORY.md` has a hard size budget (~24.4 KB) — past it, the harness emits an "index entries too long" warning and loads only part of the file, so the agent goes blind to some of its own memory (the knowledge still exists on disk, it just stops being *discoverable at recall time* — "silent knowledge loss"). Two failure modes accumulate forever: (a) per-session/per-day state files each add a permanent index line that never stops growing, (b) index lines balloon from one-line hooks into paragraphs.

## The core idea

**Index cleanliness ≠ deleting knowledge.** Facts live in topic files; `MEMORY.md` only *points* at them. So the index can shrink losslessly: shorten hooks (detail stays in the topic file), archive old state files (move, don't delete), pin durable memories so they're never touched, and stop proliferating per-day files.

## Commands

| Command | What it does | Writes? |
|---|---|---|
| `report` (default) | Size vs. budget, index-pointer count, over-long lines, archivable count | No (read-only) |
| `lint` | Lists index lines longer than `--max-chars`, worst first | No |
| `archive` | Moves OLD volatile state files to `archive/` and drops their index lines | Only with `--apply` |

## Key flags

| Flag | Default | Meaning |
|---|---|---|
| `--project PATH` | CWD | Derive memory dir from a project path |
| `--dir PATH` | — | Explicit memory dir override |
| `--keep N` | 3 | How many newest volatile files stay in the index |
| `--max-chars N` | 220 | Index-line length limit for report/lint |
| `--volatile REGEX` | session/daily/dated notes | Filenames treated as archival candidates |
| `--pin REGEX` | `^(feedback\|reference\|user)_` | Filenames NEVER archived (durable) |
| `--apply` | off | Actually move files (otherwise dry-run) |

## Decision model (mechanical, not LLM-judgment)

- **Volatile vs. durable** — a file is an archival *candidate* only if its name matches `--volatile` (default: `session_state`/`session_log`/`daily_*`/any `YYYY-MM-DD` dated note). Everything else is left alone.
- **Pin guard** — a second filter (`--pin`, default `feedback_`/`reference_`/`user_` prefixes) excludes durable notes even if they slipped past the volatile pattern. Pinned files never move.
- **Keep N newest** — candidates sorted newest-first (date-in-filename, then mtime); only the newest `--keep` stay in the index, the rest archive.
- **Archive, not delete** — files move to `archive/`, still on disk and greppable; only their index line drops from the always-loaded `MEMORY.md`.
- **Dates from filename/mtime** — no frontmatter date stamping required, avoiding an "empty date → archive everything" footgun.

## Safety

- Dry-run by default; `archive` only previews until `--apply`.
- Never deletes — worst case is a `shutil.move` into `archive/` in the same memory dir.
- Durable memory protected twice (volatile-candidate filter + pin guard).
- `report` and `lint` never touch a file.

## Credit chain

Cites its "memory that forgets" decay/tier philosophy as coming from `agent-second-brain` and `autograph` (both by `smixs`) — described as an independent, much-smaller subset of that philosophy focused on exactly one artifact (the `MEMORY.md` index), not a fork.

## Relation to TidyClaudeMD

Different artifact (MEMORY.md index vs. CLAUDE.md instructions), so not a head-to-head competitor — but directly relevant because:

1. **TidyClaudeMD depends on the same memory subsystem for its own bookkeeping.** The suite's `RUNS.md` file (run records for `claude-md-tidy-reflect`) is described in this repo's own README as "created on first run, newest at top" with no documented archiving or pruning step — the exact unbounded-growth failure mode this tool exists to prevent, but applied to TidyClaudeMD's own bookkeeping file instead of a target repo's CLAUDE.md.
2. Its pin/volatile/archive model is a cheap, purely mechanical alternative to LLM-judgment-driven slimming — no model call needed at all, at the cost of being blind to *content quality* (a pinned file full of dead instructions is never flagged; only file age/type drives the decision, never truth/liveness/consistency).
