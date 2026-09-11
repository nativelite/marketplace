---
name: atrium-fleet
description: Design, configure, write, launch, and manage a saved atrium fleet — the `atrium.fleet.json` roster you bring up with `atrium fleet up`. Use when someone wants to build a fleet, edit or troubleshoot a fleet file, choose trust/worktree/coordination settings, or ask how fleets work. This is the authoring/runner counterpart to atrium-coordinate (runtime `ctl` leading) and atrium-delegate (runtime `ctl` hand-off).
---

# Building & running atrium fleets (`atrium.fleet.json`)

You are the **fleet architect**. Someone wants a team of agents they can bring up
with one command and re-run later. Your job is to turn their goal into a correct,
reviewable `atrium.fleet.json`, explain the choices, and hand them the exact launch
line — and to answer questions about how fleets work.

This is different from the sibling skills, and you should route accordingly:

- **atrium-fleet (this one)** — *author and run a saved roster*. A file on disk,
  launched with `atrium fleet up <name>`, re-runnable, diff-reviewable.
- **atrium-coordinate** — you are *already running* and told to lead a team; you
  spawn teammates live with `atrium ctl spawn`.
- **atrium-delegate** — you are *already running* and hand off one independent part
  live with `atrium ctl spawn`.

If the person wants a repeatable, reviewable team definition → stay here. If they
want you to fan work out *right now* from inside a running pane → the runtime skills.

## The workflow you run with them

1. **Understand the goal.** What repo/dir? How many agents, doing what? Do the
   agents need to talk to each other (coordination) or just run in parallel? Should
   each work in isolation (worktrees) or share one tree? Hands-off or supervised?
   Ask — don't assume. A fleet is worth getting right because it's re-run.
2. **Propose a roster** in prose first: one line per agent (name, model, what it
   owns, whether it gets a worktree). Get a nod before writing JSON.
3. **Write `atrium.fleet.json`** (schema below). Prefer explicit `model`/`worktree`
   keys over stuffing flags into `cmd`. Keep kickoffs self-contained.
4. **Explain the launch line and the settings you chose**, then let them run it.
5. **Iterate.** Fleet files are cheap to edit and re-run; tune and relaunch.

## Where the file lives (discovery)

atrium looks for the fleet file **cwd-first, then user-global** (fallback, not
merge):

1. `./atrium.fleet.json` in the directory you launch atrium from — this is the
   normal, project-local home (check it into the repo so the team is shared).
2. else the user-global file: `%APPDATA%\atrium\fleet.json` (Windows) or
   `~/.config/atrium/fleet.json` (elsewhere).

A cwd file **shadows** the global one — if both exist, only the cwd file is read.
So a per-project fleet must live in that project's `atrium.fleet.json`. `atrium
fleet ls` lists the fleets in whichever file was found.

## Schema

One file holds a `fleets` map of `name → fleet`. Unknown keys are ignored (forward-
compatible), and an empty `agents` list is rejected.

### Fleet-level keys

| key | type | meaning |
|-----|------|---------|
| `agents` | array (required) | the roster, one pane each, in order |
| `grid` | `"RxC"` | tile layout, e.g. `"2x3"`; omit → auto-balanced from agent count |
| `identity` | string | default credential for agents that don't name their own |
| `trust` | `"plan"`\|`"accept"`\|`"automode"`\|`"skip"` | fleet trust posture when the CLI doesn't pass `--trust`. A **request**, capped by the session ceiling |
| `allow_ctl` | bool | bring up the board/bus control plane (same as `--allow-ctl`). **Required for any fleet whose agents coordinate** — without it, kickoffs that publish to a bus silently do nothing |
| `topics` | array of strings | declare the canonical bus vocabulary → bus runs **strict** (a topic outside the list is rejected, so the team converges on one set). Omit → soft-gated |
| `worktrees` | bool | shorthand: every agent gets its **own** worktree named after itself (full fan-out) |
| `worktree_base` | string | where worktree dirs go; omit → sibling `../.atrium-worktrees/<fleet>/`. Set it to give two concurrent sessions on one repo distinct bases |
| `worktree_seed` | array of strings | untracked files/dirs to link into each fresh worktree (git only checks out tracked files, so a needed `.env` would be missing). Hardlinks files / junctions dirs; never seeds build caches |

### Agent-level keys

| key | type | meaning |
|-----|------|---------|
| `name` | string (required) | label / role handle |
| `cmd` | array (required) | the command + args, e.g. `["claude"]` or `["codex"]` |
| `kickoff` | string | trailing positional prompt → the agent's **first user message**, so it starts working the instant the fleet comes up. Omit → it idles |
| `model` | string | `--model <m>` (`opus`\|`sonnet`\|`haiku`\|…) |
| `effort` | string | `--effort <e>` |
| `worktree` | string | **group name** for tree isolation: a unique value = solo worktree+branch; a value **shared** with other agents = they co-develop one worktree; absent = stays in the main tree |
| `trust` | policy | per-agent trust override (see mixed-model note below), capped by the session ceiling |
| `cwd` | string | working dir (so its `CLAUDE.md` auto-loads); resolved relative to the fleet file |
| `add_dirs` | array | extra readable dirs → `--add-dir` |
| `prompt` | string | system-prompt suffix → `--append-system-prompt` (behavior, not permissions) |
| `identity` | string | per-agent credential, overrides the fleet default |
| `can_spawn` | bool | may this agent create its own teammates via `atrium ctl spawn`? Defaults **false** for a fleet agent |

Launch arg order per agent: `cmd… [--add-dir …] [--append-system-prompt prompt]
[--model M] [--effort E] "<kickoff>"`. You can equivalently inline `--model` in
`cmd`, but the `model` key reads cleaner in a diff.

### Minimal fleet

```json
{
  "fleets": {
    "review": {
      "grid": "1x2",
      "agents": [
        { "name": "security", "cmd": ["claude", "--model", "opus"],
          "kickoff": "Read PLAN.md. Audit src/auth/ for authz bugs and secret handling. Write findings to REVIEW-security.md. Do not edit source." },
        { "name": "perf", "cmd": ["claude", "--model", "sonnet"],
          "kickoff": "Read PLAN.md. Profile the hot path in src/render/. Write findings to REVIEW-perf.md. Do not edit source." }
      ]
    }
  }
}
```

### Coordinated, isolated build fleet (the powerful shape)

```json
{
  "fleets": {
    "build": {
      "grid": "2x3",
      "allow_ctl": true,
      "topics": ["build", "review"],
      "agents": [
        { "name": "lead", "cmd": ["claude", "--model", "opus"],
          "kickoff": "You lead. Read PLAN.md. Assign via board/bus, don't code. `atrium ctl bus sub build review`." },
        { "name": "api", "cmd": ["claude", "--model", "sonnet"], "worktree": "api",
          "kickoff": "You own src/api.rs in your own worktree (already placed — don't cd, commit on your branch). Implement per PLAN.md, `python dev.py check` green, commit, then `atrium ctl bus pub build msg=api done`." },
        { "name": "store", "cmd": ["claude", "--model", "sonnet"], "worktree": "store",
          "kickoff": "You own src/store.rs in your own worktree. … `atrium ctl bus pub build msg=store done`." },
        { "name": "reviewer", "cmd": ["claude", "--model", "opus"],
          "kickoff": "Adversarial reviewer in the main tree. Verify each branch against real behavior; block the integrator on the bus until issues are fixed. Don't rubber-stamp." },
        { "name": "integrator", "cmd": ["claude", "--model", "sonnet"],
          "kickoff": "Main tree. Wait until api+store report done and reviewer is clear, then merge their branches, run the gate green, commit. Don't reinstall." }
      ]
    }
  }
}
```

## Launch & manage

```
atrium fleet up <name> [--allow-ctl] [--trust <policy>] [--max-depth N]
atrium --trust automode [--allow-ctl] up <name>     # flags-first form (same thing)
atrium fleet ls                                     # list fleets in the found file
atrium fleet clean <name>                           # reclaim worktrees a run kept
```

**Flags go BEFORE `up` in the short form** (`atrium --trust automode up build`). If
you write `atrium up build --trust automode`, atrium parses `up` as a program and
the fleet never launches through the fleet path — the classic silent miss. When in
doubt use the explicit `atrium fleet up build --trust automode`. If the fleet file
already sets `trust`/`allow_ctl`, you don't need the flags at all.

## Trust & hands-off (read this before setting `automode`)

The trust posture is what makes a fleet run without a human clicking "allow":

- `plan` = read-only, `accept` = auto-accept **edits** + a safe command allowlist,
  `automode` = claude's own auto mode (edits **and** commands, no allowlist),
  `skip` = full bypass. Bare `--trust` = `accept`; `--skip-permissions` = `skip`.
- Every fleet/agent trust value is a **request capped by the session ceiling**. An
  agent can de-escalate (run `accept` under an `automode` session) but never
  escalate past what the human approved at launch.
- **`automode` is model-gated by Anthropic.** haiku reports *"this model does not
  have automode"* — so a fleet launched `--trust automode` leaves its haiku agents
  unable to run commands hands-off, and `automode` carries **no** command allowlist
  anyway. Fix for mixed-model fleets: set `"trust": "accept"` on the non-automode
  agents. `accept` gives `acceptEdits` + the allowlist, which **every** model
  honors. This is the main reason the per-agent `trust` key exists.
- The `accept` allowlist runs common dev commands hands-off (cargo/npm/pytest
  builds+tests, and git `add`/`commit`/`merge`/`worktree`/`status`/`log`/`diff`)
  but **deliberately still prompts** on `git push`/`reset`/`clean`. Don't try to
  widen it with raw claude flags in `cmd` — use `trust`.
- atrium also pre-accepts claude's separate **folder-trust** dialog per pane cwd,
  so a hands-off launch in a fresh dir doesn't stall on "trust this folder?".

Rule of thumb: all-opus/sonnet coordinating fleet → launch `--trust automode`. Any
haiku in the roster → give those agents `"trust": "accept"`.

## Worktrees (per-agent isolation)

Turn this on when agents would otherwise fight over one working tree — two agents
editing files, or one agent's half-done WIP failing another's green gate.

- Set `"worktree": "<name>"` per agent (unique = solo, shared = a squad on one
  concern), or `"worktrees": true` to fan every agent out into its own.
- Each worktree is a real `git worktree` on branch `atrium/<fleet>/<slug(name)>`,
  checked out at `<base>/<fleet>/<slug(name)>` (base default = sibling
  `../.atrium-worktrees`).
- **Single-owner-per-file is the invariant that keeps merges clean.** Give each
  worktree a disjoint set of files; tell each agent in its kickoff to touch only
  its files and to STOP + post on the bus if it needs another's. Then the final
  merge is conflict-free by construction.
- **Teardown is safe by default.** On fleet exit atrium reclaims worktrees that are
  **clean and merged**, and **keeps** any that are dirty or unmerged (so nothing is
  lost). `atrium fleet clean <name>` reclaims kept-but-now-merged ones later.
- Kickoff norms for worktree agents: *your cwd already IS the worktree — don't cd,
  run no `git worktree` commands, commit on your current branch.*
- **Pitfall — relative sibling path deps.** If the repo's build pulls sibling
  crates/packages by **relative** path (Rust `path = "../abus"`, or a JS monorepo
  with `../pkg` links), those don't resolve from inside a worktree (it sits at a
  different depth), so the build breaks there even though it's fine in the main
  tree. Either seed/junction the siblings beside the worktrees, or keep those
  agents in the main tree. Check the build's path deps before fanning out a
  multi-crate repo. (A standalone single-crate repo has no such issue.)

## Coordination wiring

A fleet whose agents talk needs three things, or it silently no-ops:

1. `"allow_ctl": true` — brings up the board/bus. Without it, every `atrium ctl
   bus/board` call in a kickoff fails quietly.
2. **Self-contained kickoffs.** Each agent starts **blank** — it can't see your
   plan or the others. The kickoff must carry the goal, exact file paths,
   constraints, the done-signal, and the bus/board commands to use. Write a
   paragraph, not "do the api part". Commit a `PLAN.md` if agents share context —
   but note **worktree** agents only see files committed at HEAD, so either commit
   the plan or inline it into each kickoff.
3. Optional `"topics": [...]` to lock the bus vocabulary so the team doesn't
   fragment into `review`/`reviews`/`review-gate`.

The runtime coordination commands (what your kickoffs tell agents to run):
`atrium ctl board set/get/list`, `atrium ctl bus pub/sub/feed/resolve`. See the
atrium-coordinate skill for the full board/bus playbook.

## Recipes

- **Parallel review** — N read-only agents (`plan` or just "don't edit"), each
  owning a dimension, writing to its own `REVIEW-*.md`. Often no worktrees needed
  (read-only doesn't collide). Grid `1xN`.
- **Build team** — lead (main tree, delegates) + workers (each a worktree, disjoint
  files) + adversarial reviewer (main tree) + integrator (main tree, merges +
  gates). `allow_ctl: true`, `--trust automode`. The shape shown above.
- **Tool-develops-itself (bootstrap loop)** — when the fleet edits the very tool
  it's running on, the running binary is **locked**: agents develop in worktrees →
  gate green → merge → the human reinstalls (`cargo install --path . --force` or
  equivalent) → restart. There is no hot-reload; changes go live only on reinstall.
  Don't have an agent try to reinstall the binary that's hosting it.

## Gotchas checklist

- Coordinating fleet but no `allow_ctl` → bus/board silently dead.
- `--trust automode` with a haiku agent → that agent can't run commands; add
  `"trust": "accept"` to it.
- Flags after `up` → fleet doesn't launch through the fleet path.
- Worktree agents can't see uncommitted/untracked files (they're checked out from
  HEAD) → commit shared context or inline it.
- Multi-crate repo with relative sibling path deps + worktrees → worktree builds
  break; seed the siblings or stay in the main tree.
- cwd `atrium.fleet.json` shadows the global file → edit the right one.
- Two overlapping-file agents without disjoint ownership → merge conflicts; enforce
  single-owner-per-file.

## Discovery

`atrium fleet ls` (fleets in scope), `atrium fleet up`/`clean` usage on bad args,
and `atrium ctl` with no arguments (or `atrium --help`) for the authoritative,
current command surface. When unsure of a flag or key, check those rather than
guessing — the surface is the source of truth.
