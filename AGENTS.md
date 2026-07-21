# AGENTS.md - Codex guidance for the `learn-anything` plugin repo

This checkout is the source repo for the `learn-anything` learning-system plugin. It now
supports both Claude Code and Codex from one repository.

## First principles

- Read `CLAUDE.md` before changing the plugin. It remains the full maintenance manual and
  the source of the repo's versioning, changelog, blast-radius, and `/evolve` discipline.
- Treat this file as the Codex-facing overlay, not a replacement for `CLAUDE.md`.
- Every plugin behavior or documentation change still needs a semver bump in both plugin
  manifests and a `CHANGELOG.md` entry.
- Keep Claude and Codex support in one repo unless a future incompatibility makes that
  impossible.

## Codex-specific files

- `.codex-plugin/plugin.json` is the Codex plugin manifest. It points at the shared
  `skills/` directory.
- `.agents/plugins/marketplace.json` is a repo-local Codex marketplace for local testing.
- `.codex/agents/*.toml` are Codex-native versions of the drafter, judge, and critic roles.
- `skills/codex-adapter/SKILL.md` explains how to interpret remaining Claude-specific terms
  when running these workflows under Codex.
- `skills/kickoff/assets/AGENTS.md` is the Codex project-instruction template for generated
  topic repos. The existing `skills/kickoff/assets/CLAUDE.md` remains for Claude Code.
- `docs/CODEX.md` is the user-facing quickstart for installing and running the plugin with
  Codex.

## Working style

- Preserve the existing learning-system content unless the task is explicitly to change the
  pedagogy or workflow behavior.
- Prefer small compatibility overlays over broad rewrites. The current skill bodies are
  valuable domain logic; make them more Codex-native gradually.
- The old Claude `disable-model-invocation: true` flag is intentionally not used: Codex
  validation rejects it. Side-effecting skills must say "user-invoked only" in their
  descriptions and must confirm intent before expensive or mutating work.
- When a skill mentions Claude-specific mechanics, translate them this way:
  - `CLAUDE.md` project instructions -> `AGENTS.md` project instructions in Codex topics.
  - Claude plugin root variables -> locate the installed plugin root or current checkout.
  - Claude subagents in `agents/*.md` -> Codex agents in `.codex/agents/*.toml`.
  - `WebSearch`/`WebFetch` -> Codex web search or browser-backed verification when enabled.
  - Claude model names -> use the current Codex model/default unless the user explicitly
    asks for a different available model.
- Do not remove or rename Claude files while Codex support is being added. Compatibility is
  the goal.
