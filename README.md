# learn-anything

An AI-assisted self-learning system, packaged as a Claude Code plugin.

It is built for a **senior engineer who wants advise-level depth** — understanding *how*
things work and *why* they came to be, not just how to use them — with **challenge-based
practice** and **spaced-repetition retention**. Topics are unbounded: Go, FastAPI, AWS,
Kubernetes, networking — and non-code subjects like differential equations, too.

The plugin generalizes a proven, hand-built learning system ("ChoreQuest / FastAPI
Mastery"). Its conventions are treated as validated-by-usage defaults.

---

## The two-artifact model

1. **This plugin** holds *all the machinery* — skills (slash commands), subagents, and
   shared conventions. One repo, one source of truth. Updating the plugin updates every
   topic at once.
2. **Topic repos** are thin, *content-only* directories scaffolded by `/kickoff`. They
   contain zero machinery. A topic repo **+** the plugin = a complete learning system.

Why the split? The naive alternative — a template repo cloned per topic — makes every
template improvement require N manual syncs. Here, machinery lives once in the plugin
(edit once, applies everywhere), and topic repos hold only per-topic content, which never
needs syncing. The trade-off: a topic repo is not self-contained. Its `CLAUDE.md` records
the plugin repo URL and minimum version so any machine can restore the setup.

A topic repo MAY still shadow plugin definitions with project-level `.claude/skills/` or
`.claude/agents/` files of the same name (project scope wins) — this is how `/retro`
proposes per-topic overrides. Promoting an override into the plugin goes through `/evolve`.

---

## Install

Install the plugin at **user scope** so every topic directory sees it:

```
/plugin install github.com/oswalth/learn-anything
```

Or, from a local checkout:

```
/plugin install /path/to/learn-anything
```

Verify the commands appear:

```
/help
```

You should see `kickoff`, `plan-module`, `author`, `check`, `mentor`, `quiz`,
`recall`, `checkpoint`, `retro`, `update-baseline`, and `evolve` (namespaced as
`learn-anything:<name>`).

---

## Per-topic quickstart

Start each new topic in its own empty directory (e.g. `~/learn-topics/golang`):

1. **`/kickoff`** — an interview wizard. It nails down your goal and current level,
   scans prior art (CS50 / MIT OCW / roadmap.sh / books / cert paths), recommends a
   learning process and practice medium, advises on environment & spend, pins a version
   baseline, and drafts a roadmap. **It stops at the roadmap and waits for your approval.**
2. **Approve the roadmap.** Iterate conversationally ("merge modules 4–5", "move
   observability earlier") until you say it's approved. No content is generated before this.
3. **`/plan-module N`** — breaks approved module N into 1–2 h sections (cheap).
4. **`/author N.K`** — generates one section: `theory.md`, `practice.md`,
   `validation.md`, `quiz.md`, run through a drafter + critic quorum.
5. **Learn & build** — read `theory.md`, then work `practice.md` in your `workspace/`.
6. **`/check N.K`** — reviews your work against the section's criteria (never rewrites it).
7. **`/quiz N.K`** — free-recall quiz right after practice; misses become retention cards.
8. **`/recall`** — daily spaced-repetition session over due cards.
9. **`/checkpoint`** — at phase boundaries: cumulative review + one integration challenge.

Supporting commands: **`/mentor`** (Socratic session, any scope) and
**`/update-baseline`** (re-verify pinned versions via web search).

---

## The improvement loop: `/retro` → `/evolve`

The system is designed to improve itself without quality decay:

- **`/retro`** (run inside a topic) interviews you about what's working and what hurts.
  Topic-level fixes (roadmap edits, process changes, per-topic overrides) are applied
  directly with a changelog entry. Plugin-level ideas are emitted as a ready-to-paste
  `/evolve` brief — never applied from inside a topic.
- **`/evolve`** (run inside this plugin's checkout) is a guarded workflow for changing the
  plugin itself: intent interview, minimal-scope loading, blast-radius impact check,
  diff review, a smoke test against a throwaway scratch topic, semver bump, and a
  `CHANGELOG.md` entry.

`/retro` and `/evolve` are the *only* sanctioned mutation paths. Both end in changelog
entries, so the system's own history stays a first-class learning artifact.

---

## What makes the content different

Every generated section is held to two ported conventions (see
[`skills/conventions/SKILL.md`](skills/conventions/SKILL.md)):

- **Content depth standard** — for *every* concept: motivate it · **origins & evolution**
  (what world made it necessary, what it displaced) · mechanism (under the hood) ·
  tradeoffs · decision guidance · show it concretely. Tutorial-only content is a failure.
- **Practice philosophy** — challenge-based, not copy-paste: at most two worked examples,
  then skeletons with `# YOUR CODE HERE`, verifiable expected behavior, and hint ladders.

Freshness is enforced by a pinned `BASELINE.md` plus a mandatory web-research pass by a
freshness critic at generation time.

---

## Repository layout

```
learn-anything/
├── .claude-plugin/plugin.json   # name, version (semver), description
├── CHANGELOG.md                 # every /evolve change: date, what, why
├── README.md                    # this file
├── skills/                      # slash commands + the conventions knowledge skill
└── agents/                      # roadmap pair, section-drafter, 4 critics
```

The plugin ships **no bundled example topic** — it is a repeatable pattern, and
`/kickoff` scaffolds a real topic repo on demand. The file layout a topic repo gets is
documented by the skeletons in `skills/kickoff/assets/`.

See [`SPEC.md`](SPEC.md) for the full design, decision record, and command contracts.
