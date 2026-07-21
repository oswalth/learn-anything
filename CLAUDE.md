# CLAUDE.md — maintaining the `learn-anything` plugin repo

This file is for whoever (human or Claude) is working **in this checkout**. It is the
plugin's own operating manual — the equivalent of the `CLAUDE.md` `/kickoff` writes into a
*topic* repo, but for the plugin itself. **Do not confuse the two:** this file governs
machinery (skills/agents/conventions); the *template* at
[`skills/kickoff/assets/CLAUDE.md`](skills/kickoff/assets/CLAUDE.md) is what gets scaffolded
into topic repos and carries **only topic-specific facts** — never plugin conventions text.

If you are unsure whether you're in the right place: this directory has
`.claude-plugin/plugin.json` with `"name": "learn-anything"`. If it doesn't, you're in a
topic repo or somewhere else — the rules below don't apply there.

## The one hard rule

**Every change to this repo goes through [`/evolve`](skills/evolve/SKILL.md).** `/evolve`
(in the plugin) and `/retro` (inside a topic,
which emits an `/evolve` brief back to here) are the **only** sanctioned mutation paths —
see [`README.md`](README.md) and
[SPEC.md §10](SPEC.md#10-evolve--changing-the-plugin-itself-safely-d13). If you're about to
edit plugin behavior without following `/evolve`'s scope → edit → one-check → synchronize
steps, stop and run `/evolve` instead
(or apply its lean discipline manually: read only affected files, search direct references,
edit once, run one structural check, then update every manifest and the changelog).

**No exception for "just docs."** A README wording fix still gets a version bump and a
changelog line (see 0.5.2 / 0.1.1 in `CHANGELOG.md` for precedent) — the point is that the
history stays complete, not that every entry is dramatic.

## Every change ships with a version bump + changelog entry

Bump the same `version` in [`plugin.json`](plugin.json),
[`.claude-plugin/plugin.json`](.claude-plugin/plugin.json),
[`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json),
[`.codex-plugin/plugin.json`](.codex-plugin/plugin.json), and
[`.github/plugin/marketplace.json`](.github/plugin/marketplace.json), then add an entry to
[`CHANGELOG.md`](CHANGELOG.md) (newest on top) with **date · what · why**:

- **patch** — a fix, or a doc/wording change with no interface or behavior change.
- **minor** — a new command, a new agent, or new/changed behavior.
- **major** — a breaking command, packaging, or topic-repo format change. **Must** include a
  migration note in the changelog entry.

There is no path that skips this. If you can't articulate the bump size and the changelog
line, the change isn't ready to ship.

## README.md must move in lockstep

`README.md` is the only doc a new user reads before installing. Any of these **must** be
updated in the *same* change that causes them, not deferred:

- Renaming, adding, or removing a command/skill → the command reference table (§5), the
  `/help` command list (§1), the repository-layout tree, and any inline mention elsewhere.
- Changing install/update/uninstall mechanics → §1–3 and the Troubleshooting section.
- Changing the topic-repo scaffold shape → §4's tree and file list.

Before shipping, grep for stale command names — old names should only ever appear inside
`CHANGELOG.md` (history) or `SPEC.md`'s frozen decision text, never as a live instruction
elsewhere:

```bash
grep -rn "/generate\b\|/validate\b\|/review\b" --include="*.md" . | grep -v CHANGELOG.md
```

(`SPEC.md` legitimately still says `/generate`/`/validate`/`/review` — it's the frozen
original design, not live instructions; see the amendment banner at its top.)

## Skill & agent structural rules

The full authoring rules live in
[`skills/evolve/references/authoring-guide.md`](skills/evolve/references/authoring-guide.md)
— read it before writing or editing a `SKILL.md` or `agents/*.md`. Do not duplicate it here;
these are the load-bearing points to keep in your head:

- **Frontmatter `description` is the trigger mechanism** — Claude under-triggers, so state
  what the command does *and* explicit "use when the user says…" phrasings.
- **Codex-compatible trigger discipline:** Codex validation rejects the old Claude-only
  `disable-model-invocation: true` value, so side-effecting / costly / mutating skills use
  `disable-model-invocation: false` and carry **User-invoked only** / **Side-effect command**
  language in their descriptions and confirmation gates in their bodies. Cheap /
  interactive / read-mostly skills (`mentor`, `quiz`, `recall`, `check`, `checkpoint`, and
  the `conventions` knowledge skill) remain naturally invocable.
- **Progressive disclosure** — `SKILL.md` bodies `< 500` lines; push detail into
  `references/*.md`; reusable file skeletons into `assets/`.
- **One skill = one job.** Shared knowledge (the depth standard, practice philosophy,
  session sizing, changelog discipline) lives once in
  [`skills/conventions/SKILL.md`](skills/conventions/SKILL.md) and is *referenced*, never
  copied, by every drafter or command that needs it.
- **Skills vs. agents load plugin-internal files differently** — a skill follows relative
  Markdown links to siblings; an agent can't open plugin files by path at all, so the
  orchestrating skill must inline the text into its prompt. See the authoring guide's
  "Locating plugin-internal files" section before wiring up anything new that needs
  `conventions/`.

## Blast-radius discipline

Before editing shared conventions or agent contracts, search once for direct consumers and
include them in the same scoped change. Do not add review loops or scratch-topic validation;
`/evolve` performs one structural check after one edit pass.

## `SPEC.md` is a frozen decision record, not live docs

`SPEC.md` is the original approved design (2026-07-05). It carries an amendment banner:
where it disagrees with the code + `CHANGELOG.md`, **the code and changelog win**. Don't
"fix" `SPEC.md` to match current behavior — its value is as a dated record of the original
decisions (D1–D14) and their justifications. If a decision is later reversed, record that in
`CHANGELOG.md`, not by silently editing `SPEC.md`'s decision table.

## No bundled example topic

The plugin ships **zero** example/toy topics (removed deliberately in `0.1.1` — a
version-pinned baseline would never be re-verified, which contradicts the freshness doctrine
the plugin exists to enforce). Do not reintroduce an `examples/` or
`docs/chorequest-*` directory.

## Quick map

```text
learn-anything/
├── plugin.json                  # Copilot CLI manifest
├── .github/plugin/marketplace.json # Copilot marketplace
├── .github/agents/              # Copilot-native drafter roles
├── .claude-plugin/plugin.json   # name, semver version — bump every change
├── .claude-plugin/marketplace.json  # mirrors the same version — keep in sync
├── CLAUDE.md                    # this file — plugin-repo maintenance rules
├── README.md                    # user-facing install + quickstart + command reference
├── SPEC.md                      # frozen original design + decision record (D1–D14)
├── CHANGELOG.md                 # every /evolve change: date, what, why
├── skills/
│   ├── conventions/             # depth standard, practice philosophy — single source,
│   │                            #   referenced everywhere, never duplicated
│   ├── evolve/references/authoring-guide.md  # the skill/agent authoring rules
│   ├── kickoff/assets/          # file-skeleton templates scaffolded INTO topic repos
│   │                            #   (their CLAUDE.md template ≠ this file)
│   └── <command>/SKILL.md       # one skill per slash command
└── agents/                      # roadmap-drafter and section-drafter
```
