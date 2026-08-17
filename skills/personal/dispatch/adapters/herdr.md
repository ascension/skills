# Herdr adapter

Load only when the host is Herdr.

Herdr is the observable terminal. The model is the command after `--`. Confirm `--help` and `--list-models` for the chosen runner before spawn. `HERDR_PANE_ID` may be unset even inside a pane; `herdr pane current` is the live check.

## Space

List `herdr workspace list` and `herdr worktree list`. Match and create as [SPACES.md](../SPACES.md). Spawn into the resolved `workspace_id` and its `active_tab_id` (or the new workspace's first tab).

## Spawn

`herdr agent start` creates the pane when given `--split`. It does not take `--kind` or `--pane`.

```text
herdr agent start <name> \
  --cwd <worktree> \
  --workspace <resolved-workspace-id> \
  --tab <resolved-tab-id> \
  --split right \
  --no-focus \
  -- <fresh runner>
```

`<name>` is the spawn intent's `name`. Arguments after `--` must start a **fresh session** and pass the resolved `model` and `effort`.

Verify the runner's current flags. Pattern that has worked for Cursor-in-Herdr:

```text
cursor-agent -p --trust --workspace <worktree> --model <id> <packet>
```

Map `effort` onto the catalog id (for Cursor Grok, `medium` → `cursor-grok-4.6-medium`). A Herdr pane wrapped around a resumed agent (`--resume` / `--continue`) fails the gate.

Print-mode may report `idle` while still running, then drop the agent name and close the pane when the process exits. Persist the receipt to a file the packet names (or `tee` stdout) and treat that file as the read if `herdr agent read` is empty or the target is gone.

`pi --model cursor/grok-4.6` is not a working host path; Cursor Grok is `cursor-agent`.

## Wait and read

```text
herdr agent wait <name> --status idle --timeout <milliseconds>
herdr agent read <name> --source recent-unwrapped --lines <count>
```

`wait` accepts one `--status`: `idle`, `working`, `blocked`, or `unknown`. If read is empty and a receipt file was written, that file is the receipt. Confirm the runner can see the Matt skills and create subagents before the first mutating leg.
