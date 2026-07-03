## CLAUDE.md hygiene — keep every CLAUDE.md slim

Applies to **every** `CLAUDE.md` I create or edit (global, project, nested). A CLAUDE.md is a rulebook loaded into context on every session — each line costs attention, so it must earn its place. Slim it by **relocating** content, never by losing it.

1. **Behavior test.** Every line must change what I'd otherwise do. Cut background, descriptions, and anything derivable from the repo itself (file tree, code style, git history, package manifests) — or replace it with a one-line pointer.
2. **One home per rule.** Each rule lives in exactly one file; everything else links to it. Detailed procedures and step-by-step workflows belong in the skill/command that runs them or in `docs/` — CLAUDE.md keeps only the pointer plus a one-line summary of when it applies.
3. **Relocate, don't delete.** When slimming, move detail to `docs/`, `README.md`, or the relevant skill and link it from where it was. Only delete what is duplicated, superseded, or verifiably wrong — and verify first (grep for the referenced file/command; "renamed" is not "gone").
4. **Compress the writing.** Imperative bullets and tables over prose paragraphs. State the rule once; keep a "why" only when the rule is counterintuitive enough that omitting the reason invites a wrong "fix." At most one example per convention, and only where the format itself is the rule (e.g. a filename pattern).
5. **Prune on touch.** Whenever editing any CLAUDE.md, tidy the surrounding content in the same change: drop rules referencing files/commands that no longer exist, merge overlapping rules, collapse rambling sections.
6. **Size guardrail.** A project CLAUDE.md should stay under ~150 lines. Crossing it is the trigger to extract sections (per rules 2–3) and leave pointers — not a hard cap that justifies deleting information.
7. **Structure for scanning.** Stable `##` sections grouped by topic, rules as bullets under them, most safety-critical rules first. No decorative headers for a single bullet.
