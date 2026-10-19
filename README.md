# nativelite marketplace

A [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin marketplace for
the **nativelite** agent-terminal suite. Install nativelite's skills and tooling
straight into Claude Code instead of copying folders by hand.

## Add the marketplace

```
/plugin marketplace add nativelite/marketplace
```

## Plugins

### `atrium` — coordination skills

The three skills that teach a hosted agent to use [atrium](https://github.com/nativelite/atrium)'s
control plane. Install them in Claude Code with:

```
/plugin install atrium@nativelite
```

or in [Codex](https://developers.openai.com/codex), which reads this marketplace as it is:

```
codex plugin marketplace add nativelite/marketplace
codex plugin add atrium@nativelite
```

Codex lists them as `atrium:atrium-coordinate` and so on. To install by hand
instead, copy the three `atrium/skills/` folders into `~/.agents/skills/` (every
project) or a repo's `.agents/skills/`.

- **`atrium-delegate`** — the agent hands off a substantial *independent* piece of
  work when it genuinely pays, and does small or dependent work inline.
- **`atrium-coordinate`** — casts the agent as a coordinator that splits a task
  across a team of teammate agents, delegates every part via `atrium ctl`,
  monitors, collects, and reaps.
- **`atrium-fleet`** — designs, configures and runs a saved `atrium.fleet.json`
  roster: the instruction layer for `atrium fleet up`.

Without these, a hosted agent does not know atrium's control plane exists, so
install them before running `atrium --allow-ctl` sessions or `atrium fleet up`.
(From atrium 0.40.2 a codex pane is also told about `atrium ctl` by atrium
itself, the way a claude pane always was; the skills carry the full playbook.)

The skills teach the shell verbs (`atrium ctl …`). atrium's **mod**, the Claude
Code plugin of function hooks that gives a pane the same control plane as typed
`atrium_*` tools, reports its status for certain, and runs subagents as visible
panes, is not here: it ships inside the atrium binary and is written out with
`atrium mod install`, so it always matches the atrium it talks to. See the
[atrium README](https://github.com/nativelite/atrium#install-the-mod).

## Layout

```
.claude-plugin/marketplace.json   # the marketplace manifest
atrium/                           # the atrium plugin
  .claude-plugin/plugin.json
  skills/atrium-delegate/SKILL.md
  skills/atrium-coordinate/SKILL.md
  skills/atrium-fleet/SKILL.md
```

More plugins will land here as the suite grows. MIT licensed.
