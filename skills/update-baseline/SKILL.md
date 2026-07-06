---
name: update-baseline
description: >-
  Re-verify a topic's BASELINE.md against current sources via web search — update pinned
  versions and verified-on dates, and flag generated-but-unstudied sections that the changes
  may invalidate, offering regeneration. Use when the user says "update the baseline", "check
  for new versions", "is anything stale", or runs /update-baseline. Side-effect command (web
  research, file writes) — user-invoked only.
disable-model-invocation: true
---

# /update-baseline — re-verify pinned versions and flag staleness

Bring `BASELINE.md` back in sync with the world, and warn about content that may have gone
stale as a result.

## Read (context discipline)
- `BASELINE.md` (the rows to re-verify).
- `roadmap.md` + module `README.md`s only to identify which sections are **generated but not
  yet studied** — i.e. **section status `generated`** (not yet advanced to `studied` by a
  passing `/check`).
- Do not read the section bodies unless a flagged row needs you to judge impact.

## Procedure
1. For **each row** in `BASELINE.md`, web-search/fetch the authoritative source and find the
   current version/edition. Update the **verified-on date** to today regardless of whether
   the version changed.
2. Where a version **changed**, update the row and note the delta (old → new) and the nature
   of the change if visible (e.g. "minor", "breaking: default X flipped", "new major line").
3. **Flag impact.** For each changed row, determine which **generated-but-unstudied** sections
   (section status `generated`, not yet `studied`) depend on that component (by topic
   knowledge + the module Concept/Skills). List them as "may be invalidated by <component>
   <old→new>".
4. **Offer regeneration.** For each flagged section, offer to re-run `/author N.K`. Don't
   regenerate automatically — the learner decides (already-`studied` sections are usually left
   as-is with a note; `generated`-but-unstudied ones are cheap to refresh).

## Write
- Update `BASELINE.md` in place (versions + dates + notes).
- Add a `CLAUDE.md` changelog entry summarizing what changed and what was flagged (date ·
  what · why) — this is a real change to the topic's source of truth.

## Finish
Report: rows re-verified, versions that moved, sections flagged, and what the learner chose to
regenerate. If a row's live version disagrees with what generated content assumed and the
section is already `studied`, suggest a targeted `/mentor` or a card to capture the delta.
