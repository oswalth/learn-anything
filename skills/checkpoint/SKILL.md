---
name: checkpoint
description: >-
  Run a cumulative checkpoint at a phase boundary (or on demand): mixes due + sampled old
  cards, "explain it to me like I'm your junior" prompts, and ONE integration challenge
  spanning earlier modules. Writes a gap report; gaps become cards and, if severe, a proposed
  remedial section. Use when the user says "checkpoint", "I finished a phase", "cumulative
  review", or runs /checkpoint.
argument-hint: "[phase or module scope, optional]"
---

# /checkpoint — cumulative, integrative review

A checkpoint is heavier than `/recall`. It tests whether earlier material has *integrated*,
not just whether individual cards are due. Run it at phase boundaries or when the learner
wants a gut-check.

## Scope
- Default: everything completed so far (or the phase just finished, if named).
- Read (context discipline): `roadmap.md` (module status — modules marked `done` are
  whole-module-validated; `in-progress` ones are partway), the completed modules' `README.md`
  section plans (for the integration challenge), and `retention/deck/` for card selection.
  Read section `theory.md` only as needed to judge answers or author the challenge.

## The three parts
1. **Mixed recall.** Draw *all currently due* cards **plus a random sample of older, not-yet-
   due cards** from earlier modules (spaced interleaving). Grade 0–5 and reschedule each per
   the SM-2 variant (`skills/recall/references/scheduling.md`) — a checkpoint counts as a real
   review for scheduling.
2. **Teach-back.** A few "explain it to me like I'm your junior" prompts on the phase's
   central ideas. Listen for hand-waving; push on the *why* and the tradeoffs. These test
   advise-level understanding, not recall.
3. **One integration challenge.** A single problem/exercise that **spans multiple earlier
   modules** and can't be solved from one section alone. Give it challenge-style (skeleton /
   acceptance criteria + verification method, per the practice philosophy) — do not hand over
   a solution. This is where gaps between modules show up.

## Gap report
Write a short gap report (in your reply, and appended to `retention/log.md` with the date):
- What's solid, what's shaky, what's missing — concretely.
- **Turn each gap into a card** in `retention/deck/` (due tomorrow), §6.5 format, favoring
  why/contrast/integration framing.
- **If a gap is severe** (a whole subtopic didn't land), propose a **remedial section**:
  describe it and, with the learner's OK, add it to the relevant module `README.md` section
  plan as `status: planned` and note it in `CLAUDE.md`'s changelog (keep-this-file-honest).
  Do not generate it here — that's `/author`.

## Finish
Summarize readiness for the next phase and the forecast of upcoming due cards. If the
integration challenge exposed structural gaps, suggest the remedial section or a `/retro`.
