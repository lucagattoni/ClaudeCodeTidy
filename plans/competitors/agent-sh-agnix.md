# agent-sh/agnix

**URL:** https://github.com/agent-sh/agnix
**Stars (2026-07-03):** 316
**Category:** Adjacent (correctness linting, not bloat/slimming)
**License:** dual MIT/Apache-2.0
**Distribution:** npm (`agnix`), crates.io (`agnix-cli`), GitHub Releases, VS Code + JetBrains + Neovim + Zed extensions, GitHub Action

## What it is

"The missing linter and LSP for AI coding assistants" — validates CLAUDE.md, AGENTS.md, SKILL.md, hooks, and MCP configs against 432 rules across Claude Code, Codex CLI, OpenCode, Cursor, Copilot, and other agent tooling. Positioned to "catch broken agent configs before your AI tools silently ignore them."

## What it does

- **432 validation rules** spanning multiple agent-config formats and tools, not just Claude Code.
- **Autofix** for a subset of detected issues.
- **IDE integration** — VS Code and JetBrains plugins, a Neovim plugin, a Zed extension, plus a browser-based playground for trying it without installing anything.
- **CI integration** — a published GitHub Action (`agnix-ci`) to fail a build on broken agent configs.
- Listed in the community-maintained `awesome-claude-code` list.

## Problem framing

Agent config files (CLAUDE.md, SKILL.md, hooks.json, MCP server configs) can be *structurally* broken — malformed syntax, dangling references, invalid schema — in ways the AI tool doesn't surface as an error, it just silently ignores the broken part. agnix's job is to catch that before it causes silent behavior drift.

## Relation to TidyClaudeMD

**Orthogonal, not competing.** agnix asks "is this file syntactically/structurally valid," TidyClaudeMD asks "is this file's content bloated, stale, or contradictory." A CLAUDE.md can be perfectly valid per agnix's 432 rules while still being 900 lines of dead instructions (TidyClaudeMD's problem), and a CLAUDE.md can be a tight, well-curated 80 lines while still having a broken hook reference agnix would catch and TidyClaudeMD's own comprehension-gate survey might or might not notice depending on whether that reference is a "claim" its claim-driven verification checks (see `v1.3.0-candidate-improvements.md`, item 2).

## What's worth noting (complementary tool, not a feature to copy)

Given the non-overlapping scope, the natural relationship is **recommend, not replicate**: TidyClaudeMD's README or a future skill step could point users to agnix for structural/correctness validation as a companion pass, distinct from and complementary to a content-quality tidy — rather than TidyClaudeMD attempting to grow its own linting-rule corpus for syntax/schema correctness, which is a large, format-fragmented problem agnix already covers across six+ tools.
