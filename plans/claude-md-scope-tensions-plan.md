# CLAUDE.md scope-placement tensions — research + plan

Status: **proposed, not started.** Researched 2026-07-03 against official Claude Code documentation (`code.claude.com/docs/en/`: `memory.md`, `features-overview.md`, `best-practices.md`). Triggered by a real gap found in conversation: `claudemd-tidy` has no test for "this line belongs in a different file than the one it's in" — the tidy skill's five verdicts (KEEP/COMPRESS/RELOCATE/DELETE/CHALLENGE) only reason about content *within* the current repo. This doc surveys four scope-placement axes at once, since they're the same class of problem, and proposes concrete skill changes for each.

_Updated 2026-07-03 (second pass): added axis 4 (CLAUDE.md vs. AGENTS.md) per user request, researched the same way as the first three._

_Updated 2026-07-03 (third pass): the user correctly pointed out that `@AGENTS.md` is one instance of a general `@path` import mechanism, not an AGENTS.md-specific feature — generalized axis 4 accordingly, and re-verified every claim directly against the actual doc text (`code.claude.com/docs/en/memory.md`, `skills.md`) rather than trusting the research agent's paraphrase, per explicit instruction to check critically. That check caught one overstated claim (cycle detection is documented for `.claude/rules/` symlinks, not for `@import` — the agent conflated the two) and surfaced two real mechanics the agent's summary missed entirely: a one-time approval dialog that can **permanently disable** imports if declined, and backtick-escaping to distinguish a literal `@path` mention from a live import. See the corrected research below._

## The four axes

Every line in a CLAUDE.md implicitly claims to belong exactly where it sits — and, per axis 4, an entire *other file* can implicitly claim rules that CLAUDE.md never sees. Four independent questions can each falsify that:

1. **CLAUDE.md vs. Skill** — is this a fact needed every session, or a procedure/reference used only sometimes?
2. **Project CLAUDE.md vs. user/global CLAUDE.md vs. `CLAUDE.local.md`** — is this project-specific, cross-project-personal, or repo-local-personal?
3. **CLAUDE.md vs. Memory (`MEMORY.md`)** — is this a deliberate, human-authored rule, or an agent-discovered pattern?
4. **CLAUDE.md's `@import` mechanism** — does this CLAUDE.md pull in other files (of which `@AGENTS.md` is one common instance, not a special case), and if so, is the skill actually seeing what those imports contain?

`claudemd-tidy` today only reasons well about axis 1 (via RELOCATE). Axes 2, 3, and 4 are unhandled — a line can be flagged True/Live/Consistent/Actionable/non-redundant and still be filed in the wrong *scope* entirely (or, per axis 4, an entire sibling file's content can be silently invisible to every Claude Code session), and nothing currently catches that.

---

## Axis 1: CLAUDE.md vs. Skill — research

Official guidance (`features-overview.md`, `best-practices.md`, `memory.md`):

- **Size ceiling**: "Keep CLAUDE.md under 200 lines" — bloated files "cause Claude to ignore your actual instructions." (This project's own global hygiene rule already uses a stricter internal ~150-line guardrail — a deliberate, tighter-than-official convention, not a discrepancy to fix.)
- **Context-cost asymmetry**: CLAUDE.md's full content loads every session; a Skill's description loads every session but its full content only loads when invoked or matched to the task ("progressive disclosure"). This is the mechanical reason procedural/reference content is expensive in CLAUDE.md and cheap in a Skill.
- **Decision rule** (official): a fact Claude should know *every* session (build commands, conventions, "never do X" rules) → CLAUDE.md. A multi-step procedure, or reference material used only sometimes, or content that "only matters for one part of the codebase" → Skill.

**Gap check against the current skill:** `claudemd-tidy`'s RELOCATE verdict test is *"Procedure/step detail whose natural owner is a skill, `docs/`, or README"* — this already matches the official decision rule closely. **Axis 1 is already substantially covered.** One small, low-priority refinement: the RELOCATE test could cite the official framing directly ("only matters for one part of the codebase," "used sometimes, not every session") to sharpen borderline calls, but this is a wording tightening, not a new mechanism — folded into item 1 below as an optional COMPRESS-level polish, not a new verdict.

## Axis 2: Project vs. user/global vs. local CLAUDE.md — research

Official guidance (`memory.md`, "Memory" scope table and "How CLAUDE.md files load"):

| Scope | Location | Purpose | Shared with |
|---|---|---|---|
| User instructions | `~/.claude/CLAUDE.md` | Personal preferences for **all** projects | Just you |
| Project instructions | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team-shared instructions | Team, via source control |
| Local instructions | `./CLAUDE.local.md` (gitignored) | Personal, **project-specific** preferences | Just you, this project only |

Key mechanics: all scopes are **additive, not overriding** — every discovered file concatenates into context (root-down by directory, `CLAUDE.local.md` appended after `CLAUDE.md` at each level). Nested per-directory CLAUDE.md files are a documented, first-class pattern for monorepos, loaded on demand when Claude reads files in that subdirectory.

**Decision rule** (official): personal, cross-project habits → user `~/.claude/CLAUDE.md`. Team-shared, project-specific conventions → project `CLAUDE.md`, git-tracked and PR-reviewed. Personal preferences specific to *this* project only (sandbox URLs, personal branch names, local test data) → `CLAUDE.local.md`, gitignored.

**Gap check against the current skill:** nothing. This axis is entirely unhandled today. A project `CLAUDE.md` line that is actually a generic, cross-project working preference (and doesn't already exist in the global file — if it did, that's Redundant-by-order, a different, already-covered case) currently has no test that would flag it. Same for a line that's a personal, repo-specific preference someone hardcoded into the shared, team-reviewed `CLAUDE.md` instead of `CLAUDE.local.md`. **This is the concrete gap the user identified in conversation** — proposed fix below (item 2).

## Axis 3: CLAUDE.md vs. Memory (`MEMORY.md`) — research

Official guidance (`memory.md`, "CLAUDE.md vs auto memory"):

| Aspect | CLAUDE.md | Auto Memory |
|---|---|---|
| Who writes it | You (human-authored) | Claude (agent-discovered) |
| Contains | Instructions and rules | Learnings and patterns |
| Loaded into context | Full content, every session | Index only (`MEMORY.md`, capped at 200 lines / 25KB) — topic files recalled on demand, not preloaded |
| Use for | Coding standards, workflows, architecture | Build commands Claude discovered, debugging insights, preferences Claude noticed |

Neither CLAUDE.md nor Memory *enforces* anything — both are context that shapes reasoning, not a hard gate (for hard guarantees, official guidance points to `PreToolUse` hooks instead). The documented **promotion path**: when something Claude learned in Memory turns out to matter every session, the user says "add this to CLAUDE.md" (or edits directly) — moving it from agent-discovered-and-ephemeral to human-decided-and-durable.

**Decision rule** (official): deliberate, stable, team-relevant rules → CLAUDE.md. Ad hoc discovered patterns, debugging insights, machine-local tool preferences → Memory, not CLAUDE.md.

**Gap check against the current skill:** also entirely unhandled. A CLAUDE.md line that reads like a discovered debugging insight or an ad hoc noticed pattern (not a deliberate team rule) rather than an intentional instruction is invisible to all five current verdicts — True/Live/Consistent/Actionable/Redundant-by-order all pass a line like this if it happens to be accurate and unambiguous, even though it's arguably the wrong *kind* of content for a PR-reviewed rulebook. Proposed fix below (item 3) — deliberately scoped narrower than the full bidirectional promotion path, for reasons explained there.

## Axis 4: CLAUDE.md's `@import` mechanism (general), with AGENTS.md as one instance — research

Official guidance, verified directly against the source text of `memory.md` "Import additional files" and "AGENTS.md" sections (not just the research agent's paraphrase — see the update note above for why):

- **`@path` import is fully general-purpose, not AGENTS.md-specific.** Verbatim: *"CLAUDE.md files can import additional files using `@path/to/import` syntax."* Documented examples include `@README`, `@package.json`, `@docs/git-instructions.md`, and `@~/.claude/my-project-instructions.md` — arbitrary files, anywhere in the CLAUDE.md body, not just at the top.
- **AGENTS.md is explicitly the special case, not the general mechanism**: *"Claude Code reads `CLAUDE.md`, not `AGENTS.md`."* The documented bridge is exactly this same `@import` syntax (`@AGENTS.md`, with Claude-specific content appended below) or a symlink (`ln -s AGENTS.md CLAUDE.md`, Unix-only).
- **Recursive, depth-limited**: *"Imported files can recursively import other files, with a maximum depth of four hops."* **Correction from the first research pass**: no cycle detection is documented for `@import` specifically — the doc's only cycle-detection statement (*"circular symlinks are detected and handled gracefully"*) is about `.claude/rules/` symlinks, a different mechanism. The four-hop depth cap is the actual, confirmed safety bound for imports; don't assume cycle detection beyond that.
- **Path resolution**: both relative and absolute paths allowed; relative paths resolve **relative to the file containing the import**, not the working directory or repo root. Home-directory (`~`) paths are supported and are the documented way to share personal instructions across git worktrees (a gitignored `CLAUDE.local.md` doesn't follow you between worktrees; a `~`-path import does).
- **No context-cost saving**: *"Splitting into `@path` imports helps organization but does not reduce context, since imported files load at launch."* Imported content counts fully toward the ~200-line CLAUDE.md guidance — it just lives in a separate file.
- **New finding, missed by the first research pass: a one-time approval gate.** *"The first time Claude Code encounters external imports in a project, it shows an approval dialog listing the files. If you decline, the imports stay disabled and the dialog does not appear again."* This means a `CLAUDE.md` can contain a syntactically-valid `@import` that is **permanently inert** for a given user/machine if they declined it once — the file *says* it imports something, but the content may not actually be active in-session.
- **New finding, also missed: literal mentions are excluded, not just any `@word`.** *"Import parsing skips Markdown code spans and fenced code blocks. To mention a path in your CLAUDE.md without importing it, wrap it in backticks."* `` `@README` `` in backticks is inert text; `@README` outside backticks is a live import. Any detection logic has to make this distinction or it will misclassify prose that merely *mentions* an import syntax as if it were one.
- **Inspection**: `/memory` lists loaded CLAUDE.md, CLAUDE.local.md, and rules files — the doc's own wording doesn't explicitly confirm whether arbitrary `@`-imported non-CLAUDE.md files (a `README`, a `package.json`) are listed the same way; treat this as unconfirmed rather than assume `/memory` gives full import visibility.

**Skills, by contrast, have no equivalent import mechanism at all — confirmed by direct read of `skills.md` "Add supporting files."** Supporting files (`reference.md`, `examples.md`, `scripts/helper.py` — no mandated folder name, any layout works) are referenced from `SKILL.md` via **plain markdown links** (`[reference.md](reference.md)`), read by Claude only when it decides they're relevant — strictly lazy, never eagerly loaded the way a CLAUDE.md `@import` is. `SKILL.md` itself has its own, looser size guidance: *"Keep SKILL.md under 500 lines."* This means axis 4's specific tension (an eagerly-loaded, always-active import) **does not exist for Skills** — there's nothing analogous for `claudemd-tidy` to audit on the Skills side beyond what axis 1 already covers.

**Decision rule** (official): a repo supporting multiple AI coding tools keeps shared instructions in `AGENTS.md` (or any other shared file) and imports it into `CLAUDE.md`, rather than maintaining duplicated copies. More generally: anything genuinely needed every session belongs in a `CLAUDE.md` import; anything used only sometimes belongs in a Skill's lazily-loaded supporting file instead (this is really axis 1 again, applied to the choice of *how* to pull in extra content, not just whether to).

**Gap check against the current skill:** three distinct gaps, all unhandled today, sharing one root cause — the skill doesn't currently look for or follow any `@import`, AGENTS.md or otherwise:

(a) **Repo-level structural gap.** Step 2's target-finding (bullet 1) looks for `*CLAUDE.md`/`CLAUDE.md` and `CLAUDE.local.md`-style files; it never checks whether any found file contains `@import` lines, nor what they point to. A repo with an `AGENTS.md` (or any other file) that no `CLAUDE.md` imports has a real, concrete consequence beyond this skill: Claude Code itself never reads that content in *any* session. Two sub-cases: no `CLAUDE.md` exists at all (Claude Code has zero project-specific context even if other tools do, via `AGENTS.md` or otherwise), or a `CLAUDE.md` exists but doesn't import the sibling file (blindness, plus drift risk if both independently maintain overlapping rules).

(b) **Import-following gap.** When a `CLAUDE.md` *does* contain a live `@import` (correctly distinguished from a backtick-escaped mention, per the finding above), the imported content is exactly as "loaded every session" as anything written directly in `CLAUDE.md` — but Step 1's rule-loading and Step 2b's line-by-line interrogation only look at the literal `CLAUDE.md` file's own text today. The imported file's content is never audited, even though it's fully active in every session (assuming the approval-gate finding below doesn't apply).

(c) **Inert-import gap, entirely new — not in the first pass at all.** An import can be syntactically present and yet inactive if the user declined the one-time approval dialog. The skill has no way to detect this state directly (it isn't a file-content signal), but it should at minimum *know this possibility exists* rather than assume every written `@import` line is live — otherwise a CHALLENGE or audit finding could confidently reference imported content that isn't actually reaching the model in session.

---

## Proposed changes to `claudemd-tidy`

### 1. Axis 1 — sharpen RELOCATE's citation (low priority, optional)

**Proposed fix.** Add the official framing directly to the RELOCATE test in the Step 3 verdict table: *"...or content that only matters for one part of the codebase, or is used occasionally rather than every session (official Claude Code guidance: CLAUDE.md is for facts needed every session; Skills are for progressive-disclosure reference/procedure content)."* This doesn't change behavior — RELOCATE already catches these cases — it just gives the model a citable, sharper rule of thumb for borderline calls between "leave as COMPRESS" and "RELOCATE to a skill."

**Files:** `skills/claudemd-tidy/SKILL.md` (Step 3 verdict table, RELOCATE row).
**Bump:** PATCH (wording sharpened, no new capability).

### 2. Axis 2 — new Step 2b test: **Correctly-scoped?**

**Proposed fix.** Add a sixth Step 2b question: *"Correctly-scoped? Is this line's content actually about **this repo** — or is it a generic, cross-project working preference (→ belongs in user `~/.claude/CLAUDE.md`) or a personal-but-repo-specific preference not relevant to the rest of the team (→ belongs in `CLAUDE.local.md`, gitignored, create if missing)?"*

Routing: failing this test alone (the line is otherwise True/Live/Consistent/Actionable/non-redundant) is not "wrong," just misplaced — route to **CHALLENGE**, never straight to RELOCATE or DELETE, since only the user can confirm whether something is genuinely meant to be universal/personal versus deliberately pinned to this one project. Extend CHALLENGE's existing option set (`fix to match reality / keep — I'm missing context / delete`) with two more: *promote to global `~/.claude/CLAUDE.md`* / *move to `CLAUDE.local.md`*.

**Apply-phase consideration — this is the one genuinely new kind of write the skill would perform.** Every existing RELOCATE destination (`.claude/commands/`, `.claude/skills/`, `docs/`, `plans/`, `README.md`) lives *inside* the repo being tidied. "Promote to global `~/.claude/CLAUDE.md`" writes *outside* the repo, into a file shared across every project the user has. This is the same class of action `claudemd-tidy-reflect` already treats with extra care (Step 3: *"Never fork a private copy of those rules into the skill; propose the global edit and get explicit user confirmation, since that rulebook governs every session"*) — the same discipline should apply here: a promote-to-global resolution should be called out individually at confirmation time, not silently bundled into a bulk "approve all" the way an in-repo RELOCATE can be.

**Files:** `skills/claudemd-tidy/SKILL.md` (Step 2b new question, Step 3 CHALLENGE option list, Step 4 confirmation flow, Step 5 apply-phase — new destination case).
**Bump:** MINOR (new test, extends CHALLENGE's option set — not a new verdict, so not MAJOR by the suite's own table).

### 3. Axis 3 — new Step 2b test: **Rule-vs-memory?** (one direction only, deliberately)

**Proposed fix.** Add a seventh Step 2b question: *"Rule-vs-memory? Does this line read as a deliberate, team-decided instruction — or as a discovered pattern, debugging insight, or ad hoc noticed preference that reads more like something Claude learned than something the team decided?"*

Routing: same as Correctly-scoped? — failing this test alone routes to **CHALLENGE**, with the question surfaced as *"this reads like a discovered pattern rather than a deliberate rule — keep it here, or would this fit better in your memory system instead?"* **Deliberately no auto-apply and no attempt to write into `~/.claude/projects/<slug>/memory/` on the user's behalf.** Two reasons to stop at CHALLENGE and go no further:

- Memory files are explicitly documented as accumulating content that "routinely holds personal data" (this project's own global PRIMARY CHECK rule already says so) — writing into that location autonomously, even on request, adds a privacy-surface the skill doesn't currently touch anywhere else.
- The reverse direction — reading Memory to find content that should be *promoted* to CLAUDE.md — would require `claudemd-tidy` to read a completely new artifact class it has no relationship with today (Step 2's survey never touches Memory), and that promotion path is already officially documented as a manual, user-initiated action ("ask Claude to add this to CLAUDE.md, or edit the file yourself"). Not worth building a redundant path for something the platform already handles.

This keeps the check useful (flags miscategorized content) without expanding the skill's read/write surface into a new, more sensitive artifact type.

**Files:** `skills/claudemd-tidy/SKILL.md` (Step 2b new question, Step 3 CHALLENGE option list).
**Bump:** MINOR (new test, extends CHALLENGE — not a new verdict).

### 4. Axis 4 — detect and follow `@import` lines generally (AGENTS.md is one case, not the mechanism)

**Proposed fix (three parts, one item — all stem from the same root cause: the skill doesn't currently look for or follow any `@import`).**

**Part a — detect unimported sibling files.** Extend Step 2 bullet 1 to also check for `AGENTS.md` specifically (the one documented, named cross-tool convention worth checking for by default) at the repo root and nested. If found, check whether any `CLAUDE.md` in the repo imports it, via a live `@import` (not a backtick-escaped mention — see part c) or a symlink. Surface the result in the Step 4 plan as an explicit repo-level note, the same treatment already given to the `CLAUDE.local.md` finding: *"Found AGENTS.md at `<path>` — [not imported by any CLAUDE.md / imported by `<file>`]. Claude Code only reads CLAUDE.md directly; an unimported AGENTS.md is invisible to every Claude Code session, not just this tidy run."* Never auto-adds an import line — always surfaces as information at minimum, escalates to CHALLENGE when it looks like unintentional drift (both files exist, neither imports the other, grep shows overlapping rule text). Cheap enough to also run during `--report` mode.

**Part b — follow *any* live import during interrogation, not just AGENTS.md.** For every `@path` import found in a target `CLAUDE.md` (any file, not only `AGENTS.md`), treat the imported file's content as part of the "effective" CLAUDE.md for every subsequent step: Step 2b's line-by-line walk includes the imported file's lines, Step 3's verdicts can apply to blocks that physically live there, and Step 5's apply phase edits the imported file directly for any verdict landing on an imported block (never duplicating imported content back into `CLAUDE.md`). Respect the documented four-hop recursion limit — don't follow a chain deeper than that, and don't assume cycle safety beyond it since cycle detection isn't documented for imports (only for `.claude/rules/` symlinks, a different mechanism). This makes the general import pattern fully transparent to the skill, with `AGENTS.md`-importing as just one instance of it.

**Part c — don't misdetect a mention as an import, and don't assume every detected import is actually live.** Two correctness requirements for parts a/b, both surfaced by the corrected research: (1) exclude `@path` occurrences inside backticks or fenced code blocks — those are literal mentions, not imports, and treating them as one would misclassify prose as an active import; (2) an import can be syntactically present yet permanently inert if the user declined Claude Code's one-time approval dialog for it — the skill can't detect this state from file content alone, but any finding that leans on imported content should hedge accordingly (e.g., in a CHALLENGE, note "if this import was declined, this content may not actually be active in-session" rather than asserting it unconditionally).

**Files:** `skills/claudemd-tidy/SKILL.md` (Step 2 target-finding, Step 1/Step 2b import-following, Step 3/Step 5 — verdicts and edits can target an imported file).
**Bump:** MINOR (new detection + import-following capability, extends existing steps — no new verdict, no workflow-contract change).

---

## What this plan does not decide

- **Whether item 2's promote-to-global write needs to be an explicit seventh protected invariant**, or just documented apply-phase discipline (a note in Step 5, no invariant-gate escalation). The reflect skill's existing invariants list is specifically about *self-improvement* changing the *skill's own* behavior; this is the *tidy skill's normal operation* writing to a file outside the repo, a different category. Worth an explicit decision before implementing item 2, not assumed here.
- **Naming**: "Correctly-scoped?", "Rule-vs-memory?" are working names, not final — Step 2b's existing five questions (True/Live/Consistent/Actionable/Redundant-by-order) all read as single adjectives or a compact compound; these are a bit longer. Fine to bikeshed at implementation time, not a blocker.
- Whether item 1 is worth doing at all, given it's flagged as already-substantially-covered — included for completeness since the research surfaced it, not because it's clearly worth a release on its own.
- **Item 4's exact CHALLENGE-escalation threshold** ("looks like unintentional drift") isn't fully specified — "overlapping rule text" is doing a lot of work in that sentence and needs a concrete evidence rule (grep-based? semantic similarity? both files mentioning the same tool/command by name?) analogous to how DELETE's evidence rule was made concrete (cited duplicate location or grep-confirmed dead reference). Resolve at implementation time, don't hand-wave it into the shipped skill.
- **Whether `/memory`'s output actually lists arbitrary `@`-imported files** (a `README`, a `package.json`) the same way it lists CLAUDE.md/CLAUDE.local.md/rules files, or only the latter three — the doc's own wording doesn't settle this. If it doesn't, the skill has no first-party way to confirm an import actually resolved at runtime versus just being syntactically present; worth a quick empirical check (create a test import, run `/memory`, see what shows) before item 4's part (c) hedging language is finalized.
- **Axis 2's scope table is missing a tier**, noticed only during this pass's re-verification, not fixed here since it's outside today's ask: official docs list a fourth CLAUDE.md scope — **managed policy** (org-wide, deployed via MDM/settings, e.g. `/etc/claude-code/CLAUDE.md` or a `claudeMd` key in `managed-settings.json`), loaded before user and project CLAUDE.md and impossible to exclude. Axis 2's table only covered user/project/local. Not urgent (managed policy is an enterprise/IT concern, not something a repo-level tidy run would typically need to reason about), but worth folding in if axis 2 is revisited.

## Suggested execution order

Item 2 (Axis 2) is the one the user explicitly asked for first and the most concrete gap — do it first. Item 3 (Axis 3) is a natural companion, same shape of fix, no new invariant question attached to it (it never writes anywhere, CHALLENGE-only) — safe to do in the same pass as item 2 once the open question above is resolved. Item 4 (Axis 4, generalized from AGENTS.md-specific to any `@import`) is independent of items 2/3 — different files, different mechanism (target-finding + import-following, not a Step 2b line test) — and part (a) alone (detection + reporting, no import-following) is a smaller, self-contained slice worth shipping on its own; part (c)'s correctness requirements (backtick-exclusion, inert-import hedging) should land in the *same* change as part (b), not after, since part (b) is straightforwardly wrong without them (it would misclassify literal mentions as imports and overclaim inert content as active). Item 1 is optional polish, doable anytime, independent of everything else.
