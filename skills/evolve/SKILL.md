---
name: evolve
description: >-
  Safely change the learn-anything plugin ITSELF — new command, behavior change, convention
  change, new/changed critic, or bugfix. Runs a guarded workflow: intent interview →
  minimal-scope load → blast-radius impact check → draft per the authoring guide → diff
  review → smoke test on a throwaway scratch topic → semver bump + CHANGELOG entry → ship. Use when the
  user says "evolve the plugin", "add a command", "change how /author works", "the depth
  standard should…", pastes an /evolve brief from /retro, or runs /evolve. Runs from the
  plugin checkout only. Side-effect command — user-invoked only.
disable-model-invocation: true
argument-hint: "[what you want changed, or paste a /retro brief]"
---

# /evolve — change the plugin, safely

The learner says WHAT they want; you handle the HOW — correct formats, minimal context,
decision support, versioning, changelog — so plugin quality doesn't decay through ad-hoc
edits. Follow the authoring rules in
[`references/authoring-guide.md`](references/authoring-guide.md).

## Step 0 — verify you're in the plugin checkout
Confirm this directory is the `learn-anything` plugin (there is a `.claude-plugin/plugin.json`
with `"name": "learn-anything"`). **If not**, stop: tell the learner `/evolve` must run from
the plugin repo checkout, give the repo URL (`github.com/oswalth/learn-anything`), and how to
clone/cd there. Never edit the plugin from inside a topic repo.

## Step 1 — intent interview
Classify the request into one of: **new command · behavior change · convention change ·
new/changed critic · bugfix**. If a `/retro` brief was pasted, use it as the intent.
If the goal is clear but the *path* isn't, present **2–3 implementation options** with
tradeoffs each: which files change, blast radius, token-cost impact, and migration needs for
existing topic repos. Recommend one; the learner decides.

## Step 2 — scope & load minimally
Identify the affected components and read **only** those files (never the whole plugin).
**List them to the learner before editing.**

## Step 3 — impact check (blast radius)
Grep the plugin for cross-references to what you're changing. Conventions are referenced
widely — e.g. a practice-philosophy change touches `skills/conventions/`, `critic-pedagogy`,
`author`, `mentor`, and `check`. Present the **full blast radius**. Nothing outside the
approved list gets edited.

## Step 4 — draft the change
Per the authoring guide: correct `SKILL.md`/agent frontmatter; **"pushy" descriptions** (what
it does AND explicit trigger contexts — Claude under-triggers); `SKILL.md` bodies < 500 lines
with overflow pushed to `references/`; side-effect commands keep `disable-model-invocation:
true`; shared knowledge stays in `skills/conventions/` rather than being duplicated.

## Step 5 — review the diff
Show the learner **diffs, not summaries** (`git diff`). Iterate until they're satisfied.

## Step 6 — smoke test
When the change affects generation, validation, or retention behavior, exercise it against a
**throwaway scratch topic** the learner creates in a temp dir — e.g. `mktemp -d`, then run
`/kickoff` there with a tiny goal — never a bundled example (the plugin ships none, on
purpose: a pinned example would rot against the freshness doctrine). Dry-run the relevant
command against the scratch topic and report what you checked; the learner can delete the temp
dir afterward.

## Step 7 — version + changelog
- Bump the semver in `.claude-plugin/plugin.json`:
  - **major** — breaking format change to topic-repo files (existing topics need migration);
  - **minor** — new command or new behavior;
  - **patch** — a fix.
- Add a `CHANGELOG.md` entry: date · what · why. **Major bumps MUST include a migration note**
  for existing topic repos.

## Step 8 — ship
Commit, then tag the commit with an annotated git tag matching the new version:
`git tag -a v<version> -m "<one-line changelog summary>"`. Remind the learner to **push
both** — `git push && git push --tags` — and to update the plugin on their other machines
(`/plugin`). If the change came from a topic's `/retro` brief, note the originating topic in
the changelog entry.

**Anti-drift:** this workflow and topic `/retro` are the only sanctioned mutation paths; both
end in a changelog entry, keeping the system's own history a first-class artifact.
