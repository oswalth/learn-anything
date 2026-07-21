---
name: copilot-adapter
description: >-
  GitHub Copilot compatibility layer for learn-anything. Load when a shared skill mentions
  Claude-specific agent tools or project instruction files while running under Copilot in
  VS Code or Copilot CLI.
user-invocable: false
---

# Copilot adapter

Translate shared workflow terms without duplicating the command logic:

- Treat topic `CLAUDE.md` as the portable source-of-truth record. Also create and maintain `AGENTS.md` so Copilot receives topic instructions automatically.
- Resolve relative skill links within the installed Copilot plugin or this checkout.
- Map `Agent tool` to the matching agent in `.github/agents/`.
- Map `WebSearch` and `WebFetch` to Copilot web tools when available.
- Use the current Copilot model; ignore Claude model tier names.
- Treat user-invoked-only and confirmation gates in shared skills as binding.

Plugin changes follow `/evolve`: one scoped edit pass, one structural check, synchronized Claude/Codex/Copilot manifests, and one changelog entry.
