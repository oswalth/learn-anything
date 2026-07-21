---
name: retro
description: >-
  Run a retrospective on THIS topic — interview the learner about what's working and what
  hurts, then propose concrete diffs. Topic-level fixes (roadmap edits, process changes,
  per-topic overrides) are applied directly with a changelog entry; plugin-level ideas are
  emitted as a ready-to-paste /evolve brief, never applied here. Use when the user says
  "retro", "this isn't working", "the sections are too long", or runs /retro. User-invoked.
disable-model-invocation: false
---

# /retro — improve THIS topic (and route plugin ideas to /evolve)

Interview the learner about how the topic is going, then turn the feedback into concrete
changes. Crucial split: **topic-level** changes you apply here; **plugin-level** changes you
do **not** apply — you hand back an `/evolve` brief. `/retro` and `/evolve` are the only
sanctioned mutation paths, and both end in changelog entries.

## Read (context discipline)
- Topic `CLAUDE.md` (process mix, decisions, changelog), `roadmap.md` (statuses/structure).
- `retention/log.md` and a glance at the deck (are cards lapsing? which concepts?).
- Only the specific section files a complaint points at.

## Interview
Ask, one at a time, adapting to the topic:
- "What's working well enough that we should keep it?"
- "What's actively hurting — pacing, depth, difficulty, the practice style, the retention
  load?"
- "Where are you spending time that doesn't feel like learning?"
- "Are sections the right size? Too dense, too thin, too long for a sitting?"
- "Is the roadmap still the right path to your goal, or has the goal shifted?"

## Classify each piece of feedback
- **Topic-level** — specific to this topic: roadmap edits (merge/split/reorder/add/remove
  modules), process-mix change, capstone change, or a **per-topic override** (a project-level
  `.claude/skills/<name>/SKILL.md` or `.claude/agents/<name>.md` in this repo that shadows the
  plugin's, for a topic-specific behavior).
- **Plugin-level** — a change to a convention, a command's default behavior, or a critic that
  would help *every* topic, not just this one.

## Apply topic-level changes (here)
- Make the edit (regenerate `roadmap.md`, adjust the process mix in `CLAUDE.md`, or create the
  override file under this repo's `.claude/`).
- **Record it** in the `CLAUDE.md` changelog (date · what · why). Post-approval roadmap
  changes *require* this entry.
- If a change invalidates already-generated sections, say so and offer to re-`/author` them.

## Route plugin-level changes (do NOT apply)
For each plugin-level idea, emit a **ready-to-paste `/evolve` brief**:
```
/evolve
Goal: <what should change and why, in the learner's terms>
Motivation: <the concrete pain from this topic that prompted it>
Suspected scope: <which convention/command/critic seems involved — a guess, /evolve confirms>
Originating topic: <this topic's name/path>
```
Tell the learner to run it from the plugin checkout. Do not edit the plugin from here — a
topic repo has no authority to mutate shared machinery. If an override you just made locally
proves generally useful, note that promoting it into the plugin is exactly what the `/evolve`
brief is for.

## Finish
Summarize what you changed here and what you routed to `/evolve`.
