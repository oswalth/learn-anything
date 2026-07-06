# Roadmap — {{TOPIC}}

{{N}} modules across {{M}} phases. This roadmap is the **gated deliverable**: no section is
planned or generated until the learner approves it. Each module carries a **module status**:

`draft` → `approved` → `planned` → `in-progress` → `done`

- `draft` → `approved`: learner approval in `/kickoff`.
- `approved` → `planned`: `/plan-module N`.
- `planned` → `in-progress`: the first `/author N.K` in that module.
- `in-progress` → `done`: `/check N` on a whole-module PASS (a **mastery** claim, with the
  learner's confirmation — validation owns it, not authoring).
- `/plan-module N` and `/author N.K` **refuse** to run on a module that is not yet
  `approved`.
- After approval, **any** change to this roadmap requires a `CLAUDE.md` changelog entry
  (keep-this-file-honest rule). Small edits: change directly. Larger restructuring — the
  goal/environment changed, or you want a refresh against the plugin's current conventions —
  run `/replan-roadmap`, which regenerates through the drafter+judge quorum while protecting
  modules already `approved`+ by default.

(**Section** statuses — `planned` → `generated` → `studied` — live in each module's
`modules/NN-slug/README.md`, not here.)

Each module lists: **Concept** (what it teaches) · **Skills** (what it covers) · **Capstone
increment** (what the workspace gains) · **Prerequisites** (module numbers to read when
generating) · **Source** (`generated`, or the prior-art unit it maps to).

---

## Phase {{0}} — {{phase goal}}

### Module {{01}} · {{Title}}
- **status:** draft
- **Concept:** {{the through-line this module teaches}}
- **Skills:** {{the specific capabilities/subtopics}}
- **Capstone increment:** {{what the workspace gains, or "none — standalone exercises"}}
- **Prerequisites:** {{none | 01, 02}}
- **Source:** {{generated | e.g. "MIT OCW 18.01 Unit 2" | "The Rust Book ch. 4–6"}}

### Module {{02}} · {{Title}}
- **status:** draft
- **Concept:** {{…}}
- **Skills:** {{…}}
- **Capstone increment:** {{…}}
- **Prerequisites:** {{…}}
- **Source:** {{…}}

## Phase {{1}} — {{phase goal}}

### Module {{03}} · {{Title}}
- **status:** draft
- **Concept:** {{…}}
- **Skills:** {{…}}
- **Capstone increment:** {{…}}
- **Prerequisites:** {{…}}
- **Source:** {{…}}

---

## Judge's rationale

{{The key merge decisions from roadmap-judge: notable inclusions/exclusions and the single
biggest ordering call, each with its reason. Preserved so the shape of the roadmap stays
explainable later.}}
