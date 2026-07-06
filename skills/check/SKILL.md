---
name: check
description: >-
  Review the learner's own work in workspace/ against a section's (or whole module's)
  validation criteria — verdict plus prioritized feedback, never rewriting their code
  unasked. Use when the user says "validate my work", "check my work", "review section N.K",
  "check module N", or runs /check N.K or /check N. For non-code topics, checks worked
  solutions step-by-step against validation.md.
argument-hint: "N.K (section) | N (whole module)"
---

# /check N.K — review the learner's work against the criteria

Assess what the learner built/solved against the section's `validation.md`. You are a
reviewer, not an author: **never rewrite the learner's code or fill in their solution
unless they explicitly ask.** Point precisely; let them fix it.

## Argument handling
- `N.K` → validate that section's increment.
- `N` → validate the whole module across its sections (run after every section is authored and
  worked). A whole-module PASS is what can advance the module to `done` (see below).
- Missing → ask which section/module.

## Read (context discipline — only these)
- The section's `validation.md` (the criteria) — and `theory.md`/`practice.md` only if you
  need to interpret a criterion.
- The learner's work: the relevant part of `workspace/` (the increment this section asked
  for). For a whole-module validation, the module's cumulative increment.
- Topic `CLAUDE.md` only for the conventions/contrasts that frame expectations.

## How to review
1. Go through `validation.md`'s checklist item by item. For each: **met / partially met /
   not met**, with the specific evidence (file:line, a test result, an output, a step in a
   proof).
2. **Verify, don't assume.** Where the topic allows, actually run it — run the tests, execute
   the code, check the output, run the numeric check (sympy/plot). Stay within the
   `ENVIRONMENT.md` spend cap; never spin up anything that would exceed it (nothing enforces
   the cap technically, so this is on you).
3. For **non-code topics**: check worked solutions **step by step** against `validation.md`
   — is each step justified, is the method sound, does the verification method confirm it
   (differentiate-and-substitute, numeric check, etc.)?

## Output
- **Verdict:** one of *passes*, *passes with issues*, *not yet* — one line, up top.
- **Prioritized feedback:** ordered by importance. For each: what's wrong or missing, why it
  matters, and a **pointer** to the theory/hint that unblocks it — not the answer.
- **What's good:** briefly reinforce what they got right (especially the hard parts).
- If they explicitly ask you to show a solution, you may — but default to guiding.

## After validating — status ownership (mastery, not generation)
`/check` owns the two **mastery** transitions. Advance status only on a genuine PASS, and only
with the learner's explicit confirmation — never on *passes with issues* or *not yet*.

- **`/check N.K` PASS → SECTION `studied`.** On a passing section check, offer to mark that
  section's status `studied` in the module `README.md` (section vocabulary:
  `planned → generated → studied`). Confirm first; this records that the learner worked and
  passed the section, which nothing else marks. Then suggest `/quiz N.K` (to lock in
  retention) and the next section.
- **`/check N` PASS → MODULE `done`.** On a passing whole-module check (every section passes),
  offer to mark the **module status** `done` in `roadmap.md` and the `modules/README.md` board
  (module vocabulary: `draft → approved → planned → in-progress → done`). Confirm first —
  `done` is a mastery claim, so validation owns it, not authoring.
- If it does **not** pass, change nothing; suggest `/mentor N.K` for a guided derivation of the
  sticking point.
- These status edits (section→`studied`, module→`done`) are the **only** files `/check` may
  write, and only on a confirmed pass. **Never** edit the learner's `workspace/` files or
  rewrite their solution.
