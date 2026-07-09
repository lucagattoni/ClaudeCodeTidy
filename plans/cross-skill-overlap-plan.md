# Cross-skill overlap/conflict detection — plan

Status: **preliminary — under adversarial review**. Written 2026-07-09 against TidyClaudeMD v0.20.2. Implements idea #2 from `competitive-landscape-2026-07-03.md` ("Features worth prototyping"): the `--skills` class currently interrogates each `SKILL.md` in isolation; nothing catches two skills that duplicate, collide, or trigger on the same requests. Two independent projects built this (evidence it earns its cost) — their mechanics were read at source level on 2026-07-09:

- **claudoctor** (`src/lib/analyze.ts`): four mechanical tiers — *duplicates* (identical `contentHash` across ≥2 paths), *near-duplicates* (identical `bodyHash`, i.e. same body after stripping frontmatter, but different `contentHash` — "frontmatter drift"), *name conflicts* (same `name`, different content **and** different body), *overlap* (Jaccard similarity ≥0.5 on stopword-filtered `name + description` token sets; `--deep` also compares bodies). Reports wasted tokens as `tokens × (copies − 1)`.
- **pulser** (`src/rules/conflicts.ts`): extracts **quoted** trigger phrases from each skill's `description` and warns when the same phrase appears in ≥2 skills — "overlapping trigger keywords cause Claude to invoke the wrong skill."

## Goal

Extend `claudemd-tidy`'s `--skills` class with a **sweep-level** analysis pass: after the existing per-file interrogation, compare every pair of skills in the sweep for duplication, name collision, and trigger overlap — and route findings through the suite's existing verdict/evidence machinery, not a parallel one.

## Design

### The four checks (Step 2, new sweep-level bullet)

Run once per `--skills` sweep, over all skills in scope (project `.claude/skills/`; plus `~/.claude/skills/` when combined with `--user`):

| Check | Mechanics | Evidence grade |
|---|---|---|
| **Duplicate?** | Byte-identical `SKILL.md` content at ≥2 paths — `shasum` over each file, group by hash | Mechanical, grep-confirmable |
| **Near-duplicate?** | Identical body but different frontmatter — `shasum` over content with the YAML frontmatter block stripped (`awk` past the second `---`), grouped; flag groups whose full-content hashes differ | Mechanical |
| **Name-conflict?** | Same frontmatter `name:` in ≥2 skills whose content differs — matters because the name is the invocation key (`/name`), so two different skills competing for one name is a real collision, not just waste | Mechanical |
| **Trigger-overlap?** | Two skills whose `description`s would plausibly fire on the same user request. Mechanical assist first (pulser-style: quoted phrases appearing in ≥2 descriptions; plus high word-overlap between descriptions), then LLM judgment on the candidates — this suite runs *as* an LLM, so it can judge "would these two descriptions race for the same prompt?" directly instead of approximating with Jaccard the way claudoctor must | Judgment, mechanically pre-filtered |

All four are cheap: hashing is one `shasum` pass over files already inventoried by Step 2, and the pairwise judgment only runs on mechanically pre-filtered candidates, not all N² pairs.

### Verdict routing (Step 3)

Consistent with the suite's existing philosophy — mechanical evidence never auto-deletes a *skill*, because removing a whole skill is a scope/intent decision, not a dedup line-edit:

- **Duplicate** → **CHALLENGE**, evidence-cited (both paths + matching hash): "these two files are byte-identical — which is the canonical home?" Removing one is DELETE-grade *evidence*, but *which* copy survives is user-only. Follows the sweep-level CHALLENGE precedent set by the AGENTS.md visibility check (v0.11.0).
- **Near-duplicate** → **CHALLENGE**, citing the diff of the two frontmatters: "same procedure, two names/descriptions — intentional variant or drift?"
- **Name-conflict** → **CHALLENGE**, tier-1 stakes (it affects which skill actually runs when invoked).
- **Trigger-overlap** → **CHALLENGE**, minor tier unless one of the pair is safety-relevant: "these two descriptions plausibly fire on the same request — differentiate, merge, or intentional?"

The suite's own two skills (`claudemd-tidy`, `claudemd-tidy-reflect`) remain excluded as **edit targets**, but participate as **comparison references** — a third-party skill that duplicates one of them should be flagged, with the finding landing on the non-suite copy.

### Report mode

The three mechanical checks (duplicate/near-duplicate/name-conflict counts) join `--report`'s mechanical-checks tier when composed with `--skills` — they're scriptable and judgment-free, exactly what that tier is for. Trigger-overlap judgment stays out of report mode (it's judgment, and report mode skips interrogation by design).

### Run log

No format change needed: sweep-level findings go under **Instructions exercised** (e.g. "CHALLENGE ×2: sweep-level Duplicate? check") like any other verdict source, and CHALLENGE resolutions land in **User feedback** as usual.

## Files & bump

- `skills/claudemd-tidy/SKILL.md`: Step 2 — new sweep-level bullet for the `--skills` class (the four checks, mechanics, pre-filter rule); Step 3 — routing note (sweep-level CHALLENGEs, suite-skills-as-references rule); Report mode — one clause adding the mechanical trio; Target classes table — one sentence in the Skills row pointing at the sweep-level pass.
- `docs/reference.md`: Target classes → skills-class tests paragraph gains the sweep-level checks.
- `README.md`: the `--skills` fragment of the Scope bullet mentions cross-skill detection.
- `CHANGELOG.md` + version bump: **MINOR** (new capability), with git tag + GitHub release in the same pass per the v0.20.2 rule.

## Verification

Build a disposable fixture repo (scratchpad) with a skills directory containing: an exact duplicate pair, a near-duplicate pair (same body, different `name:`/`description:`), a name-conflict pair (same `name:`, different bodies), one trigger-overlap pair (distinct wording, same plausible trigger), and one clean control skill. Run `--skills` against it and confirm: all four planted findings surface as CHALLENGEs with correct evidence citations, the control skill produces none, and `--report --skills` surfaces exactly the three mechanical counts. Record the run log as normal — it doubles as the first real multi-skill-class evidence for the reflect loop.

## What this plan does not decide

- **Plugin/marketplace cache sweeping** (landscape idea: claudoctor scans `~/.claude/plugins/cache/`) — a different location set with different semantics (cached copies are *expected* duplicates of installed versions); separate plan if pursued.
- **Exact tokenization** for waste estimates — findings cite line/byte counts for now; upgrading to real token counts is landscape idea #5, orthogonal.
- **Cross-agent directories** (`~/.codex/`, `~/.cursor/rules/`) — landscape idea #4, out of scope here.
- **Auto-merge of near-duplicates** — claudoctor has it on its own roadmap; this plan deliberately stops at CHALLENGE, per the suite's user-only-decisions principle.

## Iteration log

- 2026-07-09 — preliminary draft (pass 0), pre-review.
