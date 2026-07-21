---
name: author
description: >-
  Generate — or regenerate — one section's content — README.md, theory.md, practice.md,
  validation.md, quiz.md — via the drafter + 4-critic quorum with a revise-until-pass loop.
  Use when the user says "generate section N.K", "author section N.K", "regenerate section
  N.K", "redo section N.K", "generate module N", "regenerate module N", or runs /author N.K
  (one section) or /author N (loop over a module, with an upfront cost warning). Re-running
  on a section already `generated`/`studied` is how content gets regenerated — there is no
  separate regenerate command — but it requires explicit confirmation before overwriting.
  Critic model (default sonnet) and max revise iterations (default 2, `--fast` for 0) are
  overridable per run via `model=` and `iterations=` args. Refuses on modules that aren't
  approved/planned. Side-effect command (spends tokens, writes files) — user-invoked only.
disable-model-invocation: false
argument-hint: "N.K | N  [model=opus|sonnet|haiku] [iterations=0-3 | --fast]"
---

# /author N.K — produce a section through the quorum

Generate the five artifacts for one section using the pipeline in
[`references/generation-pipeline.md`](references/generation-pipeline.md). Read that file —
it is the authoritative procedure, including the revise loop and the surface-to-learner rule.

## Argument handling
- `N.K` → generate that one section.
- `N` → generate **every planned section** in module N in order. First print an **upfront
  cost warning** (how many sections, the resolved critic model, and the resolved max revise
  iterations) and get a confirm before looping.
- Missing → ask which section.
- **Optional modifiers**, parsed from `$ARGUMENTS` alongside the required `N.K`/`N`, in any
  order:
  - `model=opus|sonnet|haiku` — critic model for this run (all four critics; the drafter
    always stays on its own default). Default: `sonnet`.
  - `iterations=0|1|2|3` — max revise-loop rounds for this run (see the pipeline reference).
    Default: `2`. `0` means the critics run once and nothing is auto-revised — objections are
    surfaced immediately.
  - `--fast` — shorthand for `iterations=0`.
  - Neither modifier changes what the critics check — only how much model tier and how many
    revise attempts are spent verifying it. State the resolved values back to the learner
    whenever they differ from the defaults.

## Regenerating (already-`generated`/`studied` content)
This is the *only* path to regenerate a section — there is no separate `/regenerate`
command. Before drafting, check the target section(s)' current **section status** in the
module `README.md`:
- **`planned`** → proceed normally, no extra confirmation needed.
- **`generated` or `studied`** → this run will **overwrite** existing content and, for a
  `studied` section, **un-certify mastery back to `generated`**. STOP and confirm explicitly
  before proceeding: name the affected section(s) and their current status, and say plainly
  that mastery status resets. Only continue on an explicit yes.
- For `N` (module scope): if *some or all* sections in the module are already `generated`+,
  fold them into the same upfront cost warning as the fresh-generation loop — list every
  section that will be touched with its current status, get one confirmation covering all of
  them, then loop through **every** section in the module (not just the still-`planned`
  ones), applying the per-section confirm rule above implicitly (already covered by the
  batch confirmation).
- If the learner declines, leave the section(s) untouched and stop.

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
  `${CLAUDE_SKILL_DIR}/../conventions/SKILL.md`). Pass the **full** text inline to
  `section-drafter`; pass each critic only its own scoped excerpt (see the pipeline
  reference's Step 0) — subagents run from the topic-repo cwd and cannot open plugin-internal
  files by relative path, and blasting the full file to every critic wastes tokens on
  sections it doesn't judge.
- Topic `CLAUDE.md`, `BASELINE.md`, `ENVIRONMENT.md`.
- The module `README.md` (this section's goal within the plan).
- **Prerequisite sections' `theory.md`** — only the prerequisites the roadmap lists for this
  module. Do not read the whole repo.
- Current `workspace/` state relevant to the capstone increment.

## Run the pipeline (summary — details in the reference)
Pass every agent the **section directory path** (`modules/NN-slug/sections/0K-slug/`) plus its
scoped conventions excerpt — not the artifact bodies. Content flows through files on disk, not
through agent prompts.
1. **Draft** — invoke `section-drafter` with the inputs above (section dir path + the **full**
   conventions text **inlined**). It **WRITES** the five files
   (`README/theory/practice/validation/quiz.md`) directly to the section directory and returns
   a confirmation (plus a change summary on revision passes).
2. **Critique in parallel** — invoke `critic-accuracy`, `critic-freshness`, `critic-depth`,
   `critic-pedagogy` (one message, parallel), each with only its own conventions excerpt, and
   with the resolved `model=` override if the learner set a non-default one. Each **READS** the
   four content files (not the README — it's navigational, not judged) from the section dir and
   returns PASS or numbered objections.
3. **Revise loop** — unless the run is `--fast`/`iterations=0`, if any critic FAILs, send all
   objections back to `section-drafter` (it edits the files in place and returns a change
   summary); re-critique per the **deterministic re-run rule** in the reference. **Max `N`
   iterations (default 2, learner-configurable).**
4. **Surface, never hide** — if objections remain unresolved after `N` iterations (or
   immediately, for `--fast`), the files stay on disk BUT you present the remaining objections
   to the learner explicitly: **prepend** a `> ⚠️ Unresolved critic objections` block to the top
   of each affected file (the orchestrator writes it) and list them in your reply. Never
   silently drop or auto-resolve them.

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
- If this was a **regeneration** of a previously `studied` section, say explicitly: mastery
  reset to `generated` (re-`/check N.K` to re-certify); retention cards from the old
  `quiz.md` are not retroactively edited — a fresh `/quiz N.K` layers new cards on top, so
  suggest reviewing `retention/deck/` for that section if the content changed materially.
  If `workspace/` already holds practice work for this section, note it may no longer match
  the regenerated `practice.md` — suggest diffing before continuing.
