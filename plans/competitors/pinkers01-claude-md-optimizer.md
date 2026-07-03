# Pinkers01/claude-md-optimizer

**URL:** https://github.com/Pinkers01/claude-md-optimizer
**Stars (2026-07-03):** 0
**Category:** Direct competitor (different paradigm — standalone GUI app, not a Claude Code skill)
**License:** MIT (source repo); hosted product (signed builds, payment, license server) is proprietary
**Built by:** Pinky Creative Studio

## What it is

An interactive, client-side optimizer for CLAUDE.md and other LLM context files — packaged as a macOS `.app`, a Windows `.exe`, or a single standalone HTML file (`optimizer.html`) runnable in any modern browser. Not a Claude Code skill and requires no agent session to use.

## Problem framing

Long-running Claude Code sessions accrete rules — preferences, anti-patterns, fixes added one at a time. After ~6 months a typical CLAUDE.md crosses a "40k-character performance ceiling," the agent starts thrashing context, and rules silently contradict each other. The tool's pitch: "a single click" instead of manual cleanup.

## What it does

1. Loads a CLAUDE.md file (drag-and-drop or path paste).
2. Splits it into sections.
3. Detects duplicates and rule conflicts.
4. Lets the user decide per section: **keep / move to a memory file / drop**.
5. Exports a ZIP: a slim `CLAUDE_NEW.md`, individual `memory/*.md` files, and an `OPTIMIZATION_REPORT.md`.

## Distinctive features

- **100% client-side** — no data leaves the browser; explicitly marketed as a privacy feature.
- **Section-level keep/move/delete** with a live character counter and a 35k-character target.
- **Duplicate detection** — both line-level and key-phrase level, across sections.
- **Conflict detection** — ships a small library of **hardcoded common rule-conflict patterns** to check against explicitly: payments stack (Stripe vs. Mollie), em-dash policy, autonomy vs. ask-first, lite vs. full mode, paid-API rules, hosting-folder rules.
- **Search/filter** by status (Keep / Memory / Delete / With duplicates / With conflicts).
- **Per-section editing** — edit textarea inline, "shrink by 50%" action, revert to original.
- **Multilingual UI** (PL / EN / NL), dark mode default, no telemetry.

## Pricing model

Source repo (parsing/UI/export logic) is MIT and free. A hosted version adds cloud sync, a license, and updates for €9 lifetime via a separate storefront. The free/open vs. paid/hosted split is on packaging and convenience, not on the core optimization logic.

## Tech stack

Vanilla HTML/CSS/JS (no framework), JSZip for export, a Python parser (`parser.py`) for the initial section split, `osacompile` for the macOS build, PyInstaller (via GitHub Actions) for the Windows build.

## Gaps relative to TidyClaudeMD

- **No repo awareness at all.** It only ever sees pasted/loaded file text — no git repo, no comprehension gate, no visibility/PRIMARY-CHECK-style safety gate, no way to verify a line's claims (a file path, a command) against the actual codebase. Its True/Live-equivalent checks are structurally impossible for this tool's design.
- Conflict detection is a fixed, hardcoded pattern library (specific to the author's own recurring situations — e.g. "Stripe vs Mollie") rather than a general contradiction-detection method; it will miss any conflict shape not on that list.
- No evidence-gated deletion, no self-improvement loop, no run-record mechanism.

## What's still worth noting

- **Contradiction detection as an explicit, named feature** (distinct from duplicate detection) is something TidyClaudeMD's CHALLENGE verdict covers only implicitly, as one of four line-level tests, without a dedicated pass or a seed list of common contradiction *shapes* (not specific content) to check for.
- Zero-install, zero-agent-session accessibility is a meaningfully different distribution model — useful for users without Claude Code installed, or for cleaning any LLM instruction file regardless of tool.
