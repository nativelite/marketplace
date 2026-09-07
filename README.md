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

Without these, a hosted agent does not know atrium's control plane exists, so
install them before running `atrium --allow-ctl` sessions or `atrium fleet up`.

## Layout

```
.claude-plugin/marketplace.json   # the marketplace manifest
atrium/                           # the atrium plugin
  .claude-plugin/plugin.json
  skills/atrium-delegate/SKILL.md
  skills/atrium-coordinate/SKILL.md
```

More plugins will land here as the suite grows. MIT licensed.
