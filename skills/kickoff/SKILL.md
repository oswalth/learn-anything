---
name: kickoff
description: >-
  Start a NEW learn-anything topic. Runs an interview wizard (goal → learner level →
  topic properties → prior-art scan → process recommendation → environment advisor →
  baseline pinning), then generates a gated roadmap and STOPS for approval before
  scaffolding. Use when the user says "kickoff", "start a new topic", "I want to learn
  X", "set up a learning repo for X", or runs /kickoff in an empty directory. This is a
  side-effect command (creates files, spends tokens on web research) — user-invoked only.
disable-model-invocation: true
argument-hint: "[topic name (optional)]"
---

# /kickoff — interview → gated roadmap → scaffold

You are running the kickoff wizard for a new learning topic in the **current directory**
(a topic repo is content-only; all machinery lives in this plugin). Your job: interview the
learner, research, draft a roadmap, and **stop at the roadmap for explicit approval** before
scaffolding anything else.

## Guardrails (read first)
- **One question at a time.** This is a conversation, not a form. Ask, wait, record, ask the
  next. Every answer is written immediately into the growing `CLAUDE.md` draft so nothing is
  lost.
- **Context discipline.** Read only what you need. If the directory is non-empty, check
  whether a topic already exists here (a `CLAUDE.md` + `roadmap.md`) and, if so, stop and
  tell the learner to pick an empty directory or continue an existing topic instead.
- **HARD STOP at the roadmap (D8).** Do NOT scaffold `modules/`, `retention/`, or
  `workspace/`, and do NOT generate any section content, until the learner *explicitly*
  approves the roadmap. The only files you create before approval are `CLAUDE.md`,
  `BASELINE.md`, `ENVIRONMENT.md`, and `roadmap.md`.
- Follow the **full question script** in
  [`references/interview-script.md`](references/interview-script.md) — it has the exact
  questions per phase. This file is the overview; the script is the detail.
- Use the process taxonomy + selection heuristics in
  [`references/process-types.md`](references/process-types.md) for Phase 5.
- Use the skeletons in [`assets/`](assets/) as the exact structure for the four files.

## Phases

Work through these in order. Detailed questions live in the interview script.

1. **Goal.** What to learn; why now; success criteria phrased as **capabilities** ("I can
   advise on X / build Y unaided / solve Z-class problems"); deadline/pace. → write the
   header + goal sections of `CLAUDE.md`.
2. **Learner relative to this topic.** Current level; **adjacent expertise to contrast
   against** (a first-class input — generalizes Django↔FastAPI; for non-code topics too);
   prior attempts and where they died. → write the contrasts + level sections.
3. **Topic properties.** Canonical book/source? How fast does the field move? Skill-like vs
   theory-like? What does *verification* look like here (tests pass / cluster converges /
   proof checks / sympy confirms)? → informs later phases.
4. **Prior-art scan (D10).** Web-research established curricula/books/paths (CS50, MIT OCW,
   teachyourselfcs, roadmap.sh, cert paths, canonical books). Produce a verdict —
   **adopt / adapt / generate** — with justification stating what the prior art covers
   well, where it is stale or misaligned with the goal, and why the verdict follows. →
   write the prior-art verdict section.
5. **Process recommendation (D1).** Using `references/process-types.md`, recommend a process
   **mix** with justification; where a capstone fits, propose **2–3 capstone candidates**;
   recommend the **practice medium** (D7: code repo / Jupyter / LaTeX-markdown problem sets
   / lab env). Learner approves or overrides. → write the process-mix + capstone sections.
6. **Environment & tooling advisor (D11).** Inventory first (hardware, OS, accounts,
   subscriptions), then a **budget ceiling** (one-time vs monthly), then a researched
   **options matrix** (option | one-time cost | monthly cost | setup complexity | learning
   value | risks) with a recommendation — learner chooses. Record a **spend cap** and a
   teardown checklist. For cloud topics, note that billing alarms + teardown are the first
   practice exercise. → write `ENVIRONMENT.md`.
7. **Baseline pinning (D2).** Fix versions, editions, and authoritative doc URLs, each with
   a verified-on date. → write `BASELINE.md`.
8. **Roadmap generation (D12).** Spawn **two `roadmap-drafter` agents in parallel** with the
   same inputs (the `CLAUDE.md` draft + the prior-art verdict + the **conventions text
   inlined** — read it yourself from `${CLAUDE_PLUGIN_ROOT}/skills/conventions/SKILL.md`,
   fallback `${CLAUDE_SKILL_DIR}/../conventions/SKILL.md`; the agents run from the topic-repo
   cwd and can't open plugin files by relative path). Then run **`roadmap-judge`** once to
   merge them, justifying every inclusion,
   exclusion, and ordering choice. Write `roadmap.md` with **every module `status: draft`**.
   (Use the Agent tool; run the two drafters in a single message so they go in parallel.)
9. **HARD STOP (D8).** Present the roadmap for review. Iterate conversationally on the
   learner's requests (merge/split/reorder modules, change scope) — regenerate `roadmap.md`
   as needed. **Only when the learner explicitly approves** do you: flip module statuses to
   `approved`, scaffold the rest of the tree (`modules/README.md` — the module-status board,
   module vocabulary `draft → approved → planned → in-progress → done`; `retention/deck/`,
   `retention/log.md`, `workspace/`), add an approval entry to the `CLAUDE.md` changelog,
   and tell the learner the next step is `/plan-module 1`.

## Writing the files
- Use `assets/CLAUDE.md`, `assets/BASELINE.md`, `assets/ENVIRONMENT.md`, and
  `assets/roadmap.md` as templates. Fill every placeholder; delete none of the required
  sections.
- The topic `CLAUDE.md` records the **plugin dependency**: the marketplace repo
  `oswalth/learn-anything`, the restore commands (`/plugin marketplace add …` +
  `/plugin install learn-anything@learn-anything`), and the minimum plugin version (the
  current version in `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`). It carries **no
  conventions text** — those live in the plugin.
- Copy the "Keep this file honest" changelog block verbatim into `CLAUDE.md`.

## When you finish (post-approval)
Confirm the tree is scaffolded and summarize: the goal, the process mix, the spend cap, and
the module count. Point the learner to `/plan-module 1`. Do not generate content.

For **later** roadmap changes (the goal or environment shifts, or refreshing against newer
plugin conventions), point the learner to `/replan-roadmap` rather than re-running
`/kickoff` — `/kickoff` refuses to run on a directory that already has a topic.
