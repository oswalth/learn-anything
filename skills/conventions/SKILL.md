---
name: conventions
description: >-
  The normative conventions for all learn-anything learning content: the Content depth
  standard, the challenge-based Practice philosophy, the evolution-lens rule, and the
  changelog discipline. This is a KNOWLEDGE skill, not a command — read it (or the
  specific section named) whenever you generate, critique, validate, mentor, or reason
  about the QUALITY of theory/practice/validation/quiz content, roadmaps, or retention
  cards. Every drafter, critic, and validator in this plugin defers to this file as the
  single source of truth so the standard is never duplicated or drifts. Load it before
  writing or judging any learning artifact.
---

# learn-anything conventions

These are the quality rules for every learning artifact this plugin produces. They are
**normative**: a section that violates them is a defect, not a stylistic choice. They were
ported from a proven hand-built system ("ChoreQuest / FastAPI Mastery", "CQ") whose
conventions are treated as validated-by-usage defaults.

This file is the *single home* of these rules. Skills and agents reference it rather than
restating it. When these rules change, they change here (via `/evolve`) and every consumer
picks the change up at once. If you are a critic or drafter, cite the specific clause you
are enforcing or applying (e.g. "depth standard: Origins & evolution block missing").

The learner is a **strong senior engineer who is a genuine beginner in this section's
subject.** Never skip a step as "obvious"; never hand-wave the mechanism. Also never
condescend or re-explain what a senior already knows about *adjacent* fields — lean on the
adjacent-expertise contrasts the topic `CLAUDE.md` records.

---

## 1. Content depth standard

**Goal: the learner must be able to *advise* on each topic — understand the tradeoffs and
justify decisions — not merely use it.** Surface-level, descriptive, or tutorial-only
content is a **failure**, not a lighter option.

For **every concept** a section introduces, `theory.md` MUST contain all six of the
following, plus a seventh (**Visualize it**) whenever the concept has structure, flow, or
hierarchy. Missing any of the six — or an applicable diagram — is a depth defect:

1. **Motivate it.** What problem does it solve? What goes wrong without it? Why does it
   exist at all? Start from the pain, not the API.
2. **Origins & evolution.** *(Structural, mandatory — do not skip.)* What world made this
   necessary? What preceded it, and what did it displace or fix? Trace the lineage far
   enough that the current design looks inevitable rather than arbitrary. The canonical
   example is Python async: `generators → yield from → @asyncio.coroutine → async def` —
   each step existed to fix a concrete limitation of the last. Every concept has such a
   lineage; find it and tell it. For non-code topics this is the intellectual history
   (who needed it, what problem in the field forced its invention, what it replaced).
3. **Explain the mechanism.** How it actually works under the hood, far enough that its
   behavior becomes *predictable* rather than magic. If the learner still has to guess
   what happens in an edge case after reading, the mechanism section is incomplete.
4. **State the tradeoffs.** Costs, failure modes, and the alternatives it competes with —
   named, and compared honestly. What does choosing this rule out?
5. **Give decision guidance.** When to use it, when not to, and how to choose between it
   and the alternatives. Concrete heuristics tied to observable conditions — **never
   "it depends"** without saying *on what* and *which way each factor pushes*.
6. **Show it concretely.** Small, correct, runnable examples first; then how it applies to
   the topic's capstone/workspace. Examples must actually run and produce the stated result.
7. **Visualize it — when the concept has structure, flow, or hierarchy.** If the concept's
   mental model is fundamentally relational or sequential — an evaluation flow (e.g. IAM
   policy evaluation), a hierarchy (account/org structure), a request path, a network
   topology, a state machine — include an ASCII diagram or a fenced ` ```mermaid ` block
   that makes the shape visible at a glance. Place it alongside the Mechanism block (§1.3)
   as a supplement, never as a replacement for the prose explanation. Skip this element
   only when the concept is genuinely flat or scalar (a single value, a config flag, an API
   call with no branching) — in that case there is nothing to diagram and its absence is not
   an objection. When it's a close call, include the diagram: a visible mental model is cheap
   insurance against content that reads as flat prose even when the other six elements are
   present.

**No length limit.** Depth and completeness beat brevity. Sections exist precisely so each
can be exhaustive within one focused pass. A section is sized to a single 1–2 h study
session (see §4), not trimmed to hit a word count.

---

## 2. Practice philosophy

`practice.md` is **challenge-based, not copy-paste ready.** The learner must *derive* the
solution from `theory.md`, not transcribe it from `practice.md`. Every `practice.md` has
this structure:

1. **Setup steps** — workspace creation, dependency installation, config. Plain
   instructions, not challenges. Get the environment ready with zero puzzle.
2. **Worked examples — at most TWO per section.** Full, runnable, correct code (or, for a
   non-code topic, a fully worked problem). Labeled explicitly **"Worked example"**. The
   learner runs/reads it to see the mechanism in action before writing their own. Two is a
   hard ceiling — more than two turns practice into a tutorial.
3. **Every other exercise is a challenge.** Provide, and only provide:
   - A **starter skeleton**: correct imports, type-annotated signatures, and the calling
     code (`main()` / test harness / problem framing) so the learner knows exactly what
     the finished thing must produce — with `# YOUR CODE HERE` in every body. **Never
     write the body.**
   - The **expected output or observable behavior** (timing, print order, return values,
     a plot shape, a passing check) so the learner can self-verify *without* a reference
     solution.
   - An expandable hints block:
     `<details><summary>Hints (read theory.md first)</summary>` … `</details>` — 2–4
     bullets that point into the relevant `theory.md` section and name the key API/idea.
     Hints ladder from "which technique class" toward "first step" — never toward the
     answer.
4. **Theory–practice linking.** Every worked example and challenge heading carries a
   `**Builds on:** theory.md § "<exact heading text>"` line directly beneath it, naming the
   theory.md heading(s) it draws on. For more than one heading, list them in a single tag
   line separated by commas, e.g. `**Builds on:** theory.md § "A", theory.md § "B"` — never
   split across multiple tag lines. This is what lets `/study` interleave theory and practice
   precisely instead of guessing; a tag naming a heading absent from `theory.md`, or a
   missing tag, is a `critic-pedagogy` FAIL **at generation/revision time**. Sections
   generated before this convention existed are unaffected and are never re-judged against
   it — `/study` detects the absence of tags and falls back to the section's `README.md`
   Order sequence for them.
5. **Drills / predict-then-run** exercises are welcome — they are already challenge-format.

**Capstone / workspace increments follow the same rule:** provide acceptance criteria and
a skeleton, never a filled-in feature.

**Non-code adaptation.** Worked example = a fully worked problem. Challenge = problem
statement **+ answer-shape + verification method** (e.g. "differentiate your solution and
substitute it back to confirm"; a numeric check via sympy or a plot). Hints ladder from
"which class of technique applies" to "what the first step is".

**Respect the spend cap.** Generators and mentor MUST design every exercise, experiment, and
capstone increment to fit within the topic's `ENVIRONMENT.md` spend cap. This is a
model-side design constraint, not a system guarantee — the plugin cannot stop real spend, so
for cloud topics the *first* practice exercise is always setting up billing alarms and a
teardown script, which are the actual backstop for the cap.

---

## 3. Evolution-lens rule

The **Origins & evolution** block (§1.2) is not optional color — it is a load-bearing part
of advise-level understanding. A senior who knows *why* a thing came to be, what it
replaced, and what pressure shaped it can reason about where it will break and what will
replace it next. Enforce it as strictly as correctness:

- Every concept gets its lineage. "It was always like this" is almost never true — dig.
- Prefer the *causal* story ("X existed; it couldn't do Y; so Z was introduced") over a
  bare timeline of dates.
- When a design is a compromise or a historical accident, say so plainly — that is exactly
  the kind of judgment that separates advising from using.
- In **adopt** mode (roadmap maps onto an external curriculum), the evolution lens is the
  plugin's *distinctive* addition on top of the source material: pre-briefings, post-unit
  Socratic debriefs, and currency patches all carry it.

---

## 4. Session sizing

One **section** ≈ one **1–2 hour** study session: roughly 30–40 min of theory reading and
60–80 min of practice. When planning modules into sections, split so each section is
exhaustive *and* fits this budget. If a concept cannot be done justice in one session,
it is two sections, not one crammed one. `critic-pedagogy` enforces this against generated
sections; `/plan-module` enforces it when planning.

---

## 5. Changelog discipline ("keep this file honest")

Every scaffolded topic `CLAUDE.md` carries this block verbatim, and it is a real rule, not
decoration:

> **⚠️ Keep this file honest.** `CLAUDE.md` (with `roadmap.md`) is the single source of
> truth for this topic. **Any** change — reordering/adding/removing/merging modules,
> swapping a tool/version choice, changing scope, process mix, or capstone — **must be
> recorded in the Changelog** (date · what · why) **before** the work diverges from the
> document. If generating or validating reveals the plan should change, stop and update
> the document first.

The plugin holds itself to the identical standard via `CHANGELOG.md` + `/evolve`.
`/retro` (in a topic) and `/evolve` (in the plugin) are the **only** sanctioned mutation
paths, and both end in a changelog entry — so the system's own history stays a first-class
learning artifact rather than eroding through silent edits.

**Post-approval roadmap changes** (D8): once a roadmap is approved, any change to it
requires a topic `CLAUDE.md` changelog entry recording date, what, and why.

---

## 6. Cross-cutting conventions

- **Context discipline (hard rule).** Every command reads ONLY the files its contract
  lists — never "the whole repo". This keeps token cost bounded and behavior predictable.
- **Freshness.** Version-sensitive claims are checked against the topic `BASELINE.md` and
  current web sources; volatile facts are phrased "as of <date>". `critic-freshness`
  owns this at generation time; `/update-baseline` re-verifies over time.
- **Adjacent-expertise contrasts.** Use the contrasts recorded in the topic `CLAUDE.md`
  (CQ's Django↔FastAPI contrasts generalized) to teach by leverage — explain the new
  thing against what the learner already knows deeply, including for non-code topics.
- **Disposable vs persistent artifacts.** `README.md`, `theory.md`, `practice.md`,
  `validation.md`, and `quiz.md` (the section drafter's five artifacts) are regenerable and
  disposable. The topic `CLAUDE.md`, `roadmap.md`,
  each module `README.md` (section plan + learner notes), and everything under
  `retention/` and `workspace/` PERSIST. Never overwrite a persistent artifact to fix a
  disposable one.
