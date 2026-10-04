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

The two skills that teach a hosted `claude` to use [atrium](https://github.com/nativelite/atrium)'s
control plane. Install them with:

```
/plugin install atrium@nativelite
```

- **`atrium-delegate`** — the agent hands off a substantial *independent* piece of
  work when it genuinely pays, and does small or dependent work inline.
- **`atrium-coordinate`** — casts the agent as a coordinator that splits a task
  across a team of teammate agents, delegates every part via `atrium ctl`,
  monitors, collects, and reaps.
- **`atrium-fleet`** — designs, configures and runs a saved `atrium.fleet.json`
  roster: the instruction layer for `atrium fleet up`.

Without these, a hosted agent does not know atrium's control plane exists, so
install them before running `atrium --allow-ctl` sessions or `atrium fleet up`.

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
