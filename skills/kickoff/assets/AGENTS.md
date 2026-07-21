# {{TOPIC}} - learn-anything topic

This is the Codex- and Copilot-facing project instruction file for this topic repo. The
topic is content-only; the `learn-anything` plugin supplies skills, agents, and conventions.

- **Topic:** {{TOPIC}}
- **Learner:** {{LEARNER - role / relevant background, one line}}
- **Method:** {{PROCESS MIX - e.g. "source-anchored companion (theory) + project-first (practice)"}}
- **Pace:** {{PACE - e.g. "~6 h/week, target: <date/goal>"}}
- **Started:** {{YYYY-MM-DD}}
- **Status:** {{kickoff in progress | roadmap draft | roadmap approved | in-progress - module NN | module NN done}}

## Keep This File Honest

`AGENTS.md`, `CLAUDE.md`, and `roadmap.md` are the source of truth for this topic. Any
change to scope, versions, process mix, capstone, module order, or environment assumptions
must be recorded in the changelog before work diverges from the plan.

## Goal & Success Criteria

{{Why this, why now - one or two lines.}}

When I'm done, I will be able to:

- {{I can advise on ...}}
- {{I can build ... unaided}}
- {{I can solve ... -class problems}}

## Adjacent-Expertise Contrasts

Teach the new by contrast with what I already know deeply:

- {{new concept}} <-> {{the adjacent thing I know}} - {{what the contrast illuminates}}
- {{...}}

## Learner Level & Prior Attempts

- **Current level:** {{never touched | dabbled | working knowledge with gaps in ...}}
- **Prior attempts:** {{where it stalled last time, and why - or "none"}}

## Topic Properties

- Canonical source: {{name - quality take, or "none good"}}
- Field velocity: {{fast / moderate / settled}}
- Skill-like vs theory-like: {{...}}
- Verification looks like: {{tests pass / cluster converges / proof checks / sympy confirms / ...}}

## Process Mix + Justification

{{Which archetype(s) and why - tied to the topic properties and prior-art verdict.}}

## Capstone Spec

{{Chosen capstone + a short acceptance sketch of the finished thing; or "No capstone - <why>".}}

## Prior-Art Verdict + Justification

- **Verdict:** {{adopt | adapt | generate}}
- **Sources considered:** {{named - CS50 / MIT OCW xx / roadmap.sh / book edition / cert blueprint}}
- **Why:** {{what the prior art covers well, where it's stale or off-goal, why the verdict follows}}

## Plugin Dependency

- **Plugin:** `learn-anything`
- **Marketplace repository:** `oswalth/learn-anything`
- **Minimum version:** {{X.Y.Z}}
- Restore from GitHub with either tool:

```bash
codex plugin marketplace add oswalth/learn-anything
codex plugin add learn-anything@learn-anything

copilot plugin marketplace add oswalth/learn-anything
copilot plugin install learn-anything@learn-anything
```

## Decisions

- {{Key decisions made during kickoff and after - one line each.}}

## Changelog

- **{{YYYY-MM-DD}}** - Kickoff: goal, level, contrasts, process mix, prior-art verdict,
  environment, and baseline established; roadmap drafted.
