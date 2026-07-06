---
name: study
description: >-
  Paces the learner through a section interactively: interleaves theory.md concepts with the
  practice.md challenges that build on them, waiting for the learner between each step —
  instead of handing over four static files and leaving them solo. Falls back to the
  section's README.md Order sequence for sections authored before theory-practice linking
  existed. Distinct from /mentor (on-demand Socratic help when stuck) and /check (grades the
  work). Use when the user says "study section N.K", "walk me through N.K", "walk me through
  this section", "guide me through this section", "start section N.K", "pace me through
  this", or runs /study N.K.
argument-hint: "N.K (section)"
---

# /study N.K — paced walkthrough of a section

Sequence the learner through one section's already-generated content instead of leaving them
to work four static files solo. This command **paces**; it does not teach Socratically (that's
`/mentor`) and does not grade correctness (that's `/check`).

## Argument handling
- `N.K` → walk through that section.
- `N` (module only) or missing → `/study` works one section at a time; ask which section.

## Read (context discipline — only these)
- The section's `README.md` (prerequisites, the fallback **Order** sequence, the next-section
  pointer).
- The section's `theory.md`, `practice.md`, `validation.md`, `quiz.md`.
Don't read prerequisite sections, the module plan, or the rest of the repo — the learner
already worked those; this command only sequences the section in front of them.

## Build the sequence
1. Parse `theory.md`'s concept headings, in file order.
2. Parse `practice.md`'s worked examples and challenges. Ones authored under the
   theory-practice linking convention (conventions §2) carry a
   `**Builds on:** theory.md § "<heading>", theory.md § "<heading>", …` line directly
   beneath their own heading (all headings it names are on that one line, comma-separated).
   Slot each item into the sequence **once**, right after the *last* heading its tag names
   (in theory.md order) — a challenge that draws on two earlier concepts needs both covered
   first, so it belongs after whichever comes second, never duplicated at both spots.
3. **No worked example or challenge in practice.md carries the tag** → this section predates
   the convention. Say so once, and fall back to the coarser sequence already in
   `README.md`'s **Order** section (theory.md fully, then practice.md in file order, then
   validation.md, then quiz.md) — do not guess a mapping.
4. **Some but not all** carry the tag → use the tag where present; place an untagged item
   right after the last tagged item that precedes it in file order. Note the gap once, don't
   block on it.

## Run the session
1. Open with the plan: how many concepts and challenges, the rough time budget (pull from
   `README.md`/the depth-standard sizing: ~30–40 min theory, ~60–80 min practice). Surface
   `README.md`'s **Before you start** prerequisites and confirm the learner has them.
2. For each theory heading, in order:
   - Point the learner at it by name ("read '<heading>' in theory.md — ~X min") and wait for
     them to confirm they're done. Don't paste the section body into chat — pacing, not
     re-generation.
   - Answer a quick clarifying question if asked. If they're genuinely stuck on the concept
     itself rather than just reading it, say so and hand off to `/mentor N.K` — that is
     mentor's job, not this command's.
   - Present whatever's tagged to that heading, in `practice.md`'s file order:
     - A **worked example** is read/run, not attempted — point to it, let them read or run
       it, and wait for a simple acknowledgment before moving on. Don't treat it like a
       challenge.
     - A **challenge** is attempted — quote its skeleton and expected output/behavior from
       `practice.md` (short, and the learner needs them in front of them). Leave the
       `<details>` hints block collapsed — let the learner ask for it rather than surfacing
       it unprompted, matching the hints ladder's own "don't reveal early" intent. Wait for
       them to finish attempting it before moving on.
   - Do not grade correctness. If asked "is this right?", you may flag something obviously
     broken, but defer the actual verdict to `/check N.K`.
3. After the last pair, summarize `validation.md`'s checklist at a glance and point to
   `/check N.K`.
4. Close by pointing to `/quiz N.K`, then whatever `README.md` names next — the following
   section, or `/check N` if this was the module's last planned section.

## Guardrails
- Never write to `workspace/`, the section files, or any status field — section/module status
  is `/check`'s exclusive job (see its status-ownership contract).
- If the learner flags a claim mid-walkthrough as wrong or outdated, or wants a bounded
  depth-add, don't edit it yourself — point them to `/patch N.K "..."` for an immediate,
  critic-verified fix, then resume pacing.
- Never invent a theory↔challenge mapping beyond what the `Builds on:` tags say or the
  `README.md` Order fallback. A wrong guess is worse than the coarse fallback.
- Let the learner skip ahead or jump around if they want — this is pacing assistance, not a
  lockstep gate.
