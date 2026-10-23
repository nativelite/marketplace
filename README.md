# nativelite marketplace

A plugin marketplace for the **nativelite** agent-terminal suite, for
[Claude Code](https://docs.claude.com/en/docs/claude-code) **and**
[Codex](https://developers.openai.com/codex) alike. Install nativelite's skills
and tooling straight into either, instead of copying folders by hand. The skills
are written for whichever agent reads them: a codex lead spawns codex teammates,
a claude lead claude ones, and a fleet file may mix the two.

## Add the marketplace

```
/plugin marketplace add nativelite/marketplace
```

## Plugins

### `atrium` — coordination skills, for claude and codex

The three skills that teach a hosted agent, claude or codex, to use
[atrium](https://github.com/nativelite/atrium)'s control plane. Install them in
Claude Code with:

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
atrium itself tells every agent pane it launches, claude or codex, that `atrium
ctl` is there; the skills carry the full playbook (the coordinate loop, the fleet
file's every key, codex's permission block, when to delegate).

The skills teach the shell verbs (`atrium ctl …`), which both agents run the
same way. One piece is Claude Code only: atrium's **mod**, a plugin of function
hooks that gives a *claude* pane the same control plane as typed `atrium_*`
tools, reports its status for certain, and runs subagents as visible panes. It is
not here: it ships inside the atrium binary and is written out with `atrium mod
install`, so it always matches the atrium it talks to. A codex pane has no mod
and needs none: it is sent to and reports through the shell verbs. See the
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
