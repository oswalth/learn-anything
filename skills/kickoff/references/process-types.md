# Process types + selection heuristics

The five learning-process archetypes the kickoff wizard recommends from (Phase 5). Most
real topics use a **mix** — commonly one archetype for theory and another for practice.
Recommend the mix, justify it against the topic properties and the prior-art verdict, and
let the learner override.

---

## The five types

### 1. Source-anchored companion
The learner works through an external canonical source (book/course/curriculum); the plugin
**wraps** it rather than replacing it. Theory becomes: a **pre-briefing** before each unit
(what to watch for, how it connects), a **post-unit Socratic debrief**, an **evolution
lens** the source usually omits (origins & what it displaced), and **currency patches**
(what changed since the source was written). Practice is generated challenge-style.

- **Best when:** a strong, well-matched canonical source exists (prior-art verdict = adopt).
- **Gives:** the source's structure + the plugin's depth/currency/retention on top.
- **Costs:** learner must actually have/consume the source; pacing is coupled to it.

### 2. Generated curriculum + capstone
The plugin generates the whole roadmap from scratch and the learner builds **one capstone**
that grows across modules (the ChoreQuest model). Theory and practice are both generated.

- **Best when:** no adequate prior art (verdict = generate), and the topic supports a single
  cohesive project that exercises most of it.
- **Gives:** maximal fit to the learner's exact goal; a portfolio artifact.
- **Costs:** most generation tokens; the roadmap quality gate matters most here.

### 3. Project-first
Start from a concrete thing to build/achieve; pull in theory **on demand** as the project
demands it, rather than front-loading. The roadmap is organized by project milestones, not
by concept taxonomy.

- **Best when:** skill-like topics where doing drives understanding; the learner is
  impatient with upfront theory; the goal is "build Y unaided".
- **Gives:** high motivation, immediate relevance.
- **Costs:** risk of gaps in foundations; needs checkpoints to catch what the project didn't
  force you to learn.

### 4. Socratic layer
Little generated exposition; the plugin mostly **asks** — guiding the learner to derive and
articulate understanding, surfacing gaps through questioning. Sits on top of whatever source
or project supplies the raw material.

- **Best when:** the learner already has partial/rusty knowledge to activate, or the topic
  is theory-like and understanding-limited rather than information-limited.
- **Gives:** deep retention, exposes illusions of understanding.
- **Costs:** slow for genuinely new material; frustrating if there's no base to build on.

### 5. Hybrid spine
An external curriculum is the **spine**, but with generated modules **added** where it's
thin and modules **dropped** where it's off-goal. Between adopt and generate.

- **Best when:** prior-art verdict = adapt — a curriculum is good but not a perfect fit.
- **Gives:** proven structure without inheriting its gaps or bloat.
- **Costs:** requires clear justification for every deviation (record it).

---

## Selection heuristics

Map the Phase 3–4 answers to a recommendation:

| Signal | Leans toward |
|---|---|
| Strong, current canonical source; verdict = adopt | **Source-anchored companion** (theory) |
| No adequate prior art; verdict = generate | **Generated curriculum + capstone** |
| Curriculum good-but-imperfect; verdict = adapt | **Hybrid spine** |
| Skill-like; goal = "build Y unaided" | **Project-first** (practice) |
| Theory-like; understanding-limited; some prior base | **Socratic layer** |
| Fast-moving field | favor companion/generate with live source checks during generation |
| Slow, settled field with a classic text | favor source-anchored companion |
| Verification = tests/cluster/build | capstone or project-first (practice medium = code/lab) |
| Verification = proof/derivation/numeric-check | problem sets (Jupyter / LaTeX-markdown), Socratic layer |

**Typical mixes**
- *Fast-moving framework, no perfect book:* generated curriculum + capstone, freshness-heavy.
- *Classic CS subject with a canonical course:* source-anchored companion (theory) +
  project-first or problem sets (practice).
- *Math/theory topic:* Socratic layer + problem sets, adopt/adapt a standard syllabus.
- *Cloud/infra cert:* hybrid spine on the cert blueprint + project-first labs, tight spend cap.

Always state the mix as "X for theory, Y for practice" when they differ, and tie the choice
to the specific answers the learner gave.
