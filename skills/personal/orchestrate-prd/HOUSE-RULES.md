# House rules

User-level map from each **role** to a default model and **effort**. Machine and subscription fact, not repo config.

## File

`~/.agents/orchestrate-prd.yaml`

```yaml
defaults:
  implement: { model: opus, effort: high }
  review:    { model: gpt-5.6-sol, effort: xhigh }
  verify:    { model: grok-4.6, effort: medium }
  recover:   { model: opus, effort: high }
  probe:     { model: grok-4.6, effort: low }
```

Every role the run will dispatch must have a row. `model` is a name the host adapter must resolve. `effort` is `medium`, `high`, or `xhigh`.

Missing file, missing role, or unreadable YAML: stop and ask.

## Resolution

Later wins. After pin, model is frozen on the ledger row.

1. House-rules default for the task's role
2. Ticket annotation — a `model:` and/or `effort:` line on the ticket
3. Approved **suggestion**, written on the ledger at pin
4. Invocation override for a role for this whole run

The coordinator sets **effort** on each spawn from the task in front of it. Model stays on the ledger row.

## Suggestions

While pinning, scan the graph and propose model overrides that the role default cannot carry. Present them. Wait. Approved → write the model on that ledger row. Declined → keep the role default. After the relay opens, raise no further suggestions.
