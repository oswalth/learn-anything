# Kickoff interview script

The exact questions to ask, phase by phase. **Ask one at a time**, wait for the answer,
write it into the growing `CLAUDE.md` (or `ENVIRONMENT.md` / `BASELINE.md`) draft, then ask
the next. Adapt wording to the learner and the topic; skip a question only if a previous
answer already fully covered it. Never dump the whole list at once.

Tone: you are interviewing a senior engineer. Be crisp, assume competence, probe for the
non-obvious. When they give a vague answer to a capability question, push once for
specificity ("what would you be able to *do* that you can't now?").

---

## Phase 1 — Goal

1. "What do you want to learn?" (the topic in one line)
2. "Why now? What triggered this — a project, a role change, a gap you keep hitting?"
3. "Finish this sentence three ways: *When I'm done, I'll be able to…* " — push each toward
   a **capability**: "advise on X", "build Y unaided", "solve Z-class problems", "pass
   exam W". These become the success criteria and every module must serve one of them.
4. "Is there a deadline or target pace? How many hours a week, roughly?"

→ Write: `CLAUDE.md` header (topic, learner, method TBD, pace, started = today, status =
"kickoff in progress") and the Goal & success criteria section.

## Phase 2 — Learner relative to this topic

5. "Where are you now with this specifically — never touched it, dabbled, or working
   knowledge with gaps?"
6. **(First-class question.)** "What do you already know deeply that we can *contrast*
   against to learn this faster? What's the closest adjacent thing in your head?" — e.g.
   Django for FastAPI; imperative concurrency for async; the numerical methods you know for
   analytic ODEs. Capture 2–5 concrete contrast pairs; content will lean on them.
7. "Have you tried to learn this before? Where did it stall, and why?"

→ Write: Current level, Adjacent-expertise contrasts (as a list of pairs), Prior attempts.

## Phase 3 — Topic properties

8. "Is there a canonical book, course, or doc set for this — and is it any good?"
9. "How fast does this field move? Is what you learn this year stale next year?"
10. "Is this more **skill-like** (you get better by doing) or **theory-like** (you get
    better by understanding), or both?"
11. "What does *verification* look like here — how do you know you got something right?
    (tests pass / cluster converges / a proof checks / sympy confirms / a plot matches)."

→ These answers feed the prior-art scan, process recommendation, and the shape of
`validation.md` later. Note them in `CLAUDE.md` under a short "Topic properties" note or
fold into the process-mix justification.

## Phase 4 — Prior-art scan (D10)

This is research, not just questions. **Use WebSearch/WebFetch.**

12. Search for established curricula, canonical books, structured paths, and cert tracks for
    the topic (CS50, MIT OCW, teachyourselfcs.com, roadmap.sh, official cert blueprints,
    the "the X book everyone recommends"). Fetch enough to judge coverage and currency.
13. Present the 2–4 strongest candidates to the learner with a one-line take on each.
14. Form a **verdict** and state it explicitly:
    - **adopt** — a prior-art curriculum is strong and well-matched; the roadmap maps onto
      its units. Theory switches to *companion mode*: pre-briefings, post-unit Socratic
      debriefs, the evolution lens, and currency patches. Practice stays generated,
      challenge-style.
    - **adapt** — a curriculum is a good spine but needs modules added/removed/reordered for
      this learner's goal.
    - **generate** — no adequate prior art; build from goal + topic properties.
15. Justify: what the prior art covers well, where it's stale or misaligned with *this*
    goal, and why the verdict follows.

→ Write: Prior-art verdict + justification (with the specific sources named).

## Phase 5 — Process recommendation (D1)

Read [`process-types.md`](process-types.md) first. Then:

16. Recommend a **process mix** (often a blend, e.g. "source-anchored companion for theory
    + project-first for practice") with a justification tied to the topic properties and
    prior-art verdict.
17. Where a capstone fits, propose **2–3 capstone candidates** — each a one-liner on what it
    would exercise. (Skip for pure-theory topics where a capstone doesn't fit; say so.)
18. Recommend the **practice medium** (D7): code repo / Jupyter notebooks / LaTeX-markdown
    problem sets / a lab environment / mix.
19. Ask the learner to approve or override the mix, the capstone (if any), and the medium.

→ Write: Process mix + justification; Capstone spec (chosen candidate + acceptance sketch),
or "no capstone — <why>".

## Phase 6 — Environment & tooling advisor (D11)

20. **Inventory:** "What are you working on — machine specs, OS? Any relevant accounts or
    subscriptions already (cloud provider, GitHub, a paid course, university access)?"
21. **Budget ceiling:** "What's your ceiling for this — one-time spend and monthly? Is $0
    the target, or is some spend fine if it buys real learning value?"
22. **Research + matrix:** Web-research the realistic setup options. Present an **options
    matrix**: option | one-time cost | monthly cost | setup complexity | learning value |
    risks/limits. Recommend one.
23. Learner chooses. Record the **spend cap** (a hard number) and a **teardown checklist**.
    For cloud topics, note that billing alarms + a teardown script are the first practice
    exercise.

→ Write: `ENVIRONMENT.md` (chosen option + full matrix + spend cap + teardown checklist).

## Phase 7 — Baseline pinning (D2)

24. Web-verify and pin the current versions/editions and authoritative doc URLs for the core
    components of the topic (language/runtime version, key libraries/tools, the canonical
    book edition, primary docs). Stamp each with today as the verified-on date.

→ Write: `BASELINE.md` table.

## Phase 8 — Roadmap generation (D12)

25. Spawn **two `roadmap-drafter` agents in parallel** (Agent tool, both in one message) with
    identical inputs: the `CLAUDE.md` draft, the prior-art verdict, and the **conventions
    text inlined** — read it yourself from `${CLAUDE_PLUGIN_ROOT}/skills/conventions/SKILL.md`
    (fallback `${CLAUDE_SKILL_DIR}/../conventions/SKILL.md`); the agents can't open
    plugin-internal files by relative path from the topic-repo cwd.
26. Run **`roadmap-judge`** once on the two drafts to merge into the final `roadmap.md`.
27. Write `roadmap.md` from `assets/roadmap.md`, **every module `status: draft`**.

## Phase 9 — HARD STOP + approval (D8)

28. Present the roadmap. Ask for changes. Iterate (regenerating `roadmap.md`) until the
    learner says, unambiguously, that they approve it.
29. On approval only: flip module statuses `draft → approved`, scaffold `modules/README.md`
    (the **module**-status board — module vocabulary `draft → approved → planned →
    in-progress → done`), `retention/deck/` + `retention/log.md`, `workspace/`; add a
    changelog entry to
    `CLAUDE.md` ("roadmap approved — N modules across M phases"); tell the learner to run
    `/plan-module 1`.
