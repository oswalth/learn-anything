# SPEC — `learn-anything`: a Claude Code plugin for AI-assisted self-learning

> **Amendments:** This is the original approved design. Behavior has since evolved via
> `/evolve`; the current source of truth is the skills/agents plus `CHANGELOG.md`. Where
> this document and the code disagree, the code and changelog win.

**Status:** approved design, ready to build.
**Origin:** designed interactively with Claude (claude.ai) on 2026-07-05, generalizing the
learner's proven "ChoreQuest / FastAPI Mastery" system (see `docs/chorequest-CLAUDE.md`,
referred to below as **CQ**). CQ's conventions are treated as validated-by-usage defaults.
**Learner profile:** senior software engineer; wants advise-level depth ("how it works",
not "how to use it"), mandatory evolution/history context, challenge-based practice.
Topics are unbounded — from Go/FastAPI/AWS/K8s/networking to non-code subjects
(e.g. differential equations).

---

## 1. What is being built

Two artifacts:

1. **The `learn-anything` plugin** — a versioned Claude Code plugin (`/plugin install`)
   containing ALL machinery: skills (slash commands), subagents, and shared conventions.
   One repo, one source of truth. Updating the plugin updates every topic at once.
2. **Topic repos** — thin, content-only directories scaffolded by `/kickoff` (no cloning).
   They contain zero machinery. A topic repo + the plugin = a complete learning system.

> **Why this split (decision record):** the original idea was "template repo cloned per
> topic", but that makes template improvements require N manual syncs across topic repos.
> Machinery therefore moves entirely into the plugin (edit once, applies everywhere;
> project-level `.claude/` can still shadow plugin definitions for per-topic overrides).
> Topic repos hold only content, which is per-topic by nature and never needs syncing.
> Trade-off accepted: a topic repo is not self-contained; its `CLAUDE.md` records the
> plugin repo URL + minimum plugin version so any machine can restore the setup.

## 2. Decision record (from the design conversation)

| # | Decision | Choice |
|---|----------|--------|
| D1 | Process-type selection | `/kickoff` interviews the learner, then **recommends** a process mix with justification; learner approves/overrides |
| D2 | Freshness | **Pinned baseline** (`BASELINE.md`) **+ mandatory web-research pass** by a freshness critic at generation time |
| D3 | Retention | Per-section quizzes + spaced-repetition deck + cumulative checkpoints. Lives **in-repo** (markdown deck) with **interactive Claude sessions** (`/quiz`, `/review`, `/checkpoint`). No Anki export in v1 |
| D4 | Packaging | **Proper plugin**, versioned, installed via `/plugin install` |
| D5 | Parallelism | Any number of concurrent topics; each topic repo is independent |
| D6 | Session sizing | One section ≈ one **1–2 h** session (~30–40 min theory, ~60–80 min practice) |
| D7 | Non-code topics | Practice/validation **medium recommended per topic** during kickoff (code repo / Jupyter / LaTeX-markdown problem sets / lab environment) |
| D8 | Roadmap gate | Roadmap is a reviewable deliverable; **no content generation until the learner approves it** |
| D9 | Token budgeting | Generation is granular: `/plan-module N` (cheap) and `/generate N.K` (one section per call) |
| D10 | Prior art | Mandatory **prior-art scan** during kickoff: adopt / adapt / generate verdict on existing curricula (CS50, MIT OCW, teachyourselfcs, roadmap.sh, cert paths…) with justification |
| D11 | Environment | Kickoff includes an **environment & tooling advisor** producing `ENVIRONMENT.md` (options scored by cost / complexity / learning value / risks; spend cap recorded and enforced) |
| D12 | Quorum shape | **Single drafter + parallel specialist critics** with revise-until-pass loop (max 3 iterations, then surface disagreements to the learner). Full **multi-draft quorum only for the roadmap** (2 independent drafts + judge) |
| D13 | Self-evolution | The plugin ships `/evolve` — an agentic, guarded workflow for changing the plugin itself |
| D14 | ChoreQuest defaults | CQ's Content depth standard, challenge-based Practice philosophy, section-based delivery, and "keep this file honest" changelog discipline are ported as plugin-level conventions |

## 3. Architecture

```
┌─────────────────────────────────────┐
│ learn-anything (plugin repo, git)   │  ← ALL machinery lives here
│  .claude-plugin/plugin.json         │     versioned; /evolve edits it;
│  skills/    (commands + conventions)│     push → every topic updates
│  agents/    (drafter, critics, …)   │
│  CHANGELOG.md                       │
└──────────────┬──────────────────────┘
               │ installed once (user scope)
   ┌───────────┴───────────┬─────────────────────┐
   ▼                       ▼                     ▼
 topic: golang         topic: aws            topic: diff-equations
 (content only)        (content only)        (content only)
```

- **Plugin scope:** user-level install so every topic sees it. Per-topic overrides:
  a topic repo MAY define project-level `.claude/skills/` or `.claude/agents/` files
  with the same names; project scope shadows plugin scope. `/retro` may propose such
  overrides; promoting an override into the plugin goes through `/evolve`.
- **Skills, not legacy commands:** implement slash commands as **skills** (`skills/<name>/SKILL.md`);
  the `commands/` directory is a legacy format.
- **Side-effect commands are user-invoked only:** `/kickoff`, `/generate`, `/evolve`,
  `/update-baseline` must set frontmatter so Claude cannot auto-invoke them
  (they cost tokens/money or mutate the plugin).

## 4. Plugin repo layout

```
learn-anything/
├── .claude-plugin/plugin.json        # name, version (semver), description
├── CHANGELOG.md                      # every /evolve change: date, what, why
├── README.md                         # install + quickstart + philosophy
├── skills/
│   ├── kickoff/SKILL.md              # + references/interview-script.md
│   │                                 # + references/process-types.md
│   │                                 # + assets/ (CLAUDE.md, BASELINE.md,
│   │                                 #   ENVIRONMENT.md, roadmap.md skeletons)
│   ├── plan-module/SKILL.md
│   ├── generate/SKILL.md             # + references/generation-pipeline.md
│   ├── validate/SKILL.md
│   ├── mentor/SKILL.md
│   ├── quiz/SKILL.md
│   ├── review/SKILL.md               # + references/scheduling.md (SM-2 variant)
│   ├── checkpoint/SKILL.md
│   ├── retro/SKILL.md
│   ├── update-baseline/SKILL.md
│   ├── evolve/SKILL.md               # + references/authoring-guide.md
│   └── conventions/SKILL.md          # depth standard, practice philosophy,
│                                     #   evolution-lens rule, changelog discipline
│                                     #   (Claude-invoked knowledge skill, not a command)
└── agents/
    ├── roadmap-drafter.md            # used twice in parallel for the roadmap
    ├── roadmap-judge.md
    ├── section-drafter.md
    ├── critic-accuracy.md
    ├── critic-freshness.md
    ├── critic-depth.md
    └── critic-pedagogy.md
```

`skills/conventions/` is the key de-duplication move: the depth standard and practice
philosophy live **once**, here, and every generator/critic/validator references them.
Topic `CLAUDE.md` files contain **only topic-specific** facts (goal, level, contrasts,
capstone, decisions, status) — this is what makes cross-topic sync a non-problem.

## 5. Topic repo layout (scaffolded by /kickoff)

```
<topic>/
├── CLAUDE.md            # topic identity: goal, learner level, adjacent-expertise
│                        #   contrasts, process mix, capstone spec, plugin repo URL
│                        #   + min version, decisions, status, changelog
├── BASELINE.md          # pinned versions, book editions, authoritative doc URLs,
│                        #   "verified on <date>" stamps
├── ENVIRONMENT.md       # chosen setup, options matrix kept for reference,
│                        #   SPEND CAP, teardown checklist
├── roadmap.md           # phases → modules; per-module status; approval gate
├── modules/
│   ├── README.md        # module status board
│   └── NN-slug/
│       ├── README.md    # section plan (id, goal, status) + learner notes — PERSISTS
│       └── sections/0K-slug/
│           ├── theory.md      # regenerable ┐
│           ├── practice.md    # regenerable │ disposable artifacts:
│           ├── validation.md  # regenerable │ re-running /generate N.K
│           └── quiz.md        # regenerable ┘ recreates them
├── retention/
│   ├── deck/*.md        # one file per card (format §6.5)
│   └── log.md           # review-session history
└── workspace/           # the capstone / notebooks / LaTeX — medium per topic
```

## 6. File formats

### 6.1 roadmap.md — module statuses (the D8 gate)

Per-module status line: `status: draft | approved | planned | generated | in-progress | done`.
`/kickoff` writes everything as `draft` and STOPS. The learner iterates conversationally
("merge modules 4–5", "move observability earlier"); roadmap edits regenerate the file.
Approval is explicit (learner says so; statuses flip to `approved`). `/plan-module` and
`/generate` MUST refuse to run on modules not yet `approved`. Post-approval roadmap
changes require a topic `CLAUDE.md` changelog entry (CQ's "keep this file honest" rule).

Each module entry carries the same spec fields CQ used, plus provenance:
**Concept / Skills / Capstone increment / Prerequisites / Source** — where Source is
`generated` or a mapping to prior-art units (e.g. `MIT OCW 18.01 Unit 2` or
`The Kubernetes Book ch. 5–6`) per the adopt/adapt verdict (D10).

### 6.2 Topic CLAUDE.md

Sections: header (topic, learner, method, pace, started, status) · "Keep this file
honest" changelog rule (copy CQ's block) · goal & success criteria (capability-phrased) ·
adjacent-expertise contrasts to lean on · process mix + justification · capstone spec
(if any) · prior-art verdict + justification · plugin dependency (repo URL, min version) ·
decisions · changelog. **No conventions text here** — that lives in the plugin.

### 6.3 BASELINE.md

Table: component | pinned version/edition | authoritative source URL | verified-on date.
`/update-baseline` re-verifies rows via web search, updates dates, and flags rows whose
new versions may invalidate generated-but-unread sections.

### 6.4 ENVIRONMENT.md

Chosen option + the full options matrix (option | one-time cost | monthly cost | setup
complexity | learning value | risks/limits) + **spend cap** + teardown checklist.
Generators and `/mentor` MUST NOT produce exercises exceeding the cap. For cloud topics,
billing alarms + teardown scripts are mandated as the first practice exercise.

### 6.5 Retention card format (retention/deck/*.md)

One markdown file per card, YAML frontmatter:

```yaml
---
id: 03-2-q4                # module-section-question
source: modules/03-fastapi-core/sections/02-routing
type: recall | why | contrast | code-prediction
status: active | suspended
interval_days: 4
ease: 2.5
due: 2026-07-12
history: [{date: 2026-07-05, grade: 3}]
---
Q: <the prompt — favors "why/when/tradeoff" questions over pure definitions>
A: <reference answer — bullet points of what a good answer must contain>
```

Scheduling: SM-2 variant (grades 0–5; grade <3 resets interval to 1 day; ease floor 1.3).
Full algorithm in `skills/review/references/scheduling.md`.

---

## 7. Command contracts

All commands are skills with `disable-model-invocation: true` unless noted.
Context discipline is a hard rule everywhere: each command reads ONLY the files its
contract lists — never "the whole repo".

### 7.1 /kickoff — interview wizard → scaffold → gated roadmap

Runs in an empty (or new) directory. One question at a time; every answer is recorded
immediately into the growing `CLAUDE.md` draft. Phases:

1. **Goal.** What to learn; why now; success criteria phrased as capabilities
   ("I can advise on X / build Y unaided / solve Z-class problems"); deadline/pace.
2. **Learner relative to this topic.** Current level; **adjacent expertise to contrast
   against** (generalizes CQ's Django↔FastAPI contrasts — a first-class input; for
   non-code topics too, e.g. "contrast analytic solutions with the numerical methods
   I know from programming"); prior attempts and where they died.
3. **Topic properties.** Does a canonical book/source exist and is it good? How fast
   does the field move? Skill-like vs theory-like? What does *verification* look like
   here (tests pass / cluster converges / proof checks / sympy confirms)?
4. **Prior-art scan (D10).** Web-research established curricula/books/paths. Verdict:
   **adopt** (roadmap maps onto the curriculum; theory switches to companion mode —
   pre-briefings, post-unit Socratic debriefs, evolution lens, currency patches;
   practice stays generated challenge-style) / **adapt** (curriculum as spine ± modules)
   / **generate** (no adequate prior art). Justification must state what the prior art
   covers well, where it is stale or misaligned with the learner's goal, and why the
   verdict follows.
5. **Process recommendation (D1).** From `references/process-types.md` (the five types:
   source-anchored companion · generated curriculum + capstone · project-first ·
   Socratic layer · hybrid spine), recommend a mix WITH justification; propose 2–3
   capstone candidates where a capstone fits; recommend the practice medium (D7).
   Learner approves or overrides.
6. **Environment & tooling advisor (D11).** Inventory first (hardware, OS, accounts,
   subscriptions), then budget ceiling (one-time vs monthly), then a researched options
   matrix (cost / setup complexity / learning value / risks) with a recommendation —
   the learner chooses. Writes ENVIRONMENT.md.
7. **Baseline pinning (D2).** Fix versions, editions, doc URLs → BASELINE.md.
8. **Roadmap generation (D12).** Spawn TWO roadmap-drafter agents in parallel with the
   same inputs (CLAUDE.md draft + prior-art verdict); roadmap-judge merges, justifying
   every inclusion, exclusion, and ordering choice. Write roadmap.md, all modules `draft`.
9. **HARD STOP (D8).** Present the roadmap for review. Iterate until the learner
   explicitly approves. Only then scaffold the remaining directory tree.

### 7.2 /plan-module N
Reads: topic CLAUDE.md, roadmap.md entry N (+ its prerequisites' README notes).
Refuses unless module N is `approved`. Produces the section plan in
`modules/NN-slug/README.md` (section id, goal, status), sized to D6 (one section =
one 1–2 h session). Learner reviews the plan before generating.

### 7.3 /generate N.K  (and /generate N = loop with an upfront cost warning)
Reads: conventions skill, topic CLAUDE.md, BASELINE.md, ENVIRONMENT.md, module README,
prerequisite sections' theory (listed in roadmap prerequisites), current workspace state.
Runs the generation pipeline (§8) → theory.md, practice.md, validation.md, quiz.md.
Sets section status `generated`. Refuses if module not `approved`/`planned`.

### 7.4 /validate N.K | /validate N
Reviews the learner's workspace increment against the section's (or whole module's)
validation criteria. Verdict + prioritized feedback; never rewrites the learner's code
unasked. For non-code topics: checks worked solutions step-by-step against validation.md.

### 7.5 /mentor [N.K | N | (empty)]
Interactive Socratic session scoped to a section, module, or general. Respects the
practice philosophy: guides derivation, doesn't hand out solution bodies; respects
ENVIRONMENT.md spend cap when proposing experiments.

### 7.6 /quiz N.K
Runs quiz.md interactively right after practice. Free-recall first, then probing
follow-ups ("why?", "when would this break?"). Misses and shaky answers are written
into `retention/deck/` as new cards (format §6.5) with `due: tomorrow`.

### 7.7 /review
Loads cards with `due <= today` (optionally filtered `/review module 3`). Interactive
free-recall session; Claude grades 0–5 with justification, probes *why* on shallow
answers, updates frontmatter scheduling in place, appends to retention/log.md.
Ends with a one-line forecast ("next: 6 cards due Thursday").

### 7.8 /checkpoint
At phase boundaries (or on demand): cumulative review mixing (a) due + sampled old
cards, (b) "explain it to me like I'm your junior" prompts, (c) ONE integration
challenge spanning earlier modules. Writes a gap report; gaps become cards and,
if severe, a proposed remedial section in the module README.

### 7.9 /retro
Interviews the learner about what's working/hurting in THIS topic. Proposes concrete
diffs: (a) topic-level changes (roadmap edits, process-mix changes, per-topic overrides
in project `.claude/`) applied directly with a changelog entry; (b) plugin-level
convention/command changes — NOT applied here; emitted as a ready-to-paste `/evolve`
brief instead.

### 7.10 /update-baseline
Re-verifies BASELINE.md rows via web search; updates versions + verified-on dates;
flags generated-but-unstudied sections whose content the changes may invalidate and
offers regeneration.

---

## 8. Generation pipeline (per section) — the quorum (D12)

```
section-drafter ──► draft (theory/practice/validation/quiz)
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

Critic contracts (each returns PASS or a numbered objection list):

- **critic-accuracy** — technical correctness; runs/execution-checks code samples where
  the environment allows; verifies claimed outputs of exercises are achievable.
- **critic-freshness** — checks every version-sensitive claim against BASELINE.md and
  current web sources; flags anything the model "remembered" that has since changed;
  requires "as of <date>" phrasing for volatile facts.
- **critic-depth** — enforces the Content depth standard (§9.1) INCLUDING the mandatory
  **Origins & evolution** block per concept; rejects tutorial-only or descriptive content.
- **critic-pedagogy** — enforces the Practice philosophy (§9.2): skeletons not solutions,
  max two worked examples, verifiable expected behavior, hint blocks; checks section
  fits the 1–2 h session budget (D6).

Roadmap-only full quorum: two independent roadmap-drafter runs + roadmap-judge merge
(structural diversity is worth paying for exactly once, at the artifact that shapes
everything downstream).

---

## 9. Conventions (ported from CQ into skills/conventions/ — quoted as normative)

### 9.1 Content depth standard
Goal: the learner can **advise** on each topic — understand tradeoffs and justify
decisions — not merely use it. Surface-level, descriptive, or tutorial-only content is
a failure. For EVERY concept, theory.md must: **Motivate it** (what problem, what goes
wrong without it) · **Origins & evolution** (NEW, structural: what world made it
necessary, what preceded it, what it displaced — the asyncio generators→yield from→
@coroutine→async def lineage is the canonical example) · **Explain the mechanism**
(under the hood, until behavior is predictable rather than magic) · **State the
tradeoffs** (costs, failure modes, competing alternatives) · **Give decision guidance**
(concrete heuristics, not "it depends") · **Show it concretely** (runnable examples,
then application to the capstone/workspace). Assume a strong senior learner who is a
genuine beginner in this section's subject; no length limit — sections exist so each
can be exhaustive in one pass.

### 9.2 Practice philosophy
practice.md is **challenge-based, not copy-paste ready**: setup steps as plain
instructions; **1–2 worked examples max** (full runnable code, labeled); every other
exercise is a challenge — starter skeleton (imports, typed signatures, calling code,
`# YOUR CODE HERE`, never the body), expected output/observable behavior for
self-verification, expandable hints block (2–4 bullets pointing into theory.md).
Same philosophy for capstone increments: acceptance criteria + skeleton, never a
filled-in feature. **Non-code adaptation:** worked example = fully worked problem;
challenge = problem statement + answer-shape + verification method (e.g. "differentiate
your solution and substitute back"; numeric check via sympy/plot); hints ladder from
"which technique class" to "first step".

### 9.3 Changelog discipline
CQ's "keep this file honest" block is copied into every scaffolded CLAUDE.md: any
change to roadmap/scope/stack MUST be recorded (date, what, why) BEFORE the work
diverges from the document. The plugin has the same rule via CHANGELOG.md + /evolve.

---

## 10. /evolve — changing the plugin itself, safely (D13)

Purpose: the learner says WHAT they want changed (sometimes only a goal, not a path);
`/evolve` handles the HOW — correct file formats, minimal context, decision support,
versioning, and changelog — so plugin quality doesn't decay through ad-hoc edits.

Run from the plugin repo checkout (the command verifies this; if run elsewhere, it
tells the learner where the checkout lives / how to clone it).

**Workflow:**

1. **Intent interview.** Classify the request: new command · behavior change ·
   convention change · new/changed critic · bugfix. If the goal is clear but the path
   is not, present 2–3 implementation options with tradeoffs (which files change,
   blast radius, token-cost impact, migration needs for existing topics) and a
   recommendation — the learner decides.
2. **Scope & load minimally.** Identify affected components; read ONLY those files
   (never the whole plugin). List them to the learner before editing.
3. **Impact check.** Grep for cross-references (e.g. a practice-philosophy change
   touches conventions, critic-pedagogy, generate, mentor, validate). Present the
   full blast radius; nothing outside the approved list gets edited.
4. **Draft the change** per `references/authoring-guide.md` (§11 knowledge baked in):
   correct SKILL.md frontmatter; "pushy" descriptions (state what it does AND the
   trigger contexts — Claude tends to under-trigger); SKILL.md bodies < 500 lines
   with overflow moved to references/ (progressive disclosure); side-effect commands
   keep `disable-model-invocation: true`.
5. **Review diff with the learner.** Show diffs, not summaries.
6. **Smoke test** on the bundled toy topic (§12.3) when the change affects generation,
   validation, or retention behavior.
7. **Version + changelog.** Semver bump in plugin.json (breaking format changes to
   topic-repo files = major; new commands/behavior = minor; fixes = patch) +
   CHANGELOG.md entry (date, what, why — mirror of the topic-level discipline).
   Major bumps MUST include a migration note for existing topic repos.
8. **Ship.** Commit; remind the learner to push and to update the plugin on other
   machines. If the change originated from a topic's /retro brief, note the topic.

**Anti-drift guarantee:** /retro (in topics) and /evolve (in the plugin) are the ONLY
sanctioned mutation paths; both end in changelog entries, so the system's own history
remains a first-class learning artifact.

---

## 11. Skill-authoring rules (bake into references/authoring-guide.md)

- Slash commands are implemented as **skills**; `commands/` is legacy.
- **Frontmatter description is the triggering mechanism**: include what it does AND
  explicit "use when the user says…" contexts; be a little pushy — Claude under-triggers.
- **Progressive disclosure:** SKILL.md < 500 lines; long material goes to
  `references/*.md` (with a table of contents if > 300 lines); reusable file skeletons
  go to `assets/`.
- One skill = one job; shared knowledge (conventions) is its own Claude-invocable
  knowledge skill rather than duplicated text.
- Agents live in `agents/*.md` with YAML frontmatter (name, description) + system
  instructions; keep them concise and maintainable — more lines ≠ better instructions.

---

## 12. Build plan (for the fresh Claude Code session)

### 12.1 Inputs to the session
- This SPEC.md
- The original ChoreQuest CLAUDE.md (source for §9 conventions — port faithfully,
  including exact wording of the depth standard and practice philosophy where possible)

### 12.2 Build order
1. Plugin scaffold: `.claude-plugin/plugin.json` (v0.1.0), README, CHANGELOG.
2. `skills/conventions/` (everything in §9) — it is a dependency of most other skills.
3. Agents (§8 contracts) — roadmap pair first, then section-drafter + 4 critics.
4. `skills/kickoff/` with `references/interview-script.md` (§7.1 phases as an explicit
   question script), `references/process-types.md` (the five types + selection
   heuristics), and `assets/` skeletons for CLAUDE.md / BASELINE.md / ENVIRONMENT.md /
   roadmap.md (§6 formats).
5. Generation chain: plan-module, generate (+ references/generation-pipeline.md),
   validate.
6. Retention: quiz, review (+ references/scheduling.md with the exact SM-2 variant
   from §6.5), checkpoint.
7. mentor, retro, update-baseline.
8. evolve (+ references/authoring-guide.md from §11).

### 12.3 Test plan (do not skip)
1. Install the plugin locally; verify all commands appear.
2. **Toy topic dry-run:** create `~/learn-topics/test-topic`, run /kickoff with a tiny,
   fast-moving, well-charted subject (suggested: "Python packaging with uv") — it
   exercises prior-art scan, baseline pinning, and freshness meaningfully in minutes.
3. Verify the D8 gate: /generate must refuse before roadmap approval.
4. Approve roadmap → /plan-module 1 → /generate 1.1 → confirm all four files exist,
   critics ran, objections surfaced if unresolved.
5. /quiz 1.1 → answer one question wrong on purpose → verify a card appears in
   retention/deck/ with due=tomorrow → /review → verify frontmatter scheduling updates.
6. /evolve dry-run: "add a --dry-run flag to /generate that prints the plan and token
   estimate without generating" → verify diff-review, version bump, changelog entry.
7. Keep the toy topic in the plugin repo under `examples/` as living documentation.

### 12.4 Definition of done (v0.1.0)
All commands implemented per §7 contracts · conventions ported verbatim-in-spirit from
CQ · test plan §12.3 green · README covers install, per-topic quickstart (kickoff →
approve roadmap → plan → generate → learn → validate → quiz → review), and the
/retro → /evolve improvement loop.

---

## 13. Handoff prompt

> Read SPEC.md and the attached ChoreQuest CLAUDE.md in full before writing anything.
> Build the `learn-anything` Claude Code plugin exactly per SPEC.md: follow the build
> order in §12.2, treat §7 command contracts and §8 agent contracts as normative, port
> the conventions in §9 faithfully from the ChoreQuest file, and implement commands as
> skills per §11. Ask me before deviating from the spec in any way, and record any
> agreed deviation in CHANGELOG.md. When the scaffold is complete, walk me through the
> test plan in §12.3 step by step, starting with installing the plugin and the toy-topic
> kickoff. Do not generate example learning content beyond the toy topic. Work
> incrementally and commit after each build-order step.
