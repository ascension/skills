# Pi adapter

Load only when the host is Pi.

## Spawn

Run from the assigned worktree. Load the user-only skills explicitly. `--no-session` is the fresh-session flag. Pass the resolved `model` and map `effort` from the current `pi --help`.

```text
pi -p --no-session --model <id> \
  --skill <agents-skills>/implement \
  --skill <agents-skills>/tdd \
  --skill <agents-skills>/code-review \
  @<packet.md>
```

Pi qualifies for implementation only when an installed extension supplies the subagent support required by the nested `/code-review`. If that gate fails, ask for a **pin** to Herdr or native; the nested review remains part of `/implement`.

`pi --model cursor/grok-4.6` is not a working host path; Cursor Grok is `cursor-agent` on another host.

## Wait and read

Wait on the `pi -p` process. The receipt is its stdout.
