# Generation pipeline (per section) — the quorum

The authoritative procedure `/author N.K` follows for one section. The shape is a **single
drafter + parallel specialist critics** with a **revise-until-pass loop** (D12).

```
section-drafter ──► draft (README / theory / practice / validation / quiz)
        ▲                    │
        │         ┌──────────┼──────────┬──────────────┐
        │         ▼          ▼          ▼              ▼
        │   critic-      critic-    critic-       critic-
        │   accuracy     freshness  depth         pedagogy
        │         └──────────┴──────────┴──────────────┘
        └──── revise loop: until ALL pass, max N iterations (default 2);
              then unresolved objections are SURFACED to the learner
              (never silently dropped or auto-resolved)
```

## Step 0 — assemble inputs (done by the /author skill)
Gather, and only gather: topic `CLAUDE.md`, `BASELINE.md`, `ENVIRONMENT.md`, the module
`README.md` section plan, the prerequisite sections' `theory.md` (only those the roadmap
lists), and relevant `workspace/` state. Also **read the conventions yourself** from the
absolute plugin path `${CLAUDE_PLUGIN_ROOT}/skills/conventions/SKILL.md` (fallback
`${CLAUDE_SKILL_DIR}/../conventions/SKILL.md`).

**Resolve the run's settings** from `/author`'s argument parsing (see its SKILL.md): the
**critic model** (default `sonnet`; learner may override with `model=opus|sonnet|haiku`) and
the **max revise iterations** `N` (default `2`; learner may override with `iterations=0-3`,
where `0` is the `--fast` alias). Carry both through every step below.

**Conventions are inlined per role, not blasted in full to everyone:**
- **`section-drafter`** gets the **full** conventions text — it must satisfy every clause.
- **`critic-depth`** gets only **§1 (Content depth standard) + §3 (Evolution-lens rule)**.
- **`critic-pedagogy`** gets only **§2 (Practice philosophy) + §4 (Session sizing)**.
- **`critic-freshness`** gets only **§6's Freshness bullet** (one bullet, not all of §6).
- **`critic-accuracy`** gets **none** — correctness isn't conventions-governed; its own
  checklist is self-contained.

This is a pure token-cost optimization, not a judgment-criteria change: each critic already
enforces only its own domain (per its own agent prompt) and never used the other sections it
was previously receiving in full.

Pass every agent the **section directory path** (`modules/NN-slug/sections/0K-slug/`) and its
scoped conventions excerpt (per above) — **not** the artifact bodies. Content flows through
files on disk, so nothing large is copied through agent boundaries (the depth standard has "no
length limit"; the prior text-passing hand-off was truncation- and token-fragile).

## Step 1 — draft
Invoke **`section-drafter`** (Agent tool) with the inputs and the section dir path. It **WRITES**
the five files (`README.md`, `theory.md`, `practice.md`, `validation.md`, `quiz.md`) directly into
the section directory and returns a confirmation (and, on revision passes, a change summary). It
does not paste the bodies back.

## Step 2 — critique in parallel
Invoke all four critics **in a single message so they run concurrently**; each **READS** the four
content files (`theory.md`, `practice.md`, `validation.md`, `quiz.md` — not `README.md`, which is
navigational and not judged) from the section dir it is given, and its scoped conventions excerpt
from Step 0. If the learner set `model=` to something other than the default, pass that as the
Agent tool's `model` parameter on all four invocations (overriding each critic's own frontmatter
default of `sonnet`); otherwise omit it and let the default apply.
- `critic-accuracy` — correctness; runs code where the environment allows; checks exercise
  outputs are achievable.
- `critic-freshness` — every version-sensitive claim vs `BASELINE.md` + live web; "as of
  <date>" phrasing for volatile facts.
- `critic-depth` — the six depth-standard elements per concept (Origins & evolution
  hardest), plus Visualize it for concepts with structure/flow/hierarchy.
- `critic-pedagogy` — challenge-based practice, ≤2 worked examples, verifiable behavior,
  hint blocks, the theory↔practice `Builds on:` tag, 1–2 h sizing.

Each returns `PASS` or a numbered objection list.

## Step 3 — revise loop (max N iterations, default 2; skipped entirely when N=0/`--fast`)
- If **all four** PASS → go to Step 5.
- If `N` is `0` (the learner asked for `--fast`) → do **not** revise regardless of outcome; go
  straight to Step 4 with whatever objections came back from the single Step 2 pass.
- Otherwise, collect every objection and send them back to `section-drafter` for a targeted
  revision (fix the objections in place; don't rewrite what passed). The drafter returns a
  **change summary** — which files it edited and what kind of change — so you can apply the
  re-run rule mechanically.

**Deterministic re-run rule** (apply after each revision — this is a rule, not a judgement
call). Re-run this set of critics, then repeat up to **`N` total iterations**:
- **ALWAYS** re-run every critic that **FAILed** last round.
- **ADDITIONALLY** re-run, keyed to what the revision changed (per the drafter's change summary):
  - `critic-accuracy` — if the revision changed **any code/command/example or any claimed output**;
  - `critic-freshness` — if the revision changed **any version/tool/default/API claim**;
  - `critic-depth` — if the revision changed **theory.md**;
  - `critic-pedagogy` — if the revision changed **practice.md**.
- A critic that **PASSed** and whose domain the revision **provably did not touch** may be
  skipped. When **uncertain** whether a domain was touched, **re-run it** (bias toward re-running).

## Step 4 — surface unresolved objections (never hide them)
If after `N` iterations (or immediately, when `N=0`) any objection stands:
- The **best** draft is already on disk (the drafter wrote it).
- The **orchestrator** prepends a clearly marked block to the top of each affected file (the
  files are on disk, so the orchestrator edits them directly):
  ```
  > ⚠️ Unresolved critic objections (generation reached max iterations):
  > - [critic] <objection>
  > These are surfaced deliberately — resolve with /mentor, a re-generate, or by editing.
  ```
- Also list them in your reply to the learner. **Never** silently drop, downgrade, or
  auto-"resolve" a critic objection.

## Step 5 — verify + bookkeeping
- The five files (`README.md`, `theory.md`, `practice.md`, `validation.md`, `quiz.md`) are
  **already on disk** — the drafter wrote them to the section directory. **Verify all five
  exist**, then:
- Flip the **section status** to `generated` in the module `README.md` (section vocabulary:
  `planned → generated → studied`; if this run regenerated a previously `studied` section,
  this flip un-certifies mastery back to `generated` — the confirm gate for that lives in
  `/author`'s preconditions, not here). Set the **module status** in `roadmap.md` to
  `in-progress` (from `planned`; stays `in-progress` thereafter) and update the
  `modules/README.md` board. Never set a module to `generated` or `done` here — `done` is a
  mastery verdict owned by `/check N`.
- Report to the learner: what was generated, whether all critics passed cleanly or objections
  were surfaced, the next commands (`/check N.K`, `/quiz N.K`), and — only if the run used a
  non-default `model=` or `iterations=` — a one-line note of what was overridden and why that
  changes the cost/rigor tradeoff for this section.

## Notes
- **Cost.** One section = 1 drafter (opus) + 4 critics (sonnet by default) + up to `N` revise
  rounds (default 2; 0 with `--fast` — critics run once, nothing is auto-revised; up to 3 with
  `iterations=3` for maximum rigor on a specific section). `/author N` multiplies this by the
  number of sections — warn before looping, and state the resolved model/iterations settings
  in that warning since they materially change the estimate.
- **Determinism of scope.** The critics judge; they do not expand scope. If a critic thinks
  the section should cover more, that is a `/plan-module` or roadmap concern, surfaced to the
  learner — not something the drafter silently balloons the section to include.
