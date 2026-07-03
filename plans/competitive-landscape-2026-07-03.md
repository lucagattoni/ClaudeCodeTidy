# Competitive landscape — CLAUDE.md maintenance tools (2026-07-03)

Source: GitHub search (`gh search repos`) across `CLAUDE.md optimize/tidy/slim/cleanup`, `AGENTS.md maintain`, `claude memory hygiene`, plus direct repo inspection (`gh repo view`) of every plausible match. Triggered by the repo rename to `TidyClaudeMD`, to sanity-check whether the name and positioning collide with an existing tool. Confirmed: `lucagattoni/TidyClaudeMD` is the current public name, visibility PUBLIC.

**Bottom line: no name collision, but a crowded and well-developed niche.** At least four repos solve the identical problem ("CLAUDE.md is bloated, shrink it losslessly") and one (`geuneda`) is already benchmarking itself against two of the others by name in its own README. This space is more contested than a from-scratch build would assume — differentiation has to be explicit and demonstrable, not assumed.

---

## Direct competitors (same problem: shrink a bloated CLAUDE.md)

### `wrsmith108/claude-md-optimizer` (19 stars)
Progressive-disclosure optimizer for CLAUDE.md **and** AGENTS.md **and** copilot-instructions.md. 6-phase workflow (detect → constraints → plan → review → execute → validate). Notable strengths this repo lacks:
- **Format-native extraction** — uses each format's own sub-doc mechanism (`@import` for CLAUDE.md, nested files for AGENTS.md, path-scoped instructions for Copilot) rather than generic markdown links.
- **Encryption-awareness** — detects git-crypt/SOPS/age and force-keeps unlock instructions inline (a real correctness bug this repo could hit silently).
- **CI-dependency scanning** — detects scripts that regex-scan the instruction file before moving content out from under them.
- **"Rich abstract" pattern with measured evals** — publishes a small usage study showing a synopsis-plus-link reduces sub-doc reads from 5/5 to 1-2/5 vs. a thin link. This repo's RELOCATE verdict leaves "a one-line pointer" with no equivalent guidance on making that pointer rich enough to avoid a following read.
- **Multi-format.** This repo is CLAUDE.md-only; wrsmith108 also covers AGENTS.md and Copilot instructions in one tool.

Where TidyClaudeMD is stronger: no self-improvement loop, no repo-comprehension gate, no CHALLENGE-style ambiguity escalation — wrsmith108's plan step is a categorization table, not a four-test (True/Live/Consistent/Actionable) interrogation per line.

### `geuneda/claude-md-optimizer` (2 stars)
Scoring-and-linting-first approach: 0-100 automated score, regex anti-pattern detection, cross-file duplicate detection, non-English token-overhead detection (CJK content costs 30-50% more tokens — this repo has no equivalent check), session-cost estimation, injection-order-aware placement (global → project → rules → Memory). Ships a standalone Python analysis script usable outside any agent session.
- Its README already runs a feature-comparison table against `wrsmith108` and a repo called `daymade` — i.e. the authors in this niche are already aware of and benchmarking each other. (Could not locate a `daymade` CLAUDE.md tool via search — likely renamed/unlisted/private since geuneda's README was written.)
- Its "8-priority optimization order" and quantified claims (35% fewer corrections, 60% context savings, citing external sources like Anthropic/HumanLayer/Arize blog posts) is a level of externally-sourced justification this repo's SKILL.md doesn't cite.

Where TidyClaudeMD is stronger: the four-test line interrogation and five-verdict taxonomy (KEEP/COMPRESS/RELOCATE/DELETE/CHALLENGE) is more structured than "Essential/Reference/Redundant" tiers; the no-loss re-verification pass and the self-improving reflect loop have no counterpart here — geuneda's tool re-scores after editing but doesn't learn across runs.

### `NicoAcosta/claude-md-optimizer` (0 stars)
Simpler 4-phase workflow (discovery → analysis → proposal → execution), hard target <50 lines (100 max), extracts to `docs/`, "preserves GSD markers" (undocumented in the README preview). Distributable via `skills.sh`, Claude Code plugin marketplace, Cursor, Codex/OpenCode, Gemini CLI — broader install surface than this repo, which is currently a manual-copy-to-`~/.claude/skills/` install only.

Where TidyClaudeMD is stronger: everything methodological — no comprehension gate, no per-line test, no evidence-gated deletion, no run-record/reflection mechanism. This is the least sophisticated of the four direct competitors.

### `Pinkers01/claude-md-optimizer` (0 stars)
Different paradigm entirely: a **standalone GUI app** (macOS/.app, Windows/.exe, or a single HTML file), not a Claude Code skill. 100% client-side, drag-and-drop, keep/move/delete per section with live char counter, duplicate + rule-contradiction detection (ships hardcoded conflict pairs like "Stripe vs Mollie", "em-dash policy"), exports a ZIP. Has a paid hosted tier (€9 lifetime).
- **Contradiction detection** is a real capability gap: this repo's CHALLENGE verdict catches "contradiction with other rules" but only within a live tidy session's read of the files — Pinkers01 ships a small library of common contradiction patterns to check against explicitly.
- Fundamentally different distribution: works for *any* LLM context file (not Claude-Code-specific), needs zero Claude Code install, but also runs with no repo/git awareness at all (no comprehension gate, no visibility check, no branching conventions) since it's a browser tool over pasted text.

Where TidyClaudeMD is stronger: repo-grounded verdicts (True/Live tests against the actual codebase) are impossible for a tool that only ever sees the pasted file text, not the repo it describes.

### `tsalkin/claude-memory-hygiene` (1 star)
Not a CLAUDE.md tool — targets `MEMORY.md`, Claude Code's separate auto-loaded memory index (the same "memory" subsystem this repo's own CLAUDE.md session is using right now). Zero-dependency Python CLI, three commands (`report`/`lint`/`archive`), dry-run by default, archives (never deletes) stale volatile files by regex, protects pinned durable files by a second regex filter.
- Striking convergent tagline: *"Keep ... under budget — without losing knowledge"* vs. this repo's *"keep every repo's CLAUDE.md slim without losing information."* Independently arrived-at, same core promise, adjacent-but-different artifact (MEMORY.md index vs. CLAUDE.md instructions).
- Its **pin/volatile regex model** is a much simpler, purely mechanical alternative to this repo's four-test line interrogation — cheaper to run (no LLM judgment needed at all, it's a pure script), but blind to *content* quality (a pinned file full of dead instructions is never flagged; only file *age/type* drives the decision).

Where TidyClaudeMD is stronger: covers CLAUDE.md, not MEMORY.md — different artifact, so this is more "adjacent tool worth knowing about" than a head-to-head competitor. If this repo's user memory system (the one active in this very session) is ever audited, `claude-memory-hygiene`'s pin/volatile/archive model is a relevant reference implementation.

---

## Adjacent tools (different problem, same file)

| Repo | Stars | What it actually does | Relation to TidyClaudeMD |
|---|---|---|---|
| `alirezarezvani/ClaudeForge` | 397 | **Generates** and continuously **syncs** CLAUDE.md to match the codebase (drift audit, link check, dependency rescan run weekly); hard 150-line cap enforced by a `PostToolUse` hook, not by LLM judgment | Solves "keep it accurate," not "keep it slim" — could point users at a hook-enforced hard cap this repo doesn't have |
| `severity1/claude-code-auto-memory` | 151 | Watches Edit/Write/Bash operations and **auto-updates** CLAUDE.md at end of turn via marker-based auto-managed sections, runs in an isolated agent to avoid spending main-session context | Growth-oriented (keeps memory current as code changes), not shrink-oriented; no overlap on the slimming problem itself |
| `BayramAnnakov/claude-reflect` | 1,137 | Captures user corrections/feedback and **feeds them into** CLAUDE.md/AGENTS.md automatically, plus mines session history for repeatable workflow patterns to turn into slash commands | Conceptually the mirror image of this repo's own reflect skill — one grows CLAUDE.md content in, the other (`claude-md-tidy-reflect`) grows the *tidy skill itself*, not the target file. Worth knowing the name is already taken for a different "reflect on CLAUDE.md changes" concept |
| `agent-sh/agnix` | 316 | Linter/LSP validating CLAUDE.md, AGENTS.md, SKILL.md, hooks, MCP configs against 432 correctness rules; IDE plugins (VS Code, JetBrains, Neovim, Zed) + GitHub Action + autofix | Solves "is this file syntactically/structurally broken," not "is this file bloated" — orthogonal, could be recommended as a complementary tool rather than a competitor |

---

## What this means for TidyClaudeMD

1. **The exact pitch — "shrink CLAUDE.md without losing information" — is not unique.** At least `wrsmith108`, `geuneda`, `NicoAcosta`, and `tsalkin` (for MEMORY.md) all claim zero/lossless information retention as their core promise. This repo's actual differentiators have to be argued on mechanism, not on the goal statement:
   - the **four-test (True/Live/Consistent/Actionable) line-level interrogation** — none of the direct competitors interrogate individual lines against the live repo state this granularly; they classify by tier (Essential/Reference/Redundant) or score by regex pattern;
   - the **CHALLENGE verdict** as a named, never-unilaterally-resolved escalation path for ambiguous content — the others either auto-decide or leave contradiction-detection as a flat warning list (Pinkers01) with no per-item user dialogue;
   - the **self-improving reflect loop with evidence-tagged run records** — no other repo found in this search learns from its own past runs; ClaudeForge and auto-memory automate *syncing content in*, not *improving the tidy methodology itself*.
2. **Real capability gaps worth borrowing from competitors** (candidates for the existing `plans/v1.3.0-candidate-improvements.md` backlog, not applied here):
   - CI/script-dependency scanning before relocating content (wrsmith108) — this repo's apply phase greps for referencing docs but not for scripts that regex-scan the CLAUDE.md itself.
   - Encryption-awareness (wrsmith108) — if a target repo's CLAUDE.md sits behind git-crypt/SOPS, this repo has no documented check.
   - A lightweight non-LLM `report`-only mode (tsalkin, geuneda) — every other tool in this space offers a free, instant, no-plan-required size/score check; this repo's only entry point is the full tidy workflow.
3. **Naming is clear.** No repo, package, or plugin found under `TidyClaudeMD`, `claude-md-tidy`, or close variants. The crowded names are all `claude-md-optimizer` (four unrelated repos share that exact name) — this repo's distinct name is a minor practical advantage (discoverability, no plugin-marketplace collision).
