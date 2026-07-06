---
name: quiz
description: >-
  Run a section's quiz.md interactively right after practice — free recall first, then "why?"
  and "when would this break?" follow-ups. Misses and shaky answers become new retention
  cards (due tomorrow). Use when the user says "quiz me on N.K", "test me on this section",
  or runs /quiz N.K.
argument-hint: "N.K (section)"
---

# /quiz N.K — post-practice recall quiz

Run the section's quiz interactively and turn misses into spaced-repetition cards. This is
the immediate-retention step, run right after the learner finishes practice.

## Read (context discipline — only these)
- The section's `quiz.md` (the questions + reference answers).
- The section's `theory.md` only if you need to judge an answer or give a pointer.
- Existing `retention/deck/` filenames only enough to pick non-colliding card ids.

## How to run it
1. Ask the questions **one at a time**. Free recall first — do **not** show options or hints.
   Wait for the learner's answer.
2. Grade the answer against the reference (what a good answer must contain), then **probe**:
   "why?", "when would that break?", "what's the tradeoff vs <alternative>?". A right-sounding
   answer with no *why* is a shaky answer.
3. Tell them how they did per question: correct / shaky / missed, with the key point they
   missed if any. Point into `theory.md`, don't lecture.

## Turn misses into cards
For every **missed or shaky** answer, create a card in `retention/deck/` using the §6.5
format:
```yaml
---
id: NN-K-qJ                     # module-section-question, unique in the deck
source: modules/NN-slug/sections/0K-slug
type: recall | why | contrast | code-prediction
status: active
interval_days: 0
ease: 2.5
due: {{tomorrow, YYYY-MM-DD}}
history: []
---
Q: <the prompt — prefer the why/when/tradeoff framing over a bare definition>
A: <reference answer as bullet points of what a good answer must contain>
```
- Favor `why` / `contrast` / `code-prediction` card types over pure `recall`.
- One card per distinct miss. Don't duplicate a card that already exists for the same point —
  if it exists, leave its schedule alone (or reset to due tomorrow if it was clearly lost).
- Fully-correct answers with solid *why* do **not** create cards.

## Finish
Summarize: score, which points became cards, and remind the learner that `/recall` will
resurface them tomorrow. If several cards were created for one concept, suggest a `/mentor`
pass on it.
