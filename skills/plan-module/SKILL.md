---
name: plan-module
description: >-
  Plan an APPROVED roadmap module into 1–2 h sections. Reads only the topic CLAUDE.md, the
  roadmap entry for module N, and its prerequisites' README notes; writes the section plan
  (id, goal, status) into modules/NN-slug/README.md. Cheap — no content generation. Use when
  the user says "plan module N", "break module N into sections", or runs /plan-module N.
  Refuses if the module is not yet approved. User-invoked only.
disable-model-invocation: true
argument-hint: "N (module number)"
---

# /plan-module N — break an approved module into sections

Plan module **N** into an ordered set of sections, each sized to one **1–2 h** study session
(~30–40 min theory, ~60–80 min practice). This is the cheap step before generation; it does
NOT write any `theory.md`/`practice.md`/etc.

## Preconditions (check, then stop if unmet)
- **Plugin compatibility (soft).** Read the topic `CLAUDE.md`'s recorded **Minimum version**
  and compare it to the installed plugin version in
  `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`. If the installed plugin is **older** than
  the topic's recorded minimum, WARN the learner (this topic expects newer plugin machinery;
  suggest updating via `/plugin`) and ask whether to proceed. This is a warning, not a block.
- `N` is provided (`$ARGUMENTS` / `$0`). If not, ask which module.
- Read `roadmap.md`. Find module N. **If its module status is not `approved`** (i.e. still
  `draft`, or already `planned`/`in-progress`/`done`), refuse:
  - `draft` → tell the learner the roadmap module must be approved first (D8 gate).
  - `planned`+ → tell them it's already planned; offer to re-plan (which overwrites
    the section list but preserves their notes) only if they confirm.

## Read (context discipline — only these)
- Topic `CLAUDE.md` — goal, success criteria, adjacent-expertise contrasts, capstone spec.
- The `roadmap.md` entry for module N — Concept / Skills / Capstone increment / Prerequisites
  / Source.
- The `README.md` notes of module N's prerequisite modules (learner notes only — to avoid
  re-teaching what they've internalized and to build on it).
- The conventions skill's **session sizing** rule (§4) — do not read the whole conventions
  file; you only need the sizing bar.

## Produce
Write/overwrite `modules/NN-slug/README.md` with:
1. A short module header: title, the module's Concept and Skills (copied from roadmap), the
   capstone increment, and prerequisites.
2. A **section plan** — an ordered list, each entry:
   - `id` — `NN.K` and a slug (e.g. `03.2 routing-and-response-models`).
   - `goal` — one line: what the learner can do after this section.
   - `status: planned` — this is a **section status** (`planned` → `generated` → `studied`),
     distinct from the module status. `/author N.K` flips the section to `generated`; a
     passing `/check N.K` flips it to `studied`.
3. A `## Notes` section (empty, marked "learner notes — persists across regeneration").

Sizing rules:
- Each section must be exhaustive in one pass **and** fit the 1–2 h budget. If a subtopic
  can't be done justice in one session, make it two sections.
- Order by dependency and motivation within the module.
- Create the `sections/0K-slug/` directories now (empty) so the tree is ready for
  `/author`, OR leave that to /author — but at minimum the README section plan must
  list every section with a stable id/slug.

## After planning
- Set module N's **module status** in `roadmap.md` to `planned` (module vocabulary:
  `draft → approved → planned → in-progress → done` — never `generated`, which is a
  *section* status only) and update `modules/README.md` (the status board).
- Tell the learner to review the section plan and then run `/author N.1`.
- Do NOT generate content.
