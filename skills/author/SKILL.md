---
name: author
description: >-
  Generate one section's content — README.md, theory.md, practice.md, validation.md, quiz.md —
  via the drafter + 4-critic quorum with a revise-until-pass loop. Use when the user says "generate
  section N.K", "author section N.K", "generate module N", or runs /author N.K (one section) or
  /author N (loop over a module, with an upfront cost warning). Refuses on modules that aren't
  approved/planned. Side-effect command (spends tokens, writes files) — user-invoked only.
disable-model-invocation: true
argument-hint: "N.K (one section) | N (whole module)"
---

# /author N.K — produce a section through the quorum

Generate the five artifacts for one section using the pipeline in
[`references/generation-pipeline.md`](references/generation-pipeline.md). Read that file —
it is the authoritative procedure, including the revise loop and the surface-to-learner rule.

## Argument handling
- `N.K` → generate that one section.
- `N` → generate **every planned section** in module N in order. First print an **upfront
  cost warning** (how many sections, that each runs a drafter + 4 critics + up to 3 revise
  rounds) and get a confirm before looping.
- Missing → ask which section.

## Preconditions (check, then stop if unmet)
- **Plugin compatibility (soft).** Read the topic `CLAUDE.md`'s recorded **Minimum version**
  and compare it to the installed plugin version in
  `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`. If the installed plugin is **older** than
  the topic's recorded minimum, WARN the learner (this topic expects newer plugin machinery;
  suggest updating via `/plugin`) and ask whether to proceed. This is a warning, not a block.
- Read `roadmap.md`. The target module's **module status** must be `approved`, `planned`, or
  `in-progress` (i.e. it has been through `/plan-module`, or is at least approved, or already
  has some sections authored). **If it is `draft`, REFUSE** — this is the D8 gate; tell the
  learner to approve the roadmap (and ideally `/plan-module N`) first.
- The section id `N.K` must exist in `modules/NN-slug/README.md`'s section plan. If module N
  is `approved` but not yet `planned`, tell the learner to run `/plan-module N` first.

## Read (context discipline — only these)
- The **conventions** — the full normative standard. Load `conventions/SKILL.md` **yourself**
  via the absolute plugin path `${CLAUDE_PLUGIN_ROOT}/skills/conventions/SKILL.md` (fallback
  `${CLAUDE_SKILL_DIR}/../conventions/SKILL.md`), then pass its text **inline** into the
  drafter and critic prompts — subagents run from the topic-repo cwd and cannot open
  plugin-internal files by relative path.
- Topic `CLAUDE.md`, `BASELINE.md`, `ENVIRONMENT.md`.
- The module `README.md` (this section's goal within the plan).
- **Prerequisite sections' `theory.md`** — only the prerequisites the roadmap lists for this
  module. Do not read the whole repo.
- Current `workspace/` state relevant to the capstone increment.

## Run the pipeline (summary — details in the reference)
Pass every agent the **section directory path** (`modules/NN-slug/sections/0K-slug/`) plus the
inlined conventions text — not the artifact bodies. Content flows through files on disk, not
through agent prompts.
1. **Draft** — invoke `section-drafter` with the inputs above (section dir path + conventions
   text **inlined**). It **WRITES** the five files (`README/theory/practice/validation/quiz.md`)
   directly to the section directory and returns a confirmation (plus a change summary on
   revision passes).
2. **Critique in parallel** — invoke `critic-accuracy`, `critic-freshness`, `critic-depth`,
   `critic-pedagogy` (one message, parallel). Each **READS** the four content files (not the
   README — it's navigational, not judged) from the section dir and returns PASS or numbered
   objections.
3. **Revise loop** — if any critic FAILs, send all objections back to `section-drafter` (it
   edits the files in place and returns a change summary); re-critique per the **deterministic
   re-run rule** in the reference. **Max 3 iterations.**
4. **Surface, never hide** — if objections remain unresolved after 3 iterations, the files
   stay on disk BUT you present the remaining objections to the learner explicitly: **prepend**
   a `> ⚠️ Unresolved critic objections` block to the top of each affected file (the
   orchestrator writes it) and list them in your reply. Never silently drop or auto-resolve them.

## Write (verify + bookkeeping)
The five files are **already on disk** — the drafter wrote them to
`modules/NN-slug/sections/0K-slug/` (`README.md`, `theory.md`, `practice.md`, `validation.md`,
`quiz.md`). **Verify all five exist**, then do bookkeeping (below). These are
**disposable/regenerable** — overwriting them is expected. Never touch persistent artifacts
(module README notes, retention/, workspace/).

## After generating
- Set the **section status** to `generated` in the module `README.md` (section vocabulary:
  `planned → generated → studied`).
- Set the **module status** in `roadmap.md` to `in-progress` (if it was `planned`; leave it
  `in-progress` if already there) and update the `modules/README.md` board. `/author` never
  sets a module to `done` — that is a mastery claim owned by `/check N`. Do **not** use
  `generated` as a module status.
- Tell the learner: start at `README.md`, then read `theory.md`, work `practice.md` in
  `workspace/`, then `/check N.K` and `/quiz N.K`.
