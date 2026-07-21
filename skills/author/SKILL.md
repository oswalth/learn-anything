---
name: author
description: >-
  Generate — or regenerate — one section's content — README.md, theory.md, practice.md,
  validation.md, quiz.md — in one section-drafter pass.
  Use when the user says "generate section N.K", "author section N.K", "regenerate section
  N.K", "redo section N.K", "generate module N", "regenerate module N", or runs /author N.K
  (one section) or /author N (loop over a module, with an upfront cost warning). Re-running
  on a section already `generated`/`studied` is how content gets regenerated — there is no
  separate regenerate command — but it requires explicit confirmation before overwriting.
  Refuses on modules that aren't approved/planned. Side-effect command (spends tokens,
  writes files) — user-invoked only.
disable-model-invocation: false
argument-hint: "N.K | N"
---

# /author N.K — produce a section in one pass

Generate the five artifacts with one `section-drafter` invocation. The learner can request a
targeted `/patch` later or regenerate the section by running `/author` again.

## Argument handling
- `N.K` → generate that one section.
- `N` → generate **every planned section** in module N in order. First print an **upfront
  cost warning** with the number of sections and get a confirm before looping.
- Missing → ask which section.

## Regenerating (already-`generated`/`studied` content)
This is the *only* path to regenerate a section — there is no separate `/regenerate`
command. Before drafting, check the target section(s)' current **section status** in the
module `README.md`:
- **`planned`** → proceed normally, no extra confirmation needed.
- **`generated` or `studied`** → this run will **overwrite** existing content and, for a
  `studied` section, **un-certify mastery back to `generated`**. STOP and confirm explicitly
  before proceeding: name the affected section(s) and their current status, and say plainly
  that mastery status resets. Only continue on an explicit yes.
- For `N` (module scope): if *some or all* sections in the module are already `generated`+,
  fold them into the same upfront cost warning as the fresh-generation loop — list every
  section that will be touched with its current status, get one confirmation covering all of
  them, then loop through **every** section in the module (not just the still-`planned`
  ones), applying the per-section confirm rule above implicitly (already covered by the
  batch confirmation).
- If the learner declines, leave the section(s) untouched and stop.

## Preconditions (check, then stop if unmet)
- **Plugin compatibility (soft).** Read the topic `CLAUDE.md`'s recorded **Minimum version**
  and compare it to the installed version in the active tool's plugin manifest. If it is
  **older** than the topic's recorded minimum, WARN the learner, suggest updating through the
  active tool's plugin manager, and ask whether to proceed. This is a warning, not a block.
- Read `roadmap.md`. The target module's **module status** must be `approved`, `planned`, or
  `in-progress` (i.e. it has been through `/plan-module`, or is at least approved, or already
  has some sections authored). **If it is `draft`, REFUSE** — this is the D8 gate; tell the
  learner to approve the roadmap (and ideally `/plan-module N`) first.
- The section id `N.K` must exist in `modules/NN-slug/README.md`'s section plan. If module N
  is `approved` but not yet `planned`, tell the learner to run `/plan-module N` first.

## Read (context discipline — only these)
- The full normative [conventions](../conventions/SKILL.md). Pass the full text inline to
  `section-drafter`; subagents run from the topic-repo cwd and cannot open plugin-internal
  files by relative path.
- Topic `CLAUDE.md`, `BASELINE.md`, `ENVIRONMENT.md`.
- The module `README.md` (this section's goal within the plan).
- **Prerequisite sections' `theory.md`** — only the prerequisites the roadmap lists for this
  module. Do not read the whole repo.
- Current `workspace/` state relevant to the capstone increment.

## Run once
Invoke `section-drafter` once with the inputs above, the section directory path, and the full
conventions text inlined. It writes `README.md`, `theory.md`, `practice.md`, `validation.md`,
and `quiz.md` directly into the section directory. Do not ask it to self-review or revise.

## Write (verify + bookkeeping)
The five files are **already on disk** — the drafter wrote them to
`modules/NN-slug/sections/0K-slug/` (`README.md`, `theory.md`, `practice.md`, `validation.md`,
`quiz.md`). **Verify all five exist**, then do bookkeeping (below). These are
**disposable/regenerable** — overwriting them is expected. Never touch persistent artifacts
(module README notes, retention/, workspace/).

## After generating
- Set the **section status** to `generated` in the module `README.md` (section vocabulary:
  `planned → generated → studied`).
- Set the **module status** in `roadmap.md` to `in-progress` (if it was `planned`; leave it
  `in-progress` if already there) and update the `modules/README.md` board. `/author` never
  sets a module to `done` — that is a mastery claim owned by `/check N`. Do **not** use
  `generated` as a module status.
- Tell the learner: start at `README.md`, then read `theory.md`, work `practice.md` in
  `workspace/`, then `/check N.K` and `/quiz N.K`.
- If this was a **regeneration** of a previously `studied` section, say explicitly: mastery
  reset to `generated` (re-`/check N.K` to re-certify); retention cards from the old
  `quiz.md` are not retroactively edited — a fresh `/quiz N.K` layers new cards on top, so
  suggest reviewing `retention/deck/` for that section if the content changed materially.
  If `workspace/` already holds practice work for this section, note it may no longer match
  the regenerated `practice.md` — suggest diffing before continuing.
