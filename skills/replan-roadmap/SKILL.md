---
name: replan-roadmap
description: >-
  Regenerate an existing topic's roadmap.md with one roadmap-drafter pass when the goal,
  environment, or plugin conventions have moved on since /kickoff
  (e.g. "I already have the AWS account now", "change my goal to X", "refresh the roadmap
  with the plugin's latest process"). Protects modules already
  approved/planned/in-progress/done by default — only `draft` modules and net-new modules
  are freely redesigned; any conflict with a protected module is surfaced as a diff for the
  learner to resolve, never silently applied. Use when the learner says "regenerate the
  roadmap", "redo the roadmap", "update the roadmap for my new goal/environment", or runs
  /replan-roadmap. Re-gates at approval like /kickoff's roadmap step. Side-effect command
  (web research, mutates roadmap.md/CLAUDE.md) — user-invoked only.
disable-model-invocation: false
argument-hint: "[what changed, optional — e.g. \"AWS account already created\", \"goal is now X\", \"just refresh with latest conventions\"]"
---

# /replan-roadmap — regenerate the roadmap against updated inputs

Re-draft `roadmap.md` with one roadmap-drafter pass against the topic's *current* inputs,
while protecting modules the learner has already acted on. This is a **structure**-level
command: it never touches section content,
`workspace/`, or `retention/`.

## When to use vs the other regeneration paths
- Roadmap scope/goal/module set changed → **this command**.
- One approved module's section boundaries need to change, but the roadmap entry itself is
  fine → `/plan-module N` (re-plan).
- One claim in an already-generated section is wrong/outdated/thin → `/patch N.K`.
- A whole section's content needs a full re-draft + re-critique → `/author N.K` (or `/author
  N`), which regenerates on re-run — see its own confirm-before-overwrite gate.

## Preconditions (check, then stop if unmet)
- Must run inside an **existing topic repo** — `CLAUDE.md` and `roadmap.md` both present. If
  not, tell the learner this is a topic-repo command, not `/kickoff` (which is for brand-new
  topics and refuses on an existing one).
- **Plugin compatibility (soft).** Same check as `/author`/`/plan-module`: compare the topic
  `CLAUDE.md`'s recorded **Minimum version** to the active tool's plugin manifest; warn
  (don't block) if the installed plugin is older.

## Read (context discipline — only these)
- Topic `CLAUDE.md`, `roadmap.md`, `modules/README.md` (the status board).
- `BASELINE.md`/`ENVIRONMENT.md` **only** if the stated driver (Step 1) touches versions or
  the environment/budget.
- The [conventions](../conventions/SKILL.md), inlined for the drafter — agents run from the
  topic cwd and cannot open plugin files by relative path.
- Do **not** read section content files, `workspace/`, or `retention/` — this command never
  touches generated content.

## Step 1 — capture what changed
- If `$ARGUMENTS` is given, treat it as the stated driver. Otherwise ask: is this a
  goal/scope change, an environment change (an account/tool/budget that's now different),
  or just a "refresh with the freshest process" with no input change (e.g. after a plugin
  update)?
- If the driver changes a `CLAUDE.md` fact — goal, success criteria, adjacent-expertise
  contrasts, process mix, capstone spec, environment/budget — make that **targeted** edit to
  `CLAUDE.md` now (a surgical edit, not a full `/kickoff` re-interview), and record it in the
  `CLAUDE.md` changelog per the "keep this file honest" rule, **before** regenerating the
  roadmap (the drafters read `CLAUDE.md` as an input).
- If the driver is a pure "freshen the process" refresh, no `CLAUDE.md` edit is needed;
  regeneration proceeds against the current `CLAUDE.md` and the plugin's current
  conventions.

## Step 2 — classify current modules
From `roadmap.md`, list every module with its status:
- **Protected** — anything not `draft` (`approved`/`planned`/`in-progress`/`done`).
- **Free** — status `draft`.
Show this classification to the learner before spawning any agent, so they know upfront
what's off-limits by default.

## Step 3 — draft once
Invoke **one `roadmap-drafter` agent**, passing: the current
`CLAUDE.md` (post any Step-1 edit), the prior-art verdict from it, the conventions text
inlined, **plus regeneration-mode inputs**:
- The full current `roadmap.md` body.
- The Protected/Free classification from Step 2.
- This instruction, verbatim in spirit: *"Modules listed as PROTECTED must appear in your
  draft with Concept/Skills/Capstone increment/Prerequisites/Source UNCHANGED and in the
  same relative order. Freely design, reorder, or add FREE (draft) modules around them. If a
  PROTECTED module genuinely conflicts with the new input, do NOT change it — instead add a
  note under 'Protected-module conflicts' explaining what and why, and leave it unchanged in
  the roadmap body."*

## Step 4 — present the diff, HARD STOP
Diff the new roadmap body against the current `roadmap.md`, module by module: call
out every FREE module added/removed/reordered/reworded, and list the **Protected-module
conflicts** section verbatim if non-empty. This is a gate exactly like `/kickoff`'s D8 — do
**not** write anything yet. Iterate conversationally on the learner's requests; unlike the
agents, the learner *may* explicitly choose to override a protected module here — that's an
informed human decision, not a silent one. Proceed to Step 5 only on explicit approval.

## Step 5 — write
- Overwrite `roadmap.md` with the approved body. Every **protected** module keeps its exact
  prior status — never reset it to `draft`. The drafter's output starts every new/changed
  **free** module at `status: draft` (its own contract, same as `/kickoff`); Step 4's
  approval **is** the D8 gate for those modules, exactly like `/kickoff`'s hard stop — so on
  approval, flip every free/new module's status from `draft` to `approved` when writing.
- Before writing, sanity-check every module's `status:` field is a **bare** value
  (`draft`/`approved`/`planned`/`in-progress`/`done`) with no inline annotation — strip
  anything else back to the bare keyword. Regeneration-mode context (what's protected, what
  conflicted) belongs only in this step's report to the learner and in the "Protected-module
  conflicts" section the drafter produced, never inline on a field other skills string-match
  against (`/plan-module`, `/author`, `/check` all read `roadmap.md` module status
  verbatim).
- Update `modules/README.md`: add rows for brand-new modules; remove rows only for `draft`
  modules that were dropped. Never remove a protected module's row.
- Append a `CLAUDE.md` changelog entry: date, the Step-1 driver, and a one-line summary of
  what the roadmap diff actually changed.

## Step 6 — report
Summarize what changed, what stayed protected/untouched, and next steps — `/plan-module N`
for any module that is newly `draft`→needs approval or was materially reworked; nothing
further needed for untouched modules.
