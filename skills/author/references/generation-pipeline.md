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
        └──── revise loop: until ALL pass, max 3 iterations;
              then unresolved objections are SURFACED to the learner
              (never silently dropped or auto-resolved)
```

## Step 0 — assemble inputs (done by the /author skill)
Gather, and only gather: topic `CLAUDE.md`, `BASELINE.md`, `ENVIRONMENT.md`, the module
`README.md` section plan, the prerequisite sections' `theory.md` (only those the roadmap
lists), and relevant `workspace/` state. Also **read the conventions yourself** from the
absolute plugin path `${CLAUDE_PLUGIN_ROOT}/skills/conventions/SKILL.md` (fallback
`${CLAUDE_SKILL_DIR}/../conventions/SKILL.md`). Pass all of these — **and the conventions
text inlined** — into the drafter and critic prompts so they don't re-fetch and never need to
open plugin-internal files by relative path (a subagent's cwd is the topic repo, not the
installed plugin).

Pass every agent the **section directory path** (`modules/NN-slug/sections/0K-slug/`) and the
inlined conventions text — **not** the artifact bodies. Content flows through files on disk, so
nothing large is copied through agent boundaries (the depth standard has "no length limit"; the
prior text-passing hand-off was truncation- and token-fragile).

## Step 1 — draft
Invoke **`section-drafter`** (Agent tool) with the inputs and the section dir path. It **WRITES**
the five files (`README.md`, `theory.md`, `practice.md`, `validation.md`, `quiz.md`) directly into
the section directory and returns a confirmation (and, on revision passes, a change summary). It
does not paste the bodies back.

## Step 2 — critique in parallel
Invoke all four critics **in a single message so they run concurrently**; each **READS** the four
content files (`theory.md`, `practice.md`, `validation.md`, `quiz.md` — not `README.md`, which is
navigational and not judged) from the section dir it is given:
- `critic-accuracy` — correctness; runs code where the environment allows; checks exercise
  outputs are achievable.
- `critic-freshness` — every version-sensitive claim vs `BASELINE.md` + live web; "as of
  <date>" phrasing for volatile facts.
- `critic-depth` — the six depth-standard elements per concept (Origins & evolution
  hardest), plus Visualize it for concepts with structure/flow/hierarchy.
- `critic-pedagogy` — challenge-based practice, ≤2 worked examples, verifiable behavior,
  hint blocks, the theory↔practice `Builds on:` tag, 1–2 h sizing.

Each returns `PASS` or a numbered objection list.

## Step 3 — revise loop (max 3 iterations)
- If **all four** PASS → go to Step 5.
- Otherwise, collect every objection and send them back to `section-drafter` for a targeted
  revision (fix the objections in place; don't rewrite what passed). The drafter returns a
  **change summary** — which files it edited and what kind of change — so you can apply the
  re-run rule mechanically.

**Deterministic re-run rule** (apply after each revision — this is a rule, not a judgement
call). Re-run this set of critics, then repeat up to **3 total iterations**:
- **ALWAYS** re-run every critic that **FAILed** last round.
- **ADDITIONALLY** re-run, keyed to what the revision changed (per the drafter's change summary):
  - `critic-accuracy` — if the revision changed **any code/command/example or any claimed output**;
  - `critic-freshness` — if the revision changed **any version/tool/default/API claim**;
  - `critic-depth` — if the revision changed **theory.md**;
  - `critic-pedagogy` — if the revision changed **practice.md**.
- A critic that **PASSed** and whose domain the revision **provably did not touch** may be
  skipped. When **uncertain** whether a domain was touched, **re-run it** (bias toward re-running).

## Step 4 — surface unresolved objections (never hide them)
If after 3 iterations any objection stands:
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
  `planned → generated → studied`). Set the **module status** in `roadmap.md` to
  `in-progress` (from `planned`; stays `in-progress` thereafter) and update the
  `modules/README.md` board. Never set a module to `generated` or `done` here — `done` is a
  mastery verdict owned by `/check N`.
- Report to the learner: what was generated, whether all critics passed cleanly or objections
  were surfaced, and the next commands (`/check N.K`, `/quiz N.K`).

## Notes
- **Cost.** One section = 1 drafter + 4 critics + up to ~3 revise rounds. `/author N`
  multiplies this by the number of sections — warn before looping.
- **Determinism of scope.** The critics judge; they do not expand scope. If a critic thinks
  the section should cover more, that is a `/plan-module` or roadmap concern, surfaced to the
  learner — not something the drafter silently balloons the section to include.
