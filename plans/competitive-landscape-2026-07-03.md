# Competitive landscape — CLAUDE.md maintenance tools

Source: GitHub search (`gh search repos`) across `CLAUDE.md optimize/tidy/slim/cleanup`, `AGENTS.md maintain`, `claude memory hygiene`, `.claude/rules`, `SKILL.md lint`, plus direct repo inspection (`gh repo view`) of every plausible match. Triggered by the repo rename to `TidyClaudeMD`, to sanity-check whether the name and positioning collide with an existing tool. Confirmed: `lucagattoni/TidyClaudeMD` is the current public name, visibility PUBLIC.

**Inclusion bar.** A project earns a place in this document only if it brings a **fresh idea or feature genuinely worth implementing in TidyClaudeMD** — not because it exists, not because it's a nominal competitor, and not just because it's another entry in a crowded niche. Every entry below states, specifically, what that idea is. Projects reviewed and found to bring nothing implementable are listed once, briefly, at the bottom — for the record, not as competitors.

---

_**Updated 2026-07-09** against TidyClaudeMD v0.20.2 (up from v0.9.1 at the original 2026-07-03 pass — scope expanded from CLAUDE.md-only to four target classes: project/user CLAUDE.md, `.claude/rules/`, `SKILL.md` skills, and auto memory, plus a self-improving reflect loop and a git-commit-per-iteration reversibility gate). Re-checked all 9 previously catalogued repos; two ideas from the original pass (encryption-awareness, CI-dependency scanning) are already built in, shipped in v0.6.0. Searched fresh angles matching the expanded scope and found two new `--skills`-class projects that clear the bar: `TheStack-ai/pulser` and `nick2781/claudoctor`. Re-applied the inclusion bar to the full set — two previously-listed projects (`NicoAcosta/claude-md-optimizer`, `BayramAnnakov/claude-reflect`) no longer clear it and moved to "Reviewed, no fresh idea." Ranked idea list: [Features worth prototyping](#features-worth-prototyping-ranked-by-convergence)._

---

## Projects that clear the bar

### `wrsmith108/claude-md-optimizer` (19 stars, active — pushed 2026-06-25)
Progressive-disclosure optimizer for CLAUDE.md, AGENTS.md, and copilot-instructions.md in one tool.

**Idea 1 — format-native multi-file coverage.** Uses each format's own sub-doc mechanism (`@import` for CLAUDE.md, nested files for AGENTS.md, path-scoped instructions for Copilot) to cover all three in one pass, rather than treating a sibling instruction file as out of scope. A version of this for TidyClaudeMD would promote AGENTS.md from an import-visibility check to a fully audited target, and extend to copilot-instructions.md the same way.

**Idea 2 — a measured eval of the rich-abstract pointer.** Published a small usage study (thin link → agent re-opened the sub-doc 5/5 times on test questions; rich abstract → 1-2/5) rather than asserting the pattern works. TidyClaudeMD's RELOCATE verdict already mandates a rich-abstract pointer by rule — a lightweight post-RELOCATE eval, generating a few realistic questions about the relocated content and checking via a fresh read whether the pointer alone answers them, would turn that rule from an assumption into a measured, self-validating one.

### `geuneda/claude-md-optimizer` (2 stars, dormant since 2026-03-18)
Scoring-and-linting tool for CLAUDE.md.

**Idea — a combined budget across every loaded instruction source, not just per-file.** Enforces a numeric guardrail *per file type* (project CLAUDE.md ≤150 lines, user CLAUDE.md ≤50, each `.claude/rules/*.md` ≤30, `MEMORY.md` ≤200) **and** a combined total across all of them at once (≤250 lines, optimal ≤180). A single trackable number for "how much context does this session actually load, end to end" is a genuinely different signal than any one file's own guardrail — and Step 2's survey already assembles the full picture (every CLAUDE.md, every rules file, and with `--all`, every target class) that computing it would need.

### `Pinkers01/claude-md-optimizer` (0 stars, dormant since 2026-05-06)
A standalone GUI app, not a Claude Code skill — duplicate + rule-contradiction detection against a small hardcoded conflict-pair library (e.g. "Stripe vs Mollie").

**Idea — an explicit, maintained contradiction-pattern library.** TidyClaudeMD's Consistent? test already probes contradiction *shapes* generically (autonomy vs. ask-first, default vs. exception, conflicting thresholds). A small, explicit, growing list of specific, named recurring conflicts alongside it would be a cheap complement — and a natural thing for the reflect loop to grow over time as real contradictions get resolved across repos.

### `tsalkin/claude-memory-hygiene` (1 star, active — pushed 2026-06-19)
A zero-dependency Python CLI targeting `MEMORY.md` — the exact artifact TidyClaudeMD's `--memory` class covers.

**Idea — a zero-session mechanical tier for memory hygiene.** Runs with no LLM call at all: a pin/volatile regex split, dry-run by default, archive-never-delete. A mechanical pre-tier for TidyClaudeMD's `--memory` class — file age/size only, no content judgment, cron- or hook-runnable — would be a genuine complement to the content-aware judgment the full class already does, for the moments a live session isn't worth spinning up.

### `TheStack-ai/pulser` (18 stars, active — pushed 2026-06-30)
Full doc: [`competitors/thestack-ai-pulser.md`](competitors/thestack-ai-pulser.md). A `SKILL.md` linter.

**Idea 1 — skill behavioral testing (`pulser eval`).** Actually *runs* a skill against YAML-defined test cases via `claude -p` and asserts on the output, tracking regressions across runs with a distinct exit code from a fresh failure. Of everything found across both search passes, this is the single idea with the clearest payoff for the least conceptual distance from what TidyClaudeMD already does: a regression is exactly "this skill's eval passed in an earlier run log, fails now," and the log format already has the raw material for that comparison. A `--skills --eval` mode would move the `--skills` class from checking structure to checking behavior.

**Idea 2 — reversibility without git.** A single-level backup-and-`undo` — lighter than TidyClaudeMD's per-iteration commits, but needing zero version control. A lighter fallback tier built the same way, for `--skills`/`--memory` targets that aren't git-tracked, would extend autonomous-safe editing to cases the current reversibility gate routes to confirm-first today.

### `nick2781/claudoctor` (0 stars, active — created 2026-05-22, roadmap items still landing)
Full doc: [`competitors/nick2781-claudoctor.md`](competitors/nick2781-claudoctor.md). A cross-agent skill/CLAUDE.md linter — the most structurally ambitious single find in this search.

**Idea 1 — sweep plugin/marketplace caches, not just live skill directories.** Scans `~/.claude/plugins/cache/` and `~/.claude/plugins/marketplaces/` alongside live skill directories; its own example run found real scale there (55 duplicates, 57 conflicts across 393 skills). Not hypothetical for this repo: mid-session on 2026-07-08, this exact machine's plugin cache was found holding two full stale copies of TidyClaudeMD itself (`0.9.1/` and `0.18.0/`). Extending `--skills --user`'s locations to include plugin cache/marketplace directories would catch exactly this.

**Idea 2 — cross-skill overlap and conflict detection.** Byte-identical duplicates, near-duplicates (same body, different frontmatter), name conflicts, and similarity-based overlap (Jaccard on description tokens) — all computed *across* the skills in one sweep, not just within each file. pulser independently built a lighter version of the same idea (trigger-keyword overlap); two projects converging on it is good evidence a "Skill-overlap?" test, run once across the whole `--skills` sweep, is worth adding.

**Idea 3 — exact tokenization.** Real per-file token counts via `@anthropic-ai/tokenizer`, not an estimate. Swapping a real tokenizer call into `--report`'s mechanical-checks tier (already scriptable and judgment-free) would sharpen every size/cost number the suite reports, replacing today's heuristics.

**Idea 4 — a shareable, offline report artifact.** `report --format html` — one file, dark-mode aware, severity counts, review columns — built for a team to look at together, not just the person who ran the tool. The project already has dataviz/artifact-design tooling available in-session, making an equivalent `--report --output report.html` a low-effort extension.

**Idea 5 — a missing-content completeness check.** Flags a CLAUDE.md missing canonical sections (Tone, Tools, Workflow). A version of this for TidyClaudeMD should route through CHALLENGE ("no build/test commands documented anywhere — intentional, or missing?") rather than auto-inserting boilerplate, staying consistent with how the suite treats anything only a human can judge.

### `alirezarezvani/ClaudeForge` (402 stars)
Generates and continuously syncs CLAUDE.md to match the codebase — a different problem (accuracy, not bloat) with one mechanism worth borrowing.

**Idea — a hard-enforced size cap, not just an advisory one.** A `PostToolUse` hook that hard-blocks at 150 lines, enforced by the hook itself, not LLM judgment. A stricter variant of TidyClaudeMD's existing advisory-only companion hook — for users who want a hard line instead of a reminder — would be a small, self-contained addition.

### `severity1/claude-code-auto-memory` (152 stars)
Auto-updates CLAUDE.md at end of turn — a growth-oriented tool, but its execution model is worth borrowing.

**Idea — isolated-agent execution to protect the main session's context.** Runs its update logic in a separate agent specifically so the work doesn't spend the invoking session's own context budget. A fork/sub-agent execution mode for a heavy `--skills`/`--memory`/`--all` run would let a large tidy pass happen without spending the invoking session's own context on it.

### `agent-sh/agnix` (334 stars, active — pushed 2026-07-05)
A 432-rule correctness linter for CLAUDE.md/AGENTS.md/SKILL.md/hooks/MCP configs — a different problem (is this file broken, not is it bloated) with a distribution shape worth noting.

**Idea — IDE-level presence.** Ships as VS Code, JetBrains, Neovim, and Zed extensions surfacing findings inline, not just a CLI or a conversational skill. No CLAUDE.md/rules/skills/memory hygiene tool found in this search has any IDE-level presence at all — an open surface for whoever builds it first.

---

## Reviewed, no fresh idea (2026-07-09)

Projects genuinely reviewed and found to bring nothing implementable beyond what's already better represented above, or nothing relevant to TidyClaudeMD's actual mission. Kept here for the record so the search isn't silently re-run later.

| Repo | Stars | Why it doesn't clear the bar |
|---|---|---|
| `NicoAcosta/claude-md-optimizer` | 0 | Simplest of the direct competitors methodologically; its one distinguishing trait — distribution to skills.sh, Cursor, Codex/OpenCode, Gemini CLI — is a packaging decision, not an implementable feature, and porting TidyClaudeMD's actual invocation surface to other agent runtimes is a different-product question |
| `BayramAnnakov/claude-reflect` | 1,235 | Mines session history to auto-generate slash commands from repeated patterns — a different mission (growing new capability) from TidyClaudeMD's (shrinking existing instruction files) |
| `sgamma/skills` | 0 | Personal skill collection; its CLAUDE.md optimizer is a minor, undifferentiated feature |
| `BENZEMA216/self-purify` | 0 | CLAUDE.md optimization is one minor feature of a broader security/audit plugin, nothing distinctive |
| `Goodsmileduck/claude-registry` | 1 | CLAUDE.md optimization is one listed capability of a plugin marketplace, nothing distinctive |
| `Pinkers01/claude-md-optimizer` — zero-install GUI aspect | — | The GUI-over-pasted-text shape itself (as opposed to its contradiction-pattern library, listed above) can't do repo-grounded verification, which is core to how TidyClaudeMD works — a different product, not a feature to add |

---

## Features worth prototyping, ranked by convergence

Each idea is cited to the specific project(s) it came from — every one above cleared the "worth implementing" bar. Ranked by how many independent projects converged on the same idea, then by how concretely it applies to this repo.

1. **Skill behavioral testing.** *(pulser)* — a `--skills --eval` mode, reusing the run-log history to detect regressions.
2. **Cross-skill overlap/conflict detection.** *(pulser, claudoctor — two independent projects)* — a "Skill-overlap?" test run once across a whole sweep, not per-file.
3. **A combined budget across all loaded instruction sources.** *(geuneda)* — one number for project + user + rules + memory together.
4. **Multi-format, multi-ecosystem coverage.** *(wrsmith108, claudoctor — two independent projects)* — AGENTS.md as a full target, copilot-instructions.md, and awareness of `~/.codex/`, `~/.hermes/`, `~/.cursor/rules/`.
5. **Exact tokenization.** *(claudoctor)* — a real tokenizer call in place of heuristic size/cost estimates.
6. **Sweeping plugin/marketplace caches.** *(claudoctor)* — confirmed valuable on this exact machine.
7. **A non-git reversibility fallback.** *(pulser)* — backup-and-undo for targets the current gate can't autonomously touch.
8. **Measuring the rich-abstract pointer.** *(wrsmith108)* — a self-validating check for a rule TidyClaudeMD already enforces.
9. **A shareable report artifact.** *(claudoctor)* — an HTML/Markdown export of the plan or summary.
10. **A missing-content completeness check.** *(claudoctor)* — routed through CHALLENGE, never auto-inserted.
11. **An explicit contradiction-pattern library.** *(Pinkers01)* — grown over time by the reflect loop.
12. **A zero-session mechanical tier for memory hygiene.** *(tsalkin)* — cron- or hook-runnable, no LLM call.
13. **A hard-enforced size cap option.** *(ClaudeForge)* — a stricter sibling to the existing advisory hook.
14. **Isolated-agent execution for heavy runs.** *(severity1/claude-code-auto-memory)* — protects the invoking session's own context.
15. **IDE-level presence.** *(agnix)* — a distribution surface no tool in this niche has claimed yet.

### What this doesn't change

Naming is still clear — no collision found under `TidyClaudeMD`, `claude-md-tidy`, or close variants, across either the 2026-07-03 or 2026-07-09 passes.
