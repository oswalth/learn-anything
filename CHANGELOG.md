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

## [0.6.0] — 2026-07-06

Added a root `CLAUDE.md` — the plugin repo's own operating manual — so its maintenance
discipline is always loaded, not just recalled when someone happens to run `/evolve` (minor —
new always-on behavior via an auto-loaded file; no command/format change).

- **Added `CLAUDE.md`** at the repo root: the one-hard-rule statement that all changes go
  through `/evolve`/`/retro`, the version-bump + changelog requirement (with the semver
  table), the "README must move in lockstep" rule (with a grep to catch stale command names),
  a condensed pointer to the skill/agent authoring rules in
  `skills/evolve/references/authoring-guide.md` (not duplicated), the blast-radius-check
  reminder, a note that `SPEC.md` is a frozen decision record the code/changelog supersede,
  the no-bundled-example-topic rule, and a repo map. Explicitly distinguishes itself from the
  topic-repo `CLAUDE.md` template in `skills/kickoff/assets/CLAUDE.md`, which is a different
  document for a different audience.
- **`README.md`** repository-layout tree now lists `CLAUDE.md`.
- **`marketplace.json`**'s mirrored `plugins[0].version` bumped alongside `plugin.json`, per
  the new sync rule this same change documents.
- *Why:* the plugin already had every one of these rules — scattered across `README.md`,
  `SPEC.md §10`, `skills/evolve/references/authoring-guide.md`, and `skills/conventions/
  SKILL.md §5` — but nothing surfaced them to a Claude session (or a human) that opens this
  repo directly without going through `/evolve` first. A root `CLAUDE.md` is what Claude Code
  auto-loads every session, closing that gap without duplicating the source-of-truth text
  anywhere.

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
