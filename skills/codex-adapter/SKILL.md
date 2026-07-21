---
name: codex-adapter
description: >-
  Codex compatibility layer for the learn-anything plugin. Load this when running any
  learn-anything workflow under Codex, when a skill mentions Claude-specific concepts such
  as CLAUDE.md, Agent tool, WebSearch/WebFetch, opus/sonnet/haiku, or .claude-plugin, or
  when installing/testing the plugin with Codex.
---

# Codex adapter for learn-anything

This is a translation layer. The learning-system content is shared with Claude Code, but
Codex uses different names for a few durable surfaces.

## Translate these terms

- `CLAUDE.md` project instructions -> `AGENTS.md` in Codex topic repos. If both files exist,
  read both; treat `AGENTS.md` as the Codex-facing instruction file and `CLAUDE.md` as the
  compatibility/source-of-truth topic record unless they conflict.
- `.claude-plugin/plugin.json` -> `.codex-plugin/plugin.json` for Codex packaging.
- `.claude-plugin/marketplace.json` -> `.agents/plugins/marketplace.json` for the local
  Codex marketplace in this repo.
- Relative skill links resolve within the installed Codex plugin or this checkout.
- `Agent tool` / Claude subagent -> the matching Codex role in `.codex/agents/*.toml`.
- `WebSearch` / `WebFetch` -> Codex web search/browser-backed verification. For volatile
  facts, run Codex with `--search`.
- `opus`, `sonnet`, `haiku` -> Claude model tiers. In Codex, use the current default model
  unless the user explicitly chooses an available Codex model.
- `disable-model-invocation: false` on side-effecting skills -> Codex-compatible replacement
  for the old Claude hard trigger flag. Treat "User-invoked only" in descriptions as binding:
  ask for explicit user intent before mutating files, spending substantial tokens, or doing
  broad live web research.

## Topic repo behavior under Codex

When `/kickoff` or the kickoff skill creates a topic repo under Codex, create both:

- `CLAUDE.md` for compatibility with the original topic format.
- `AGENTS.md` for Codex project instructions.

Both should carry the same topic facts: goal, learner profile, success criteria,
adjacent-expertise contrasts, process mix, capstone, plugin dependency, decisions, and
changelog. Keep them in sync when topic-level facts change.

## Existing skills

Do not rewrite a shared skill only to rename Claude terms. Prefer this translation layer
unless the old wording would make Codex execute the wrong behavior.

Behavioral edits still follow the repo's lean `/evolve` discipline: minimal scope, one edit
pass, one structural check, synchronized manifests, version bump, and changelog entry.
