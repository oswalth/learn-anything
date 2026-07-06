# Changelog

All notable changes to the `learn-anything` plugin are recorded here. This file mirrors
the "keep this file honest" discipline that scaffolded topic repos carry: **every change
made through `/evolve` MUST add an entry** (date, what, why). `/evolve` and topic-level
`/retro` are the *only* sanctioned mutation paths, so this changelog is a first-class
history of the system's own evolution.

Versioning is semver (see [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json)):

- **major** — breaking format changes to topic-repo files (existing topics need migration).
- **minor** — new commands or new behavior.
- **patch** — fixes.

Major bumps MUST include a migration note for existing topic repos.

## [0.11.0] — 2026-07-06

Added regeneration support at all three content granularities. Learner feedback (from
`learn-aws-ion`) was that content needs periodic refresh for reasons that have nothing to do
with it being wrong — a plugin update landed a fresher process, an input changed (e.g. an
AWS account that didn't exist at kickoff time now does), or the learner's goal itself moved
— and there was no sanctioned way to regenerate short of hand-editing files (minor — new
command + new behavior on two existing commands, no topic-repo file *format* changes, so no
migration needed). Originating topic: `learn-aws-ion`.

- **New `skills/replan-roadmap/SKILL.md`.** Regenerates `roadmap.md` through the same
  `roadmap-drafter` ×2 + `roadmap-judge` quorum `/kickoff` uses, against the topic's current
  `CLAUDE.md`/environment/conventions. Protects modules already
  `approved`/`planned`/`in-progress`/`done` by default — only `draft` and net-new modules are
  freely redesigned; any conflict with a protected module is surfaced as a
  "Protected-module conflicts" note rather than silently applied. Re-gates at learner
  approval exactly like `/kickoff`'s D8 hard stop. Justified as a new skill (rather than
  folding into an existing one) because no existing command's job covers it: `/kickoff`
  structurally refuses to run on an existing topic, and `roadmap.md` is a *persistent*
  artifact (unlike the five disposable section files `/author` overwrites), so it needed its
  own gated, diff-reviewed procedure.
- **`agents/roadmap-drafter.md`, `agents/roadmap-judge.md`.** Now documented as shared
  between `/kickoff` and `/replan-roadmap`; both gained a "regeneration mode" input
  (current `roadmap.md` + a PROTECTED/FREE module classification) and instructions to leave
  PROTECTED modules' fields untouched, flagging conflicts instead of silently resolving them.
- **`skills/author/SKILL.md`, `skills/author/references/generation-pipeline.md`.** Re-running
  `/author N.K` (or `/author N`) on a section already `generated`/`studied` is now the
  explicit regeneration path — no separate command. It requires confirming before
  overwriting, warns that regenerating a `studied` section un-certifies mastery back to
  `generated`, and notes that stale retention cards aren't retroactively edited and
  in-progress `workspace/` work may no longer match a regenerated `practice.md`.
- **`skills/plan-module/SKILL.md`.** Re-planning an already-`planned`+ module now diffs the
  proposed section list against the current one by id/slug and requires separate
  confirmation before removing, renumbering, or materially changing the goal of any section
  already `generated`/`studied` — re-planning can no longer silently orphan authored content.
- **`skills/kickoff/SKILL.md`, `skills/kickoff/assets/roadmap.md`.** Both now point to
  `/replan-roadmap` for post-approval roadmap changes instead of leaving it undocumented.
- **`skills/evolve/references/authoring-guide.md`.** `replan-roadmap` added to the
  `disable-model-invocation: true` roster (seven side-effecting commands now) and to the
  soft plugin-compatibility-check list.
- **`README.md`** command reference table and §7 updated for `/replan-roadmap` and the
  amended `/author`/`/plan-module` regeneration behavior.
- **Smoke-tested the new agent mechanism directly** (`kickoff`/`author`/`plan-module`/
  `replan-roadmap` are all `disable-model-invocation: true` by design, so they can't be
  self-triggered to test end-to-end in one sitting; the `/author`/`/plan-module` confirm/diff
  logic added above is orchestration-only text, verified by inspection). Invoked
  `roadmap-drafter` ×2 and `roadmap-judge` directly (Agent tool) with a constructed
  regeneration scenario — a 2-module roadmap with Module 01 `approved` (PROTECTED) and
  Module 02 `draft` (FREE), and a changed goal (added cross-account access). Both drafters
  correctly reproduced Module 01 verbatim, freely redesigned/expanded the FREE module into a
  multi-phase cross-account track, and used the "Protected-module conflicts" escape hatch
  instead of silently editing Module 01 when they judged it relevant. The judge merged both
  drafts, preserved Module 01, and de-duplicated the conflict notes as designed — **but**
  rendered the protected module's status as `approved *(PROTECTED — reproduced verbatim...)*`
  instead of a bare `approved`, which would have broken every other skill's exact-string
  status match (`/plan-module`, `/author`, `/check`) had it been written to disk. Fixed by
  adding an explicit bare-status-value requirement to `roadmap-drafter.md` and
  `roadmap-judge.md`, plus a defensive strip-before-write check in `/replan-roadmap` Step 5.

## [0.10.0] — 2026-07-06

Added a seventh, conditional element to the Content depth standard — **Visualize it** —
for concepts whose mental model is structural, sequential, or relational (an evaluation
flow like IAM policy evaluation, an account/org hierarchy, a request path, a network
topology, a state machine). Learner feedback (from `learn-aws-ion`) was that even sections
hitting all six depth-standard elements still read as flat prose for these concept types
without a diagram (minor — new depth-standard clause and drafter/critic behavior change, no
topic-repo file format changes, so no migration needed). Originating topic: `learn-aws-ion`.

- **`skills/conventions/SKILL.md` §1.** New clause 7: an ASCII diagram or fenced
  ` ```mermaid ` block, placed alongside the Mechanism block (§1.3) as a supplement (never a
  replacement for the prose). Explicitly conditional — skipped for concepts that are
  genuinely flat/scalar (a single value, a config flag, a non-branching API call), with no
  objection raised for that omission.
- **`agents/section-drafter.md`.** `theory.md` generation instructions now include the
  conditional diagram, with guidance to prefer Mermaid for named-node/directed-edge shapes
  (flowcharts, sequences, hierarchy graphs — renders inline on GitHub and most markdown
  viewers) and ASCII for simple box-and-arrow shapes or where Mermaid syntax doesn't fit.
- **`agents/critic-depth.md`.** New check item 7: judges applicability itself per concept —
  objects to a missing diagram only when the concept plainly has structure/flow/hierarchy to
  draw, never invents structure to justify an objection. Frontmatter description updated to
  match.
- **Cross-reference wording** updated for consistency: `skills/author/references/generation-pipeline.md`,
  `skills/patch/SKILL.md` (the **thin** classification now covers a missing applicable
  diagram), and `README.md`'s depth-standard summary bullet.
- Smoke-tested against a throwaway scratch topic (`mktemp -d`, IAM policy evaluation as the
  structural case and the policy `Version` field as the deliberately flat case, run directly
  through `section-drafter` + `critic-depth` rather than a full `/kickoff`, since the change
  is scoped to drafting/critique behavior, not roadmap generation): the drafter emitted an
  ASCII diagram for the structural concept and correctly omitted one for the flat concept,
  and `critic-depth` PASSed both, correctly not objecting to the flat concept's missing
  diagram.

## [0.9.0] — 2026-07-06

Added `/patch N.K "note"`, an on-demand, bounded correction command — a learner who flags a
wrong or outdated claim (or asks for more depth on one concept) mid-mentoring or mid-study gets
it fixed and re-verified immediately, instead of only being caught by the critic quorum at
`/author` time or waiting for a full regeneration (minor — new command, no topic-repo file
format changes, so no migration needed). Originating topic: `learn-aws-ion`.

- **New skill `skills/patch/SKILL.md`.** `disable-model-invocation: true` (it mutates section
  content, matching the plugin's mutating-command pattern). Classifies the learner's flag as
  **wrong** / **outdated** / **thin**, edits only the flagged passage directly with the Edit
  tool (never invokes `section-drafter` — this stays a scalpel, not a redraft), then verifies
  with exactly one matching critic (`critic-accuracy` / `critic-freshness` / `critic-depth`),
  max 2 revise-and-reverify rounds. Never changes section/module status or touches
  `workspace/`/`retention/` — `/check` alone owns mastery transitions.
- **`agents/critic-accuracy.md`, `agents/critic-freshness.md`, `agents/critic-depth.md`.**
  Descriptions now note they're also invoked stand-alone by `/patch`; each gained a short
  "When invoked by /patch" section instructing the verdict to scope to the edited passage
  named in the prompt, while unprompted observations about the rest of the file are surfaced
  but non-blocking. `critic-pedagogy` is unchanged — `/patch`'s three flag types don't map to
  it.
- **`skills/mentor/SKILL.md`, `skills/study/SKILL.md`.** One guardrail line each: on a
  wrong/outdated/thin flag, point the learner to `/patch N.K` rather than editing themselves —
  both commands' no-edit identity is otherwise unchanged.
- **`README.md`.** New `/patch` row in the command table; a sentence under "What makes the
  content different" describing the post-authoring correction path; `study` and `patch` added
  to §1's command list (the former was a pre-existing omission from the `/study` addition in
  0.7.0, fixed here since the same line was already being edited).

## [0.8.0] — 2026-07-06

`/evolve`'s Step 8 (ship) now tags the commit with an annotated git tag matching the new
version (`v<version>`) in addition to committing, so the plugin's version history is
recoverable from `git tag` and not just from `CHANGELOG.md` prose (minor — new behavior,
no topic-repo file format changes, so no migration needed).

- **`skills/evolve/SKILL.md` Step 8.** After committing, run
  `git tag -a v<version> -m "<one-line changelog summary>"`, then remind the learner to
  push both the branch and the tag (`git push && git push --tags`).
- **`README.md` §6.** Notes that shipping an `/evolve` change tags the commit and that the
  push step now includes tags.
- **No backfill.** The 6 versions already shipped (0.1.1 → 0.7.0) are left untagged by
  design — this change only affects future ships.

## [0.7.0] — 2026-07-06

Added `/study N.K`, a paced walkthrough command that interleaves a section's `theory.md`
concepts with the `practice.md` challenges that build on them — waiting for the learner
between each step — instead of handing over four static files and leaving them to work
solo (minor — new command, plus an additive convention/generation-contract change).

- **New skill `skills/study/SKILL.md`.** Reads a section's `README.md` (prerequisites, the
  Order sequence, the next-section pointer) plus `theory.md`/`practice.md`/`validation.md`/
  `quiz.md`, and paces the learner through them: theory heading → the challenge(s) that build
  on it → next heading, ending with pointers to `/check N.K` and `/quiz N.K`. It only
  sequences — it does not teach Socratically (`/mentor`'s job) or grade correctness
  (`/check`'s job), and never writes to `workspace/` or any status field.
- **New practice-philosophy clause (`skills/conventions/SKILL.md` §2).** Every worked
  example and challenge in `practice.md` now carries a `**Builds on:** theory.md §
  "<heading>"` line naming the theory.md heading(s) it draws on — the data `/study` uses to
  interleave precisely instead of guessing. `agents/section-drafter.md` now writes this tag;
  `agents/critic-pedagogy.md` now FAILs a missing tag or one naming a heading absent from
  `theory.md`; `skills/author/references/generation-pipeline.md`'s critic-pedagogy summary
  mentions it. Sections generated before this change lack the tag — `/study` detects that and
  falls back to the coarser `README.md` Order sequence for them, so no existing topic needs
  migration.
- **Trigger-phrase fix.** `/mentor`'s description previously claimed "walk me through this",
  which is genuinely ambiguous between on-demand help and paced sequencing. Moved that phrase
  (and its variants) to `/study`; `/mentor` now lists "explain N.K to me" instead, and each
  skill's guardrails point at the other for the case it doesn't handle.
- **`README.md`** command table gains a `/study N.K` row (between `/author` and `/check`);
  the daily-loop walkthrough shows `/study 1.1` right after `/author 1.1`; the Practice
  philosophy bullet under "What makes the content different" notes the new theory-linking tag.
- *Why:* learner feedback (from `learn-aws-ion`, via a pasted `/evolve` brief) was that the
  plugin "generates content and leaves a user 1-to-1 with it — it doesn't feel like a guided
  agentic learning flow." `/mentor` already covered on-demand help, but nothing paced a
  learner *through* a section step by step. The precise variant (tagged theory↔challenge
  links, chosen over a purely heuristic file-order pacing) was picked deliberately: the
  heuristic alone risks silently mispairing a challenge with the wrong concept, which is worse
  for a first-time learner than a well-labeled coarse fallback.

## [0.6.0] — 2026-07-06

Added a root `CLAUDE.md` — the plugin repo's own operating manual — so its maintenance
discipline is always loaded, not just recalled when someone happens to run `/evolve` (minor —
new always-on behavior via an auto-loaded file; no command/format change). Bundled in the same
release: `section-drafter` now always emits a fifth, navigational `README.md` per section
(minor — new file in the generation contract, additive/non-breaking for existing sections).

- **Added `CLAUDE.md`** at the repo root: the one-hard-rule statement that all changes go
  through `/evolve`/`/retro`, the version-bump + changelog requirement (with the semver
  table), the "README must move in lockstep" rule (with a grep to catch stale command names),
  a condensed pointer to the skill/agent authoring rules in
  `skills/evolve/references/authoring-guide.md` (not duplicated), the blast-radius-check
  reminder, a note that `SPEC.md` is a frozen decision record the code/changelog supersede,
  the no-bundled-example-topic rule, and a repo map. Explicitly distinguishes itself from the
  topic-repo `CLAUDE.md` template in `skills/kickoff/assets/CLAUDE.md`, which is a different
  document for a different audience.
- **`section-drafter` now writes a fifth artifact, `README.md`**, alongside `theory.md`,
  `practice.md`, `validation.md`, `quiz.md` — a short navigational front door written last,
  once the other four are final: a "Before you start" prerequisite checklist (pointing to
  exactly where each prerequisite is actually handled, even if `practice.md`'s own Setup block
  also covers it) and an "Order" section with the concrete sequence to work the other four
  files in plus rough time budgets, ending with a pointer to the next section (or `/check N`
  if it's the module's last one). README.md is explicitly excluded from the four critics'
  read list — it's navigational, not judged content, so it never triggers a critic re-run.
  `skills/author/SKILL.md`, `skills/author/references/generation-pipeline.md`, and
  `skills/conventions/SKILL.md`'s disposable-artifacts list were updated to match.
- **`README.md`** repository-layout tree now lists `CLAUDE.md`; its daily-loop walkthrough now
  mentions generating and starting from `README.md`.
- **`marketplace.json`**'s mirrored `plugins[0].version` bumped alongside `plugin.json`, per
  the new sync rule this same change documents.
- *Why (CLAUDE.md):* the plugin already had every one of these rules — scattered across
  `README.md`, `SPEC.md §10`, `skills/evolve/references/authoring-guide.md`, and
  `skills/conventions/SKILL.md §5` — but nothing surfaced them to a Claude session (or a
  human) that opens this repo directly without going through `/evolve` first. A root
  `CLAUDE.md` is what Claude Code auto-loads every session, closing that gap without
  duplicating the source-of-truth text anywhere.
- *Why (README.md artifact):* a learner found section 01.1 of `learn-aws-ion` disorienting to
  start cold — the AWS-account prerequisite was buried inside `practice.md`'s Setup block
  instead of being the first thing surfaced, and nothing stated the intended file order. That
  topic had already validated the fix as a per-topic override (`.claude/agents/
  section-drafter.md`); this promotes it into the plugin default via `/evolve` so every topic
  gets it, not just the one that discovered the need.

## [0.5.2] — 2026-07-06

Made the plugin actually installable and documented the full lifecycle (patch — packaging +
docs; no command/behavior change).

- **Added `.claude-plugin/marketplace.json`** — a marketplace manifest listing the
  `learn-anything` plugin (`source: "."`). Claude Code installs plugins *from a marketplace*,
  not from a raw repo URL, so without this manifest `/plugin install …` failed with
  "Marketplace not found". Install is now `/plugin marketplace add oswalth/learn-anything`
  then `/plugin install learn-anything@learn-anything`.
- **Rewrote `README.md`** into a no-questions-left guide: install (GitHub + local + CLI),
  update (marketplace-cache vs installed-plugin mental model), manage/enable/disable/
  uninstall, per-topic setup walkthrough, full command reference, restore-on-another-machine,
  and a troubleshooting section (including the exact "Marketplace not found" error).
- **Topic `CLAUDE.md` plugin-dependency section** (kickoff asset + skill) now records the
  marketplace repo and the exact restore commands, not just a bare URL.
- *Why:* the plugin was not installable as written, and install/update steps were wrong or
  missing.

## [0.5.1] — 2026-07-06

Made two advisory-only fields honest about what they guarantee (patch — wording fix plus one
small real compatibility warning; no interface change).

- *What:* `/author` and `/plan-module` now do a **soft min-plugin-version compatibility check**
  at start — they read the topic `CLAUDE.md`'s recorded **Minimum version** and compare it to
  the installed plugin version in `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`; if the
  installed plugin is older, they WARN the learner (suggest `/plugin` update) and ask whether to
  proceed — a warning, not a hard block. Reframed **spend-cap** language across
  `ENVIRONMENT.md` (kickoff asset), `conventions`, `mentor`, and `check` from
  "system-enforced / hard ceiling / no exercise may exceed" to an honest split: the **model**
  is instructed to design exercises within the cap (kept — the right instruction), but the
  **system does not guarantee** it, so billing alarms + teardown (already mandated as the first
  cloud exercise) are the real backstop. Updated the `CLAUDE.md` "Minimum version" wording and
  added a compatibility/honesty note to the authoring guide.
- *Why:* both fields previously implied enforcement that nothing actually performed — the min
  plugin version was recorded but never read, and an LLM instruction to respect the cap is not
  system enforcement.

## [0.5.0] — 2026-07-06

Hardened the `/author` generation pipeline: content now flows through files on disk, and the
critic re-run rule is deterministic (minor — pipeline behavior change).

- *What:* `section-drafter` now **writes** the four section files (`theory/practice/validation/
  quiz.md`) directly to `modules/NN-slug/sections/0K-slug/` (gained the `Write` tool), and the
  four critics **read** them from that directory instead of receiving the bodies inline — the
  orchestrator passes each agent only the section directory path plus the inlined conventions
  text. Replaced the fuzzy re-run guidance (re-run just the failed critics, re-run affected ones
  when unsure) with a **deterministic re-run rule** keyed to what the revision changed: always
  re-run failed critics, plus accuracy on any code/command/example/output change, freshness on
  any version/tool/default/API change, depth on any `theory.md` change, pedagogy on any
  `practice.md` change (bias toward re-running when uncertain). The drafter now returns a
  **change summary** each revision so the orchestrator can apply the rule mechanically. The
  `/author` "write" step becomes verify-the-four-files-exist + bookkeeping; unresolved objections
  are still surfaced, with the orchestrator prepending the ⚠️ block to the on-disk files.
- *Why:* passing "no length limit" depth content as inlined text through the drafter and four
  critic boundaries was truncation-prone and token-heavy; writing once to disk and reading back
  removes that fragility. The old re-run-affected-when-unsure rule was too fuzzy — a depth-fix
  that introduced an inaccuracy could slip past an already-passed accuracy critic; the
  deterministic rule closes that cross-domain gap.

## [0.4.0] — 2026-07-06

Opened the five interactive, read-mostly commands to Claude auto-invocation while keeping the
side-effecting ones gated (minor — behavior change: five commands become model-invocable).

- *What:* removed `disable-model-invocation: true` from `mentor`, `quiz`, `recall`, `check`,
  and `checkpoint`, so Claude can auto-invoke them from natural language; `kickoff`, `author`,
  `plan-module`, `update-baseline`, `retro`, and `evolve` stay user-invoked only. Dropped the
  now-false "User-invoked only" phrase from the five opened descriptions, and reconciled the
  authoring guide's `disable-model-invocation` guidance to document the split and its principle
  (side-effecting/costly/mutating ⇒ gated; cheap/interactive/read-mostly ⇒ open).
- *Why:* the blanket `disable-model-invocation: true` across all commands nullified the
  plugin's own "pushy description = trigger" rule — the elaborate "use when the user says…"
  prose was dead weight. Opening the safe read-mostly commands lets natural phrasing like
  "quiz me" or "review my flashcards" work, while costly/mutating commands remain gated behind
  an explicit learner trigger. `check`/`checkpoint` only advance status on explicit learner
  confirmation and never touch workspace code, so their auto-invocation is safe.

## [0.3.0] — 2026-07-06

Split the status state-machine into two distinct vocabularies and gave the terminal states
owners (minor — behavior change: new status semantics + new transitions owned by `/check`).

- *What:* the single ambiguous vocabulary `draft → approved → planned → generated →
  in-progress → done` (used for *both* module and section status) is now two vocabularies:
  - **Module status** (`roadmap.md`, `modules/README.md` board): `draft → approved → planned
    → in-progress → done`. `generated` is **removed** from the module vocabulary.
    Transitions: `draft→approved` = `/kickoff` approval; `approved→planned` = `/plan-module
    N`; `planned→in-progress` = the first `/author N.K` in the module; **`in-progress→done` =
    a passing whole-module `/check N`** (a mastery claim, with learner confirmation).
  - **Section status** (each `modules/NN-slug/README.md` section plan): `planned → generated
    → studied`. `planned` = `/plan-module`; `generated` = `/author N.K` after the four files
    are produced; **`studied` = a passing per-section `/check N.K`** (with learner
    confirmation — previously nothing ever marked a section studied).
  - `/check` now **owns** the two mastery transitions (`section→studied`, `module→done`) and
    advances them only on a genuine PASS with the learner's confirmation; it still never edits
    the learner's `workspace/` code. Revised the old "Do not change the section status
    yourself" rule accordingly.
  - Files touched: `skills/kickoff/assets/roadmap.md`, `skills/kickoff/assets/CLAUDE.md`,
    `skills/kickoff/SKILL.md`, `skills/kickoff/references/interview-script.md`,
    `skills/plan-module/SKILL.md`, `skills/author/SKILL.md`,
    `skills/author/references/generation-pipeline.md`, `skills/check/SKILL.md`,
    `skills/checkpoint/SKILL.md`, `skills/update-baseline/SKILL.md`. `update-baseline`'s
    "generated-but-unstudied" now precisely means section status `generated` (not yet
    `studied`).
- *Why:* the old single vocabulary was ambiguous — `generated` meant different things for a
  module vs a section, and "module is generated" was meaningless when only some sections were
  done. Worse, **no command ever wrote `done`**: it was a dangling terminal state with no
  owner. Splitting the vocabularies and assigning the mastery transitions to `/check` (where
  validation, not generation, belongs) makes the state-machine complete and unambiguous.
- *Migration (no real topics exist yet):* this is a **semantics** change to the status lines
  in `roadmap.md` / module-READMEs, not a file-format change. Any pre-existing topic would
  reinterpret a module-status of `generated` as `in-progress`, and section plans keep using
  `generated` (now explicitly a section status). No structural migration required.

## [0.2.1] — 2026-07-06

Made the shared conventions actually locatable (patch — bugfix to the dedup mechanism; no
topic-repo format change, no migration).

- *What:* the orchestrating skills now **resolve the conventions file by absolute path** —
  `${CLAUDE_PLUGIN_ROOT}/skills/conventions/SKILL.md` (documented plugin-install dir), with
  `${CLAUDE_SKILL_DIR}/../conventions/SKILL.md` as a fallback — read it themselves, and
  **inline its text into each agent's prompt**. Updated `skills/author/SKILL.md` +
  `skills/author/references/generation-pipeline.md` (Step 0) and `skills/kickoff/SKILL.md` +
  `skills/kickoff/references/interview-script.md`. The agents (`section-drafter`,
  `critic-depth`, `critic-pedagogy`, `roadmap-drafter`) no longer say "read
  `skills/conventions/SKILL.md` at the path given" / "read them if a path is given"; they now
  treat the **inlined** conventions as the normative source (§-number references kept). Added a
  "Locating plugin-internal files" subsection to `skills/evolve/references/authoring-guide.md`.
- *Why:* the previous instruction referenced a bare relative path that is **unresolvable** — a
  command runs from the topic-repo cwd (where `skills/conventions/SKILL.md` does not exist; the
  plugin is installed elsewhere), and subagents can't read plugin-internal files by relative
  path either. The "conventions live once, everyone references them" claim rested on a path
  that could never be located.

## [0.2.0] — 2026-07-05

Renamed three commands to distinctive, collision-free names (minor — user-facing
command-interface change). **These are the new invocation names; use them from now on.**

- **`/generate` → `/author`** (skill dir `skills/generate/` → `skills/author/`).
- **`/validate` → `/check`** (skill dir `skills/validate/` → `skills/check/`).
- **`/review` → `/recall`** (skill dir `skills/review/` → `skills/recall/`, including
  `references/scheduling.md`).
- *What else changed:* frontmatter `name:` in each moved skill, and every live
  cross-reference across `README.md`, `skills/**`, and `agents/*` — slash-command tokens
  (`/author N.K`, `/check N.K`, `/recall`), the checkpoint skill's pointer to
  `skills/recall/references/scheduling.md`, the `/help` command list, and the quickstart
  steps. Natural-language trigger phrases ("review", "validate my work", "generate section")
  are **kept in the renamed skills' descriptions** for discoverability, since users still
  say those words.
- *Why:* `/review` collides with Claude Code's built-in review / code-review, and
  `generate`/`validate` are over-generic verbs that clash with built-ins and other plugins,
  hurting disambiguation. Done now, while no real topic repos exist, so there is nothing to
  migrate.

## [0.1.1] — 2026-07-05

Content removal + doc alignment; no behavior or command-interface change (patch).

- **Removed `examples/`** (the bundled `uv-packaging-topic/` toy topic) — a version-pinned
  baseline that would never be re-verified, contradicting the freshness doctrine the plugin
  exists to enforce. A plugin is a *repeatable pattern*; `/kickoff`'s `assets/` skeletons
  already document the topic-repo file structure.
- **Removed `docs/`** (`chorequest-CLAUDE.md`) — a third copy of the depth standard and
  practice philosophy, which live once in `skills/conventions/SKILL.md` and `SPEC.md §9`.
  Violated the plugin's own "live once" rule.
- **Retargeted `/evolve` Step 6 smoke test** from the bundled toy topic to a throwaway
  scratch topic the maintainer creates in a temp dir (`mktemp -d` + `/kickoff`); updated the
  skill description and `README.md` accordingly.
- **Amendment banner added to `SPEC.md`** noting it is the frozen original design and that
  the skills/agents plus this changelog are the current source of truth.
- *Why:* dedup + freshness self-consistency — the plugin should not ship duplicated
  provenance content or a topic-specific baseline that will rot.

## [0.1.0] — 2026-07-05

Initial release. Built per `SPEC.md` (design conversation of 2026-07-05), generalizing the
proven "ChoreQuest / FastAPI Mastery" learning system into a reusable plugin.

- **Plugin scaffold** — `plugin.json` (v0.1.0), `README.md`, this changelog.
- **Conventions knowledge skill** (`skills/conventions/`) — the single home of the
  Content depth standard, Practice philosophy, evolution-lens rule, and changelog
  discipline, ported faithfully from ChoreQuest. Every generator/critic/validator
  references it instead of duplicating the text.
- **Agents** — `roadmap-drafter`, `roadmap-judge`, `section-drafter`, and the four
  critics (`critic-accuracy`, `critic-freshness`, `critic-depth`, `critic-pedagogy`).
- **Skills / commands** — `kickoff`, `plan-module`, `generate`, `validate`, `mentor`,
  `quiz`, `review`, `checkpoint`, `retro`, `update-baseline`, `evolve`.
- **Toy example topic** — `examples/uv-packaging-topic/` as living documentation.
