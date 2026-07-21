# Authoring guide — how to write learn-anything skills & agents

The rules `/evolve` follows when creating or changing plugin components, so quality stays
consistent. This encodes both the Claude Code skill/plugin format and this plugin's own
conventions.

## Slash commands are skills
- Implement every command as a **skill**: `skills/<name>/SKILL.md`. The `commands/` directory
  is a legacy flat-file format — do not use it for new work.
- In a plugin, skills auto-namespace as `/learn-anything:<name>` (and usually `/<name>` when
  unambiguous). Agents are `@learn-anything:<name>`.

## SKILL.md frontmatter
```yaml
---
name: <kebab-case, matches the directory>
description: >-
  <what it does> AND <explicit trigger contexts>.  # see "pushy descriptions" below
disable-model-invocation: false    # Codex-valid; use description/body gates for side effects
argument-hint: "N.K | N"          # optional UI hint for expected args
---
```
- **`name`** — matches the skill directory name.
- **`description` is the triggering mechanism.** Only the first ~1,500 chars are used for
  matching, so front-load them. Claude **under-triggers** — be a little **pushy**: state what
  the command does *and* concrete "use when the user says …" phrasings. Every command skill in
  this plugin follows that pattern; match it.
- **Trigger discipline.** Codex validation rejects the old Claude-only
  `disable-model-invocation: true` value, so shared skills use `disable-model-invocation:
  false` when the field is present. Side-effecting, costly, or mutating commands must enforce
  user intent in two places:
  - Put **User-invoked only** / **Side-effect command** language in the first 1,500 chars of
    the description.
  - Confirm before overwriting files, spending substantial tokens, doing broad web research,
    or mutating topic/plugin state.
  In this plugin that user-invoked-only set is: `kickoff`, `author`, `plan-module`,
  `replan-roadmap`, `update-baseline`, `patch`, `retro`, and `evolve`.
- **Naturally invocable** skills: `mentor`, `quiz`, `recall`, `check`, `checkpoint`, and the
  `conventions` knowledge skill remain safe to trigger from natural language, with any status
  changes still requiring explicit confirmation where the skill says so.
- Arguments arrive as `$ARGUMENTS` (all), `$0`/`$1`/… (positional), or named via an
  `arguments:` list. Command bodies here read `$ARGUMENTS`/`$0` (e.g. `N.K`).

## SKILL.md body
- Keep it **< 500 lines**. It is a *procedure*, not a document: numbered steps, explicit
  "read only these files" (context discipline), and clear stop/refuse conditions.
- **Progressive disclosure:** move long or reference material into `references/*.md` (add a
  table of contents if a reference exceeds ~300 lines) and reusable file templates into
  `assets/`. Link them from the body with relative markdown links so Claude loads them on
  demand rather than paying for them every invocation.
- **One skill = one job.** Don't fold two commands together. Shared knowledge lives in
  `skills/conventions/` and is *referenced*, never copy-pasted into multiple skills.
- Reference sibling files with a relative Markdown link (`references/foo.md`). Relative links
  work across Claude, Codex, and Copilot packaging.

## Locating plugin-internal files (skills vs. agents)
A command's cwd is the **topic repo**, and a subagent's cwd is the topic repo too — neither
points into the installed plugin, so a bare relative path like `skills/conventions/SKILL.md`
does not resolve. Distinguish the two consumers:
- **Skills** link plugin-internal siblings relatively, for example
  `[conventions](../../conventions/SKILL.md)`. Do not depend on a tool-specific plugin-root token.
- **Agents** never open plugin files by path. The orchestrating skill reads the shared
  knowledge and **inlines its text into each agent's prompt**; agents treat that inlined text
  as the normative source. Do not instruct an agent to "read `skills/conventions/SKILL.md`" —
  it cannot.

## Agents (`agents/<name>.md`)
```yaml
---
name: <kebab-case>
description: <what it specializes in and when it's invoked>
tools: Read, WebSearch, WebFetch   # optional — least privilege; add Bash only if it must run code
model: sonnet                      # optional; use the cheapest tier reliable for the task
---
<concise system prompt: role, inputs it will be given, what to produce, output format>
```
- Keep agents **concise and maintainable — more lines ≠ better instructions.** State the role,
  the inputs the orchestrator will pass, the exact output contract, and the quality bar
  (defer to `skills/conventions/SKILL.md` rather than restating it).
- Use least-privilege tools: reading gets `Read`, current facts get web tools, and code
  execution gets `Bash` only when required.
- Plugin-shipped agents may **not** define `hooks`, `mcpServers`, or `permissionMode`, and the
  only valid `isolation` value is `worktree`.
- Default new agents to the cheapest tier that reliably does the job; pin a pricier tier only
  with a stated reason. Keep workflows single pass unless the learner explicitly reruns them.

## Conventions changes are special
The depth standard, practice philosophy, evolution-lens rule, session sizing, and changelog
discipline live **once** in `skills/conventions/SKILL.md`. Changing them is high blast-radius:
every drafter and validator defers to that file. When `/evolve` touches conventions,
the impact check must enumerate every consumer, and the change is almost always at least a
**minor** bump (a **major** bump if it changes the *format* of files in existing topic repos).

## Compatibility & honest guarantees
- Topic repos record a **minimum plugin version** in their `CLAUDE.md` (the version they were
  created against). `/author`, `/plan-module`, and `/replan-roadmap` do a **soft**
  compatibility check at start: they compare it to the installed plugin version and *warn*
  (not block) if the plugin is older. Keep that check working when you change these skills
  or the version machinery.
- Be honest about **model-instruction vs. system-guarantee**. An LLM instruction ("generators
  and mentor MUST design within the spend cap") is not enforcement — don't write copy that
  implies the system *guarantees* it. For cloud spend, billing alarms + teardown are the real
  backstop, and the plugin mandates them as the first practice exercise.

## Versioning reminder (mirror in the changelog)
- **major** — breaking command, packaging, or topic-repo format change → include a migration note.
- **minor** — new command or new behavior.
- **patch** — a fix.
Every `/evolve` change ends with a `plugin.json` bump **and** a `CHANGELOG.md` entry
(date · what · why).
