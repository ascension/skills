# Spaces

A **space** is the host's work container for a pane (Herdr workspace). Match an active one, or create.

## Keys

Take these from the spawn intent, filling gaps from the environment (`git branch --show-current`, ticket ref, cwd):

| Key | What it is |
| --- | --- |
| `issue` | Issue or PRD number (`2333` or `#2333`) |
| `branch` | Git branch name |
| `work` | Short work description |
| `cwd` | Worktree path |

## Match

List the host's active spaces (Herdr: `herdr workspace list` plus `herdr worktree list`). Score each space, first rank that hits wins:

1. **Issue** — the number appears in the space label (`#2333` or a bounded `2333`)
2. **Branch** — the branch equals a worktree's `branch` for that space, or appears in the label
3. **Work** — the short description is a case-insensitive substring of the label

Several spaces at the same rank: prefer the one whose worktree `path` equals `cwd`, else the focused space, else the first.

## Create

No match: create a space and use it. Label, in this order of available keys:

```text
#<issue> <work>
#<issue>
<branch>
<work>
```

Herdr: `herdr workspace create --cwd <path> --label '<label>' --no-focus`. The JSON result names `workspace_id` and the first `tab_id`.

Hosts without a space verb spawn in the current session context.
