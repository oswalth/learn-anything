# learn-anything

An AI-assisted self-learning plugin for **Claude Code, Codex, and GitHub Copilot**. It builds
depth-first learning roadmaps, generates challenge-based study sections, reviews your work,
and maintains spaced-repetition cards in a content-only topic repository.

The workflows are intentionally lean: roadmap and section generation each use one drafter
pass, with no separate review agents, automatic revision rounds, or scratch-topic validation.

## How it works

1. Install this plugin once in your AI tool.
2. Create one empty repository per topic.
3. Run `/kickoff` and approve the generated roadmap.
4. Plan, author, study, check, and quiz one section at a time.
5. Commit the topic repository to keep your roadmap, notes, practice, and retention history.

The plugin contains the machinery. Topic repositories contain only learning state and content,
so updating the plugin updates every topic without copying workflow files between repositories.

## Requirements

- Git
- One or more of Claude Code, Codex, GitHub Copilot CLI, or VS Code with GitHub Copilot
- Web access for kickoff research, baseline updates, and fast-moving topics
- Access to `https://github.com/oswalth/learn-anything`

## Claude Code

### 1. Install from GitHub

In Claude Code:

```text
/plugin marketplace add oswalth/learn-anything
/plugin install learn-anything@learn-anything
/reload-plugins
```

Local development fallback:

```text
/plugin marketplace add /absolute/path/to/learn-anything
/plugin install learn-anything@learn-anything
/reload-plugins
```

### 2. Verify

```text
/plugin list
/help
```

The plugin should be enabled and its commands should appear as
`/learn-anything:<command>`; bare command names usually work when unambiguous.

### 3. Create and use a topic

```bash
mkdir -p ~/learn-topics/golang
cd ~/learn-topics/golang
claude
```

Then run `/kickoff` and answer one question at a time. Approve the roadmap explicitly before
the plugin scaffolds modules, retention, and workspace directories.

### 4. Update or remove

```text
/plugin marketplace update learn-anything
/plugin install learn-anything@learn-anything
/reload-plugins

/plugin uninstall learn-anything@learn-anything
/plugin marketplace remove learn-anything
```

## Codex

### 1. Install from GitHub

```bash
codex plugin marketplace add oswalth/learn-anything
codex plugin add learn-anything@learn-anything
```

Local development fallback:

```bash
codex plugin marketplace add /absolute/path/to/learn-anything
codex plugin add learn-anything@learn-anything
```

### 2. Verify

```bash
codex plugin list
```

Start a new Codex thread after installation. Use web-enabled Codex for research-heavy commands.

### 3. Create and use a topic

```bash
mkdir -p ~/learn-topics/golang
cd ~/learn-topics/golang
codex --search
```

Ask: `Use the learn-anything kickoff skill to start a topic for Go.` Codex loads the same
shared skills as Claude and creates both `CLAUDE.md` and `AGENTS.md` for portability.

### 4. Update or remove

Use `codex plugin --help` for the update/remove commands supported by your installed Codex
version. Re-add the plugin after updating a local checkout so its cached files are refreshed.

## GitHub Copilot

The same Copilot-format plugin works in **VS Code Copilot Chat** and **GitHub Copilot CLI**.

### 1. Install from GitHub

VS Code:

1. Enable the `chat.plugins.enabled` setting if agent plugins are not already enabled.
2. Run `Chat: Install Plugin From Source` from the Command Palette.
3. Enter `https://github.com/oswalth/learn-anything`.
4. Confirm the repository trust prompt.

Copilot CLI:

```bash
copilot plugin install https://github.com/oswalth/learn-anything
```

VS Code automatically discovers plugins installed by Copilot CLI. Local development fallback:

```bash
git clone https://github.com/oswalth/learn-anything.git
```

Then add the checkout with VS Code's `chat.pluginLocations` setting:

```json
{
  "chat.pluginLocations": {
    "/absolute/path/to/learn-anything": true
  }
}
```

### 2. Verify

VS Code:

1. Open Extensions and search `@agentPlugins @installed`.
2. Confirm `learn-anything` is enabled.
3. Type `/` in Copilot Chat and look for `/learn-anything:kickoff`.

Copilot CLI:

```bash
copilot plugin list
```

In an interactive CLI session, `/skills list` should include the learn-anything skills.

### 3. Create and use a topic

Create and open an empty directory in VS Code, then run `/learn-anything:kickoff` in Copilot
Chat. In Copilot CLI:

```bash
mkdir -p ~/learn-topics/golang
cd ~/learn-topics/golang
copilot -i "Use the /kickoff skill to start a learn-anything topic for Go"
```

Copilot creates `CLAUDE.md` as the portable topic record and `AGENTS.md` as its native project
instructions.

### 4. Update or remove

VS Code checks plugin updates with `Extensions: Check for Extension Updates`. Manage enable,
disable, and uninstall actions in the `@agentPlugins @installed` Extensions view.

```bash
copilot plugin update learn-anything
copilot plugin uninstall learn-anything
```

## Daily workflow

The command names and behavior are the same in all three tools:

```text
/kickoff            interview, research, draft a roadmap, stop for approval
/plan-module 1      split approved module 1 into 1-2 hour sections
/author 1.1         generate five section files in one drafter pass
/study 1.1          walk through theory and challenges interactively
/check 1.1          review your work against validation criteria
/quiz 1.1           run free recall and create cards for misses
/recall             review cards that are due
/check 1            validate the whole module and mark it done on approval
/checkpoint         run cumulative review at a phase boundary
```

Use `/learn-anything:<command>` where a tool requires or displays the plugin namespace.

## Command reference

| Command | Purpose |
|---|---|
| `/kickoff` | Interview, research, generate one roadmap draft, and scaffold after explicit approval |
| `/plan-module N` | Break an approved module into 1-2 hour sections |
| `/author N.K` or `/author N` | Generate or regenerate section content in one drafter pass |
| `/replan-roadmap` | Generate one revised roadmap draft while protecting started modules |
| `/study N.K` | Interleave theory with the challenges that build on it |
| `/check N.K` or `/check N` | Review learner work and update mastery status on confirmation |
| `/mentor [N.K or N]` | Run a Socratic mentoring session without giving away solutions |
| `/quiz N.K` | Run a free-recall quiz and create retention cards for misses |
| `/recall [module N]` | Review due spaced-repetition cards |
| `/checkpoint` | Run cumulative review and an integration challenge |
| `/patch N.K "note"` | Make one bounded correction to generated content |
| `/retro` | Improve the current topic and emit plugin-level evolve briefs |
| `/update-baseline` | Re-verify pinned versions and flag potentially stale sections |
| `/evolve` | Change this plugin in one pass and synchronize all tool packages |

Mutating or research-heavy commands require explicit user intent. Re-running `/author` on
generated or studied content asks before overwriting and resets a studied section to generated.

## Topic repository

After roadmap approval, a topic contains:

```text
topic/
├── CLAUDE.md        # portable topic source of truth and change history
├── AGENTS.md        # Codex/Copilot project instructions
├── BASELINE.md      # pinned versions, editions, and authoritative URLs
├── ENVIRONMENT.md   # setup, budget, spend cap, and teardown checklist
├── roadmap.md       # phases, modules, prerequisites, and module status
├── modules/         # section plans, generated content, and learner notes
├── retention/       # spaced-repetition cards and review log
└── workspace/       # learner-owned code, labs, proofs, and capstone work
```

Generated section files are disposable and may be regenerated. Topic instructions, roadmap,
module notes, retention history, and workspace content are persistent.

## Lean architecture

- `/kickoff` and `/replan-roadmap` call one roadmap drafter.
- `/author` calls one section drafter per section.
- `/patch` applies one bounded edit.
- `/evolve` reads the affected files, edits once, runs one structural check, synchronizes all
  manifests, and updates the changelog.
- Quality requirements live once in `skills/conventions/SKILL.md` and are supplied directly to
  drafters and learning workflows.

The learner approval gate still protects roadmap scope, and `/check` still validates learner
work. The removed review loops only judged AI-generated artifacts repeatedly.

## Repository layout

```text
learn-anything/
├── plugin.json                       # Copilot plugin manifest
├── .github/plugin/marketplace.json   # Copilot marketplace
├── .github/agents/                   # Copilot-native drafter agents
├── .claude-plugin/                   # Claude manifest and marketplace
├── .codex-plugin/                    # Codex manifest
├── .agents/plugins/marketplace.json  # Codex marketplace
├── .codex/agents/                    # Codex-native drafter agents
├── agents/                           # Claude-native drafter agents
├── skills/                           # shared commands, adapters, and conventions
├── CHANGELOG.md
├── CLAUDE.md
└── AGENTS.md
```

## Troubleshooting

**Plugin commands do not appear:** restart the tool or reload plugins, then confirm the plugin
is installed and enabled. In VS Code, verify `chat.plugins.enabled` and search
`@agentPlugins @installed`.

**Marketplace install fails:** verify `git ls-remote https://github.com/oswalth/learn-anything`
works with your credentials. GitHub shorthand is `oswalth/learn-anything`, not a browser URL.

**A command name conflicts:** use `/learn-anything:<command>`.

**An updated plugin still behaves like the old version:** update the marketplace, reinstall or
update the plugin, and start a fresh chat/thread so cached skill instructions are reloaded.

**`/evolve` refuses to run:** clone this repository and run it from the plugin checkout, not a
topic repository.
