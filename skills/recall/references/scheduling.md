# Spaced-repetition scheduling — the SM-2 variant

The exact algorithm `/recall` uses to reschedule a card after grading it. It is a variant of
SM-2 adapted to the card frontmatter in §6.5, which stores `interval_days`, `ease`, `due`,
and `history` — **but no explicit repetition counter** (the counter is inferred from
`interval_days`, so nothing extra needs storing).

## Card scheduling fields
```yaml
interval_days: 4        # current inter-review interval, in days
ease: 2.5               # ease factor (E-Factor); starts at 2.5, floor 1.3
due: 2026-07-12         # next review date
history: [{date: 2026-07-05, grade: 3}]   # append one entry per review
```
New cards (created by `/quiz` on a miss) start `interval_days: 0`, `ease: 2.5`, and
`due: <tomorrow>`.

## Grades (0–5)
Grade recall quality on this scale (the grader assigns it, with a one-line justification):

| Grade | Meaning |
|---|---|
| 5 | Perfect, immediate, complete — including the *why*. |
| 4 | Correct with minor hesitation or a small gap. |
| 3 | Correct but effortful / shallow — got there, but the reasoning was thin. |
| 2 | Wrong, but the correct answer felt familiar once seen. |
| 1 | Wrong; barely recognized it. |
| 0 | Blank — no recall. |

**Grade < 3 is a lapse** (a failed review). Grade ≥ 3 is a pass.

## The update, given grade `q`

Apply in this order.

**1. Update ease** (do this for every review, pass or lapse):
```
ease' = ease + (0.1 - (5 - q) * (0.08 + (5 - q) * 0.02))
if ease' < 1.3: ease' = 1.3          # ease floor
```
Reference values of the delta `(0.1 - (5-q)*(0.08 + (5-q)*0.02))`:

| q | Δease |
|---|---|
| 5 | +0.10 |
| 4 | +0.00 |
| 3 | −0.14 |
| 2 | −0.32 |
| 1 | −0.54 |
| 0 | −0.80 |

**2. Compute the new interval:**
```
if q < 3:                       # LAPSE
    interval_days' = 1          # reset to 1 day; card must be re-earned
else:                           # PASS
    if interval_days == 0:      # first successful review of this card
        interval_days' = 1
    elif interval_days == 1:    # second successful step (or first after a lapse)
        interval_days' = 6
    else:
        interval_days' = round(interval_days * ease')   # use the updated ease
```

**3. Set the next due date:**
```
due' = today + interval_days'  (calendar days)
```

**4. Append to history:**
```
history += {date: today, grade: q}
```

Write all four fields back into the card's frontmatter in place. Do not rewrite the Q/A body.

## Worked examples

- **New card, graded 4.** `interval_days 0→1`, `ease 2.5→2.5`, due = tomorrow.
- **`interval_days: 1`, ease 2.5, graded 5.** ease→2.6, interval→6, due = today+6.
- **`interval_days: 6`, ease 2.6, graded 3.** ease→2.46, interval = round(6 × 2.46) = 15,
  due = today+15.
- **`interval_days: 15`, ease 2.46, graded 1 (lapse).** ease→1.92, interval→1 (reset),
  due = tomorrow. Next pass will jump 1→6 again, rebuilding.
- **Ease floor.** A card graded 0 repeatedly drops toward 1.3 and stops there, so mature
  intervals grow slowly rather than collapsing to nothing.

## Suspension
If a card lapses many times (e.g. ≥ 4 lapses in its history without reaching a long
interval), `/recall` may mark it `status: suspended` and suggest a `/mentor` session or a
remedial section — a leech, in Anki terms. Suspended cards are skipped by `/recall` until
reactivated.
