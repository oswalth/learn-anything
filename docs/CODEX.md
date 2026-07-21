# Using `learn-anything` With Codex

This repo is a dual-target plugin: Claude Code support remains in `.claude-plugin/`, and
Codex support lives beside it in `.codex-plugin/`, `.codex/`, `.agents/`, and `AGENTS.md`.

## Install locally from this checkout

From anywhere:

```bash
codex plugin marketplace add /Users/vladimircarpa/pets/learn-anything
codex plugin list --marketplace learn-anything-local --available --json
codex plugin add learn-anything@learn-anything-local
```

Start a new Codex thread after installing so the plugin skills are discovered.

If you change this plugin locally, refresh and reinstall:

```bash
codex plugin marketplace upgrade learn-anything-local
codex plugin add learn-anything@learn-anything-local
```

Then start a new thread again.

## Start a new learning topic

Create an empty topic repo, then run Codex with web search enabled because kickoff and
freshness checks rely on current sources:

```bash
mkdir -p ~/learn-topics/golang
cd ~/learn-topics/golang
codex --search
```

In Codex, ask:

```text
Use the learn-anything kickoff skill to start a new topic for Go.
```

Codex should interview you, research prior art, draft `CLAUDE.md`, `AGENTS.md`,
`BASELINE.md`, `ENVIRONMENT.md`, and `roadmap.md`, then stop for roadmap approval.

## Continue a topic

Use natural language and name the skill when you want deterministic routing:

```text
Use learn-anything plan-module for module 1.
```

```text
Use learn-anything author for section 1.1 with the default rigor.
```

```text
Walk me through section 1.1 using learn-anything study.
```

```text
Check my work for section 1.1 against validation.md.
```

```text
Quiz me on section 1.1.
```

```text
Run due recall cards.
```

For one-shot terminal runs:

```bash
codex exec --search -C ~/learn-topics/golang "Use learn-anything recall for due cards."
```

## Use the Codex agents directly

The Codex agent definitions live in `.codex/agents/`. They mirror the Claude subagents:

- `section-drafter`
- `roadmap-drafter`
- `roadmap-judge`
- `critic-accuracy`
- `critic-freshness`
- `critic-depth`
- `critic-pedagogy`

In normal use, the skills should orchestrate these roles. For debugging, you can ask Codex
to use one by name:

```text
Use the critic-depth agent to review modules/01-foundations/sections/01-intro/theory.md
against the learn-anything depth standard.
```

## Current compatibility notes

- The shared `skills/` directory still contains some Claude-era names and environment
  variables. Use `skills/codex-adapter/SKILL.md` as the translation layer.
- The plugin is intentionally dual-target. Do not delete the Claude files unless the repo
  deliberately drops Claude Code support later.
- Prefer `codex --search` for kickoff, authoring, freshness, baseline updates, and any
  topic that depends on current docs, versions, pricing, APIs, or public curricula.
