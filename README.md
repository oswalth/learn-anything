# learn-anything

An AI-assisted self-learning system, packaged as a Claude Code plugin.

It is built for a **senior engineer who wants advise-level depth** — understanding *how*
things work and *why* they came to be, not just how to use them — with **challenge-based
practice** and **spaced-repetition retention**. Topics are unbounded: Go, FastAPI, AWS,
Kubernetes, networking — and non-code subjects like differential equations, too.

The plugin generalizes a proven, hand-built learning system ("ChoreQuest / FastAPI
Mastery"). Its conventions are treated as validated-by-usage defaults.

---

## Table of contents

- [The two-artifact model](#the-two-artifact-model)
- [Requirements](#requirements)
- [1. Install the plugin](#1-install-the-plugin)
- [2. Update the plugin](#2-update-the-plugin)
- [3. Manage the plugin (list, disable, uninstall)](#3-manage-the-plugin-list-disable-uninstall)
- [4. Set up a learning repo (per topic)](#4-set-up-a-learning-repo-per-topic)
- [5. The daily loop + full command reference](#5-the-daily-loop--full-command-reference)
- [6. The improvement loop: /retro → /evolve](#6-the-improvement-loop-retro--evolve)
- [7. Restore a topic on another machine](#7-restore-a-topic-on-another-machine)
- [What makes the content different](#what-makes-the-content-different)
- [Troubleshooting](#troubleshooting)
- [Repository layout](#repository-layout)

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
the plugin's marketplace/repo and minimum version so any machine can restore the setup
(see [§7](#7-restore-a-topic-on-another-machine)).

A topic repo MAY still shadow plugin definitions with project-level `.claude/skills/` or
`.claude/agents/` files of the same name (project scope wins) — this is how `/retro`
proposes per-topic overrides. Promoting an override into the plugin goes through `/evolve`.

---

## Requirements

- **Claude Code** with the plugin system (marketplaces). If `/plugin` isn't recognized,
  update Claude Code first.
- **git** available on your PATH (Claude Code clones the marketplace repo).
- For the GitHub install path: access to `github.com/oswalth/learn-anything` via your
  normal git auth (SSH key or `gh auth`). The repo may be private; manual installation
  works as long as your git can read it.

---

## 1. Install the plugin

> **Key idea:** `/plugin install` does **not** take a raw repo URL. Plugins are installed
> **from a marketplace**. This repo is *both* the marketplace and the plugin, so you first
> **add the marketplace**, then **install the plugin from it**. Because the marketplace and
> the plugin are both named `learn-anything`, the install target is
> `learn-anything@learn-anything` (that's `plugin@marketplace`).

### Option A — from GitHub (normal use)

```text
/plugin marketplace add oswalth/learn-anything
/plugin install learn-anything@learn-anything
```

Note the shorthand is `owner/repo` — **not** `github.com/oswalth/...` and **not** a full
URL. (Non-GitHub hosts need the full `https://host/owner/repo.git` form.)

### Option B — from a local clone (development / offline)

```text
/plugin marketplace add /absolute/path/to/learn-anything
/plugin install learn-anything@learn-anything
```

You can point at the directory (it must contain `.claude-plugin/marketplace.json`) or at
the `marketplace.json` file directly.

### Scope

`/plugin install` installs at **user scope** by default — the plugin is then available in
**every** directory/project you open, which is what you want (each topic lives in its own
directory). The interactive installer lets you pick a scope; user is the default.

### Verify it worked

```text
/plugin list           # learn-anything should show as installed + enabled
/help                  # scroll to the plugin's commands
```

If commands don't appear immediately, run `/reload-plugins`. You should see, namespaced as
`learn-anything:<name>` (and usually as bare `/<name>`):

`kickoff` · `plan-module` · `author` · `check` · `mentor` · `quiz` · `recall` ·
`checkpoint` · `retro` · `update-baseline` · `evolve`

### CLI equivalents (scripting / dotfiles)

Every step has a non-interactive CLI form:

```bash
claude plugin marketplace add oswalth/learn-anything
claude plugin install learn-anything@learn-anything          # --scope user (default) | project | local
claude plugin list
```

---

## 2. Update the plugin

**Mental model — two independent things:**

- the **marketplace cache** (a local copy of `marketplace.json` listing available versions), and
- the **installed plugin** (the version you actually run).

Refreshing the marketplace does **not** by itself upgrade the installed plugin. Custom
(non-Anthropic) marketplaces have **auto-update disabled by default**, so updating is a
deliberate two-step:

```text
/plugin marketplace update learn-anything    # refresh the catalog from the repo
/plugin install learn-anything@learn-anything  # reinstall to pull the new version
/reload-plugins                              # apply in the current session (no restart needed)
```

(Under the hood, `/plugin install` over an existing install upgrades it. If you prefer an
explicit clean swap: `/plugin uninstall learn-anything@learn-anything` then install again.)

**Optional — enable auto-update** so new pushes are picked up at Claude Code startup:
open `/plugin` → **Marketplaces** tab → select `learn-anything` → *Enable auto-update*.
When an update lands you'll be prompted to `/reload-plugins`.

**Because all machinery lives in the plugin, updating it updates every topic at once** — no
per-topic sync. The plugin's own change history is in [`CHANGELOG.md`](CHANGELOG.md).

---

## 3. Manage the plugin (list, disable, uninstall)

```text
/plugin                                   # interactive manager (Discover/Installed/Marketplaces/Errors)
/plugin list                              # installed plugins (--enabled / --disabled to filter)
/plugin marketplace list                  # added marketplaces

/plugin disable learn-anything@learn-anything   # keep installed, turn off
/plugin enable  learn-anything@learn-anything   # turn back on
/reload-plugins                                  # apply enable/disable now

/plugin uninstall learn-anything@learn-anything          # remove the plugin
/plugin marketplace remove learn-anything                # remove the marketplace (also removes its plugins)
```

Installed plugins are cached under `~/.claude/plugins/`; added marketplaces are tracked in
`~/.claude/plugins/known_marketplaces.json`. You normally never touch these by hand.

**Team / multi-machine auto-install** — declare the marketplace and plugin in a project's
`.claude/settings.json` so collaborators are prompted to install on trust:

```json
{
  "extraKnownMarketplaces": {
    "learn-anything": {
      "source": { "source": "github", "repo": "oswalth/learn-anything" },
      "autoUpdate": true
    }
  },
  "enabledPlugins": {
    "learn-anything@learn-anything": true
  }
}
```

---

## 4. Set up a learning repo (per topic)

Each topic gets its **own empty directory**. The plugin (installed at user scope) is visible
there automatically — you do **not** clone or copy anything into the topic repo.

```bash
mkdir -p ~/learn-topics/golang
cd ~/learn-topics/golang
claude          # start Claude Code in this directory
```

Then, inside Claude Code:

```text
/kickoff
```

`/kickoff` is an **interview wizard**. It asks one question at a time and, as you answer,
writes the topic's source-of-truth files. It walks through:

1. **Goal** — what you want to learn and the capabilities you want ("I can advise on X",
   "build Y unaided", "solve Z-class problems"), plus pace/deadline.
2. **Your level & adjacent expertise** — what you already know deeply that we can teach the
   new material *against* (e.g. Django ↔ FastAPI). A first-class input.
3. **Topic properties** — is there a canonical source? how fast does it move? what does
   *verification* look like?
4. **Prior-art scan** — it web-researches established curricula/books/paths (CS50, MIT OCW,
   roadmap.sh, cert blueprints…) and gives an **adopt / adapt / generate** verdict.
5. **Process recommendation** — a learning-process mix + a practice medium, with 2–3
   capstone candidates where a capstone fits. You approve or override.
6. **Environment & spend** — inventories your setup, sets a **spend cap**, writes
   `ENVIRONMENT.md` (for cloud topics, billing alarms + teardown are the first exercise).
7. **Baseline pinning** — pins current versions/editions/doc URLs into `BASELINE.md`.
8. **Roadmap** — two roadmap drafters run in parallel and a judge merges them into
   `roadmap.md` (phases → modules), every module marked `draft`.

Then it **STOPS and waits** — this is the hard gate:

```text
# Iterate conversationally until you're happy:
#   "merge modules 4 and 5"
#   "move observability earlier"
#   "drop the gRPC module, add one on profiling"
# Then, explicitly:
"the roadmap is approved"
```

**No section content is generated until you approve the roadmap.** On approval, `/kickoff`
flips the modules to `approved`, scaffolds the rest of the tree
(`modules/`, `retention/`, `workspace/`), and points you at `/plan-module 1`.

After kickoff, a topic repo looks like:

```text
~/learn-topics/golang/
├── CLAUDE.md        # topic identity: goal, level, contrasts, process mix, plugin dependency, changelog
├── BASELINE.md      # pinned versions / editions / doc URLs (+ verified-on dates)
├── ENVIRONMENT.md   # chosen setup, options matrix, spend cap, teardown checklist
├── roadmap.md       # phases → modules, each with a status (the approval gate lives here)
├── modules/         # per-module READMEs (section plans + your notes) and generated sections
├── retention/       # spaced-repetition deck (one card per file) + review log
└── workspace/       # where YOU build/solve — the capstone, notebooks, problem sets
```

---

## 5. The daily loop + full command reference

Once a topic's roadmap is approved, the rhythm per module is:

```text
/plan-module 1     →  break approved module 1 into 1–2 h sections (cheap, no content yet)
/author 1.1        →  generate section 1.1 (README + theory + practice + validation + quiz), via the critic quorum
                      … start at README.md, then theory.md, then practice.md in workspace/ …
/check 1.1         →  review your work against the section's criteria (never rewrites your code)
/quiz 1.1          →  free-recall quiz; misses become spaced-repetition cards
/recall            →  daily: review the cards that are due
# repeat /author 1.K … for each section, then:
/check 1           →  whole-module validation; on a pass, marks module 1 done
/checkpoint        →  at phase boundaries: cumulative review + one integration challenge
```

**All commands** (`N` = module number, `N.K` = section K of module N):

| Command | What it does | Invoke |
|---|---|---|
| `/kickoff` | Interview → gated roadmap → scaffold a topic repo | you |
| `/plan-module N` | Break approved module N into 1–2 h sections | you |
| `/author N.K` \| `/author N` | Generate a section (or a whole module, with a cost warning) through the drafter + 4-critic quorum | you |
| `/check N.K` \| `/check N` | Review your workspace against the section's / module's criteria; can mark section `studied` / module `done` on a pass | you or Claude |
| `/mentor [N.K \| N]` | Socratic session; guides you to derive, never hands over solutions | you or Claude |
| `/quiz N.K` | Post-practice free-recall quiz; misses become cards | you or Claude |
| `/recall [module N]` | Spaced-repetition session over due cards (SM-2) | you or Claude |
| `/checkpoint` | Cumulative, integrative review at a phase boundary | you or Claude |
| `/retro` | Retrospective on this topic; applies topic fixes, emits `/evolve` briefs for plugin fixes | you |
| `/update-baseline` | Re-verify pinned versions via web search; flag stale generated sections | you |
| `/evolve` | Safely change the **plugin itself** (run from the plugin checkout) | you |

*"Invoke: you"* = the command has side effects or costs tokens, so Claude won't auto-run it —
type it yourself. *"you or Claude"* = safe, interactive commands Claude may also trigger when
you say things like "quiz me" or "review my flashcards".

---

## 6. The improvement loop: /retro → /evolve

The system is designed to improve itself without quality decay:

- **`/retro`** (run inside a topic) interviews you about what's working and what hurts.
  Topic-level fixes (roadmap edits, process changes, per-topic overrides) are applied
  directly with a changelog entry. Plugin-level ideas are emitted as a ready-to-paste
  `/evolve` brief — never applied from inside a topic.
- **`/evolve`** (run inside this plugin's checkout) is a guarded workflow for changing the
  plugin itself: intent interview, minimal-scope loading, blast-radius impact check,
  diff review, a smoke test against a throwaway scratch topic, semver bump, and a
  `CHANGELOG.md` entry. After shipping an `/evolve` change, push the plugin repo and
  [update](#2-update-the-plugin) it on your other machines.

`/retro` and `/evolve` are the *only* sanctioned mutation paths. Both end in changelog
entries, so the system's own history stays a first-class learning artifact.

---

## 7. Restore a topic on another machine

A topic repo is content-only and depends on the plugin. To pick a topic up elsewhere:

1. Install the plugin (see [§1](#1-install-the-plugin)) — the topic's `CLAUDE.md` records
   the marketplace/repo and the **minimum plugin version** it was created against.
   `/author` and `/plan-module` warn you if the installed plugin is older than that minimum.
2. Get the topic directory onto the machine (it's just files — clone/sync however you like).
3. `cd` into it, start Claude Code, and continue with the [daily loop](#5-the-daily-loop--full-command-reference).

No per-topic setup beyond having the plugin installed.

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

## Troubleshooting

**`Marketplace "github.com/oswalth/learn-anything" not found`** — you passed a URL to
`/plugin install`. Add the marketplace first, then install by name:
`/plugin marketplace add oswalth/learn-anything` → `/plugin install learn-anything@learn-anything`.
Use the `owner/repo` shorthand, not a full `github.com/...` URL.

**`/plugin install learn-anything` fails** — the install target needs the marketplace:
it's `learn-anything@learn-anything` (`plugin@marketplace`), not just `learn-anything`.

**Commands don't show up in `/help`** — run `/reload-plugins`; confirm with `/plugin list`
that the plugin is installed **and enabled**.

**A command name is shadowed** (another plugin/built-in also defines it) — use the
namespaced form, e.g. `/learn-anything:recall`.

**`/plugin marketplace add oswalth/learn-anything` fails** — the repo must be reachable by
your git auth. Verify the branch with the `marketplace.json` is pushed
(`git ls-remote <remote>`), and that you can clone it manually.

**Installed an update but still see old behavior** — reinstalling refreshes files, but the
session may be stale: run `/reload-plugins` (or restart Claude Code).

**`/evolve` says it must run from the plugin checkout** — `/evolve` mutates the plugin, so
run it from a git clone of this repo, not from a topic directory.

---

## Repository layout

```text
learn-anything/
├── .claude-plugin/
│   ├── plugin.json        # the plugin manifest: name, version (semver), description
│   └── marketplace.json   # the marketplace manifest that makes it installable
├── CLAUDE.md              # how to maintain THIS repo: versioning, changelog, skill/agent rules
├── CHANGELOG.md           # every /evolve change: date, what, why
├── README.md              # this file
├── SPEC.md                # full design, decision record, command contracts (frozen + amended)
├── skills/                # slash commands + the conventions knowledge skill
└── agents/                # roadmap pair, section-drafter, 4 critics
```

The plugin ships **no bundled example topic** — it is a repeatable pattern, and
`/kickoff` scaffolds a real topic repo on demand. The file layout a topic repo gets is
documented by the skeletons in `skills/kickoff/assets/`.

See [`SPEC.md`](SPEC.md) for the full design, decision record, and command contracts.
