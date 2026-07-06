---
name: patch
description: >-
  On-demand, bounded fix for ONE flagged claim in an already-generated section — applied
  immediately during study, without waiting for a full /author regeneration. Use when the
  learner flags a specific line as wrong or outdated, or asks for more depth on one concept,
  mid-mentoring or mid-study. Verifies the edit with the matching critic (accuracy for wrong
  claims, freshness for outdated facts, depth for thin coverage) before finishing. Mutates
  section content, so it is user-invoked only: /patch N.K "what's wrong or what needs more
  depth".
disable-model-invocation: true
argument-hint: "N.K \"note describing the correction or depth request\""
---

# /patch N.K — bounded, on-demand section correction

Fix or deepen ONE flagged claim/concept in an already-generated section, right now, without
re-running the full drafter + 4-critic quorum. This is a scalpel: touch only what the learner
flagged, verify with one matching critic, stop.

## When to use vs `/author`
- One wrong or outdated claim, or one concept that's thin on depth → `/patch`.
- Multiple issues, a section that needs re-planning, or the section should re-earn all four
  critics' blessing → re-run `/author N.K` instead (regenerates through the full quorum).

## Argument handling
- Missing `N.K` or note → ask which section and what's wrong or thin.
- The section must already be `generated` (or later) in the module `README.md`. If it's still
  `planned`, there's nothing to patch — tell the learner to run `/author N.K` first.

## Read (context discipline — only these)
- The section directory's content file(s) the flag concerns — start with whichever file the
  learner named or the flagged concept most plausibly lives in (usually `theory.md`); open
  `practice.md`/`validation.md`/`quiz.md` too only if the flag or its downstream effects reach
  them.
- Topic `CLAUDE.md` (adjacent-expertise contrasts, so the fix stays consistent in voice) and,
  for outdated-fact flags, `BASELINE.md`.
- The conventions, inlined from `${CLAUDE_PLUGIN_ROOT}/skills/conventions/SKILL.md` (fallback
  `${CLAUDE_SKILL_DIR}/../conventions/SKILL.md`) — apply only the clause the flag needs (the
  depth standard §1 for a depth request; the "as of `<date>`" freshness phrasing §6 for a
  staleness flag).

Do not read the rest of the module, other sections, or `workspace/`/`retention/` — this stays
bounded to one section's content files.

## Step 1 — classify the flag
From the learner's note, classify as exactly one of:
- **wrong** — a claim is factually incorrect.
- **outdated** — a claim was true once but the world has moved on (version, default, API,
  "current best practice").
- **thin** — a concept is present but missing one of the depth-standard elements
  (conventions §1) — including an applicable Visualize it diagram for a
  structural/flow/hierarchy concept — or the learner explicitly asked for more depth on it.

If the note doesn't clearly fit one of these, ask rather than guessing.

## Step 2 — make the targeted edit
Edit the flagged file(s) directly with the Edit tool:
- **wrong** — correct the claim in place.
- **outdated** — correct it and phrase the corrected fact "as of `<date>`" per conventions §6.
- **thin** — add the missing depth-standard element(s) for that one concept (conventions §1);
  extend it, don't re-derive the whole concept from scratch.

Never touch unrelated claims, restructure headings, or expand past the flag. If fixing it
reveals the section needs broader rework, stop, say why, and point to `/author N.K` instead of
improvising a bigger edit.

## Step 3 — verify with the matching critic
Invoke (Agent tool) exactly the critic matching the classification, giving it the section
directory path and an explicit note that this is a **`/patch` verification pass**, scoping it
to the edited passage (quote it):
- **wrong** → `critic-accuracy`
- **outdated** → `critic-freshness`
- **thin** → `critic-depth`

Tell the critic it may still surface other things it notices in the file, but that only
objections about the edited passage itself block the patch — everything else is FYI, surfaced
to the learner, never auto-applied (that would break bounded scope).

- If the critic objects to the edited passage, revise once more and re-verify — **max 2
  rounds** (this is a scalpel, not the full revise loop). If still failing after 2 rounds,
  leave the file as it was before Step 2, tell the learner exactly what's unresolved, and
  suggest `/author N.K`.
- Otherwise, the patch stands.

## Step 4 — report, no status change
Tell the learner what changed (quote before/after), which critic verified it, and any
unprompted extra objections the critic surfaced (for awareness, not acted on).

**Never** change section or module status (`studied`/`done` are untouched — a patch doesn't
re-certify mastery; `/check` alone owns those transitions) and never write to `workspace/` or
`retention/`. If the correction touches material already quizzed, mention that retention cards
aren't retroactively edited — a fresh `/quiz N.K` pass will pick up the corrected content.

`theory.md`/`practice.md`/`validation.md`/`quiz.md` are disposable, regenerable artifacts
(conventions §6) — patching them needs no topic `CLAUDE.md` changelog entry, unlike a
roadmap/scope change.
