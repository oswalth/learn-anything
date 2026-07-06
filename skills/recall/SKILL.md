---
name: recall
description: >-
  Run a spaced-repetition review session over due retention cards — free recall, grade 0–5
  with justification, probe the "why" on shallow answers, and update each card's SM-2
  schedule in place. Use when the user says "review", "do my flashcards", "what's due", or
  runs /recall (optionally /recall module N to filter).
argument-hint: "[module N]  (optional filter)"
---

# /recall — spaced-repetition session

Run a recall session over the cards that are **due**, grade them, and reschedule each using
the SM-2 variant in [`references/scheduling.md`](references/scheduling.md).

## Select the cards
- Load cards from `retention/deck/*.md` where `status: active` and `due <= today`.
- If `module N` was given (`$ARGUMENTS`), filter to cards whose `source` is under module NN.
- If nothing is due, say so and give the forecast (when the next cards come due). Don't invent
  a session.
- Cap a session at a sane size (≈ 20 cards); if more are due, do the most-overdue first and
  tell the learner how many remain.

## Run it
For each card, **one at a time**:
1. Show only `Q`. Free recall — no hints, no options.
2. Compare the learner's answer to `A` (the required points) and **probe the why** if the
   answer was shallow ("right, but *why* does it work that way?", "when would that fail?").
3. Assign a **grade 0–5** (scale in the scheduling reference) **with a one-line
   justification** — the learner should know why they got a 3 not a 5.
4. **Reschedule in place:** update `ease`, `interval_days`, `due`, and append to `history`
   per the SM-2 variant. Write the frontmatter back; leave the Q/A body untouched.
5. Flag leeches: if a card has lapsed repeatedly, consider `status: suspended` and note it.

## Log + forecast
- Append a one-line entry to `retention/log.md`: date, cards reviewed, count by grade band
  (e.g. "12 cards: 7 pass, 3 shaky, 2 lapse"), any suspensions.
- End with a **one-line forecast**: "next: 6 cards due Thursday" (compute from the updated
  `due` dates across the deck).
- If a concept keeps lapsing, suggest `/mentor` on it or a `/checkpoint`.

Do not create new cards here (that's `/quiz` and `/checkpoint`), and do not touch section
content.
