# {{TOPIC}} — learn-anything topic

- **Topic:** {{TOPIC}}
- **Learner:** {{LEARNER — role / relevant background, one line}}
- **Method:** {{PROCESS MIX — e.g. "source-anchored companion (theory) + project-first (practice)"}}
- **Pace:** {{PACE — e.g. "~6 h/week, target: <date/goal>"}}
- **Started:** {{YYYY-MM-DD}}
- **Status:** {{kickoff in progress | roadmap draft | roadmap approved | in-progress — module NN | module NN done}}

> ### ⚠️ Keep this file honest
> This `CLAUDE.md` (with `roadmap.md`) is the **single source of truth** for this topic.
> **Any** change — reordering/adding/removing/merging modules, swapping a tool or version,
> changing scope, the process mix, or the capstone — **must be recorded in the
> [Changelog](#changelog)** (date · what · why) **before** the work diverges from the
> document. If generating or validating reveals that the plan should change, stop and
> update this file first.

---

## Goal & success criteria

{{Why this, why now — one or two lines.}}

When I'm done, I will be able to (capabilities):
- {{I can advise on … }}
- {{I can build … unaided}}
- {{I can solve … -class problems}}

## Adjacent-expertise contrasts to lean on

Teach the new by contrast with what I already know deeply:
- {{new concept}} ↔ {{the adjacent thing I know}} — {{what the contrast illuminates}}
- {{…}}

## Learner level & prior attempts

- **Current level:** {{never touched | dabbled | working knowledge with gaps in …}}
- **Prior attempts:** {{where it stalled last time, and why — or "none"}}

## Topic properties

- Canonical source: {{name — quality take, or "none good"}}
- Field velocity: {{fast / moderate / settled}}
- Skill-like vs theory-like: {{…}}
- Verification looks like: {{tests pass / cluster converges / proof checks / sympy confirms / …}}

## Process mix + justification

{{Which archetype(s) and why — tied to the topic properties and prior-art verdict.}}

## Capstone spec

{{Chosen capstone + a short acceptance sketch of the finished thing; or "No capstone — <why>".}}

## Prior-art verdict + justification

- **Verdict:** {{adopt | adapt | generate}}
- **Sources considered:** {{named — CS50 / MIT OCW xx / roadmap.sh / book edition / cert blueprint}}
- **Why:** {{what the prior art covers well · where it's stale or off-goal · why the verdict follows}}

## Plugin dependency

- **Plugin:** `learn-anything` — marketplace repo `oswalth/learn-anything`
- **Minimum version:** {{X.Y.Z}} — the plugin version this topic was created against;
  `/author` and `/plan-module` warn if the installed plugin is older.
- This repo is content-only. The plugin supplies all skills, agents, and conventions.
- **Restore with Claude Code:** add marketplace `oswalth/learn-anything`, then install
  `learn-anything@learn-anything` with `/plugin`.
- **Restore with Codex:** add marketplace `oswalth/learn-anything`, then add
  `learn-anything@learn-anything` with `codex plugin`.
- **Restore with Copilot CLI:** add marketplace `oswalth/learn-anything`, then install
  `learn-anything@learn-anything` with `copilot plugin`.

## Decisions

- {{Key decisions made during kickoff and after — one line each.}}

## Changelog

- **{{YYYY-MM-DD}}** — Kickoff: goal, level, contrasts, process mix, prior-art verdict,
  environment, and baseline established; roadmap drafted.
