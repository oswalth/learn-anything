---
name: evolve
description: >-
  Change the learn-anything plugin ITSELF — new command, behavior change, convention change,
  or bugfix — in one scoped edit pass followed by one cheap structural check. Keeps Claude,
  Codex, and Copilot packaging synchronized, bumps semver, and updates CHANGELOG.md. Use when the
  user says "evolve the plugin", "add a command", "change how /author works", "the depth
  standard should…", pastes an /evolve brief from /retro, or runs /evolve. Runs from the
  plugin checkout only. Side-effect command — user-invoked only.
disable-model-invocation: false
argument-hint: "[what you want changed, or paste a /retro brief]"
---

# /evolve — change the plugin in one pass

The learner says WHAT they want; you handle the HOW — correct formats, minimal context,
decision support, versioning, changelog — so plugin quality doesn't decay through ad-hoc
edits. Follow the authoring rules in
[`references/authoring-guide.md`](references/authoring-guide.md).

## Step 0 — verify you're in the plugin checkout
Confirm this directory is the `learn-anything` plugin (there is a `.claude-plugin/plugin.json`
with `"name": "learn-anything"`). **If not**, stop: tell the learner `/evolve` must run from
the plugin repo checkout, give the repo URL (`github.com/oswalth/learn-anything`), and how to
clone/cd there. Never edit the plugin from inside a topic repo.

## Step 1 — resolve intent
If a `/retro` brief was pasted, use it as the intent. Ask a question only when the requested
outcome is ambiguous or requires a compatibility decision; otherwise proceed.

## Step 2 — scope & load minimally
Identify the affected components and read **only** those files (never the whole plugin).

## Step 3 — impact check (blast radius)
Search once for cross-references to renamed, removed, or shared behavior. Include direct
consumers in the same edit; do not run a broad review cycle.

## Step 4 — draft the change
Per the authoring guide: correct `SKILL.md`/agent frontmatter; **"pushy" descriptions** (what
it does AND explicit trigger contexts); `SKILL.md` bodies < 500 lines with overflow pushed to
`references/`; side-effect commands keep **User-invoked only** language plus explicit
confirmation gates; shared knowledge stays in `skills/conventions/` rather than being
duplicated.

## Step 5 — one cheap validation
Run exactly one focused structural check appropriate to the change: parse changed JSON/TOML,
check changed frontmatter, or search for stale references. Do not spawn reviewers, iterate on
quality, or create a scratch topic. Fix only a concrete failure from this check.

## Step 6 — synchronize version + changelog
- Bump the same semver in `plugin.json`, `.claude-plugin/plugin.json`,
  `.claude-plugin/marketplace.json`, `.codex-plugin/plugin.json`, and
  `.github/plugin/marketplace.json`:
  - **major** — breaking command, packaging, or topic-repo format change;
  - **minor** — new command or new behavior;
  - **patch** — a fix.
- Add a `CHANGELOG.md` entry: date · what · why. **Major bumps MUST include a migration note**
  for existing topic repos.

## Step 7 — report
Summarize the change, the one validation performed, and any migration note. Commit, tag, and
push only when the learner explicitly asks. If the change came from a topic's `/retro` brief,
note the originating topic in the changelog entry.

**Anti-drift:** this workflow and topic `/retro` are the only sanctioned mutation paths; both
end in a changelog entry, keeping the system's own history a first-class artifact.
