# Notes — runtime adapters and spawn policy

Human exploration for this skill only. Not loaded by `SKILL.md`. Do not treat as instructions.

**Status:** topic 1 written. Adapters live in `dispatch`. Topic 2 (Herdr workspace/tab chrome) still parked.
**Date:** 2026-08-16

## Goal

Keep `/orchestrate-prd` as the **control plane** — ledger, frontier, packets, receipts, failure bounds, review loop — and stop baking environment verbs into that plane. Users should be able to run the same skill in Buzz, Herdr, a native harness, or whatever comes next, and have the coordinator spawn and track sub-agents the way that environment actually works.

Separately, users must be able to set default models and override them per task. Effort is not a static agent property: the coordinator chooses model and effort for each spawn from the task it is handing off.

## The split

The control plane is not what needs to be agnostic. This skill *is* the control plane.

What needs to be swappable is the **runtime adapter**: the thin wiring that turns a portable spawn intent into the local verbs for creating, mentioning, waiting on, and reading agents.

```
control plane (this skill)
  decides: what to run, when, with which model + effort, what "done" looks like
        │
        │  spawn intent
        │  {name, packet, model, effort, isolation, wait}
        ▼
runtime adapter (environment wiring)
  Buzz  → @mention / buzz messages send --mention / wait on channel reply
  Herdr → named workspace + stage tabs; pane split + herdr agent start / wait / read
  Native → isolated Agent/Task primitive, zero inherited turns
  Pi     → print-mode one-shot
        │
        ▼
  receipt back to the ledger
```

Today those two layers are mixed in [`FRESH-SESSIONS.md`](./FRESH-SESSIONS.md). The capability gate and packets belong to the skill. The Herdr command recipes, Pi flags, and hardcoded model IDs belong to adapters — and Buzz is missing entirely.

## What exists today

Pinned in the skill:

- One launcher chosen at "open the relay", then reused for every leg.
- Approved models hardcoded: Claude Opus, Codex GPT-5.6-sol, Cursor Grok 4.6.
- Effort only appears as a model-slug suffix (`cursor-grok-4.6-high`), not as a first-class knob.
- Review "prefers a different approved model family" but has no config and no selection rule.
- Isolation is a hard gate: new process / session / subagent, no inherited turns, no resume.

This machine already has both candidate hosts:

- `herdr` at `~/.local/bin/herdr` — pane/agent lifecycle (`start`, `wait`, `read`, `prompt`).
- `buzz` at `~/.local/bin/buzz` — channel messages with `--mention`; agents are reactive and wake on `@mention`. Nest at `~/.buzz/`. `HERDR_ENV` and `BUZZ_RELAY_URL` are unset in this shell, so detection cannot be "env var present" alone.

## Portable spawn intent

The skill should never say `herdr agent start` or `@Duncan`. It should emit an intent the adapter translates:

| Field | Meaning |
| --- | --- |
| `name` | Unique leg name: phase, task id, attempt |
| `packet` | The existing implementation / review / verify / recover packet |
| `role` | `implement` \| `review` \| `verify` \| `recover` \| `probe` |
| `model` | Resolved local model id, after defaults + per-task + coordinator judgment |
| `effort` | Portable thinking level for this spawn |
| `isolation` | Fresh context required (current gate) |
| `wait` | Until idle / done / blocked, with timeout |

The adapter's job is those five verbs: **detect**, **spawn**, **wait**, **read**, **track**. A host may also **project** control-plane progress onto its own chrome (Herdr workspace/tab labels). Everything else stays in the skill.

## Model and effort

Three layers, applied in order. Later wins.

1. **User defaults** — house rules: default model and effort per role (`implement`, `review`, `verify`, `recover`).
2. **Per-task override** — this ticket, this finding, this verification run.
3. **Coordinator judgment** — the orchestrating agent must still pick the right pair for *this* handoff. A probe is not an aggregate review. A one-line fix is not a first implementation of a hard ticket. Effort is dynamic because the task is.

The coordinator does not inherit "whatever the worker's profile says." It passes model and effort at spawn time.

Buzz already speaks this language: app-wide Agent Defaults, then per-agent Model and Effort (`medium` / `high` / `xhigh`). The skill should use the same portable effort vocabulary and let each adapter map it onto local flags (`--model`, Codex reasoning effort, Cursor slug suffix, Buzz agent config).

Sketch, not a decided format:

```yaml
defaults:
  implement: { model: opus, effort: high }
  review:    { model: gpt-5.6-sol, effort: xhigh }
  verify:    { model: grok-4.6, effort: medium }
  recover:   { model: opus, effort: high }
  probe:     { model: grok-4.6, effort: low }

# optional, on a ticket or ledger row
tasks:
  TICKET-12: { model: opus, effort: xhigh }
```

Open: where this file lives (user-level vs repo `docs/agents/`), and whether the skill speaks **roles** that map to local IDs or raw local model IDs.

## The live tension: Buzz standing agents vs the fresh-session gate

Herdr and native isolated agents can satisfy the current gate: new process, no inherited turns.

Buzz's native mechanic is the opposite. Agents are standing players. They have instructions, core memory, and channel history. You do not spawn a throwaway process; you `@mention` a named agent and it takes a turn. Delegation is mention-in-channel; completion is mention-the-delegator-back.

That is the design fork that matters most:

- **Keep the isolation gate.** The Buzz adapter must still produce a fresh context per leg (ephemeral agent, one-shot session, or a mention whose only task context is the packet). Standing memory becomes the adapter's problem, not a reason to weaken the skill.
- **Relax the gate for Buzz.** Named party members (orchestrator / executor / reviewer) are the unit of work. Isolation is "self-contained packet," not "empty context."
- **Hybrid.** Fresh session for mutating implement/fix legs; standing named reviewer is allowed for aggregate review.

This is not a packaging question. It decides whether Buzz is a first-class host or a translation that fights the skill.

## Proposed shape (when we build)

Do not implement yet. When we do:

1. Keep `SKILL.md` as the control plane. It already is.
2. Shrink `FRESH-SESSIONS.md` to the gate + packets + a pointer: load the adapter for the detected or pinned runtime.
3. Add disclosed adapter files, one per host, loaded only on that branch:
   - detect / pin
   - how to spawn with model + effort
   - how to wait and read
   - how to prove freshness (or the decided Buzz exception)
4. Add a user-facing defaults file for model/effort, with per-task overrides recorded on the ledger row.

Scope is this skill. Do not extract a shared runtime for other skills unless a second caller appears later.

## Herdr workspace layout

When the host is Herdr, the adapter should make the run *visible* as a space, not a pile of anonymous panes.

**Workspace.** One Herdr workspace per PRD run. Label leading text is the PRD number, then the task name:

```text
#2333 runtime adapters
```

`herdr workspace create --cwd <worktree> --label '#2333 …'` (or `workspace rename` if the space already exists). Creating a workspace also creates its first tab and root pane.

**Stage tabs.** Panes for a given phase live in a tab marked for that phase — emoji, name, and a live `n/m` count. Sketch:

| Stage | Tab label shape | What the count is (open) |
| --- | --- | --- |
| Prototyping | `✨ Prototype 2/3` | prototype questions settled |
| Planning | `📋 Plan 5/5` | tickets / decisions ready |
| Implementing | `🔨 Implement 3/8` | ledger implement legs done |
| Testing | `🧪 Test 0/4` | verification / checks green |
| Shipping | `🚀 Ship 0/1` | PR published + review-clean |

`herdr tab create --workspace <id> --label '…'` then `herdr tab rename <tab_id> '…'` whenever the ledger changes. Spawn the matching leg into an available pane *in that tab* (`pane split` there, then `agent start`). Do not dump every agent into the first tab.

**Keep labels current.** Status is not a one-time decorate. After each receipt the adapter rewrites the tab label (`3/8` → `4/8`, `5/5` when the stage is done). The control plane still owns the numbers; Herdr only projects them. `pane rename` / `pane report-metadata --custom-status` can mark the active pane, but the tab label is the thing you scan.

**Focus.** Create and rename with `--no-focus` unless the coordinator is moving the human to that stage. Do not steal the focused pane on every status tick.

**Phase coverage.** This skill today starts after `/to-tickets` (implement → verify → publish → review). The tab set is the *whole* process so the space tells one story. If we enter at orchestrate-prd, Prototype and Plan can already read `5/5` (or be omitted — open). Buzz and native have no tab chrome; they do not fake this. A Buzz projection, if we ever want one, would be channels or a canvas, not Herdr labels.

Herdr verbs that matter here: `workspace create|rename`, `tab create|rename|focus`, `pane split|rename|move --tab`, plus the existing `agent start|wait|read`.

## Settled (topic 1, grilling)

- **Discovery** — detect the live host (not “is the CLI on PATH”). On a tie, fail and ask for a **pin**.
- **Isolation** — every relay leg is a **fresh session** on every host ([ADR 0001](./docs/adr/0001-fresh-session-on-every-host.md)). Buzz: one-shot process; if that probe fails, do not run on Buzz. No standing party for legs.
- **Model identity** — **roles** map to a local model id + **effort**. The control plane speaks roles; the adapter resolves the host string.
- **Effort** — portable `medium` / `high` / `xhigh`. Three layers, later wins: **house rules** per role, per-task override, coordinator judgment. Model stays on the role default unless a per-task override, invocation, or approved **suggestion** says otherwise. Suggestions are raised only at pin; mid-run the coordinator stays on the role default and spends effort.
- **House rules** — user-level, not repo `docs/agents/`. Per-task overrides live on the ledger row; invocation may pin a host or override a role for this run.
- **Glossary** — skill-local [`CONTEXT.md`](./CONTEXT.md). Root plugin glossary stays untouched.

## Open questions

1. ~~Discovery~~ detect; tie → fail and ask for a pin.
2. ~~Isolation~~ fresh session on every host; Buzz one-shot or don't run there.
3. ~~Model identity~~ roles.
4. ~~Glossary home~~ skill-local `CONTEXT.md`.
5. ~~Effort vocabulary~~ `medium` / `high` / `xhigh`; three layers.
6. ~~House rules location~~ user-level.
7. ~~Buzz freshness~~ one-shot process; standing party is not how legs run.
8. ~~Progress~~ receipts only. Live Herdr `n/m` is topic 2.
9. ~~Coordinator judgment~~ effort is free; model needs override, invocation, or approved suggestion.
10. ~~Missing house rules~~ fail and ask.
11. ~~Per-task override surface~~ ticket if present, else ledger, else house rules; invocation may override a role for the run.
12. ~~Suggestion timing~~ at pin only. Mid-run stays on the role default and spends effort.
13. Herdr tab `n/m` — what is counted per stage? Topic 2.
14. Herdr emoji set. Topic 2.
15. Prototype/Plan tabs when entering at implement. Topic 2.

## Sources

- This skill: `SKILL.md`, `FRESH-SESSIONS.md`
- Herdr: https://herdr.dev/docs/agent-automation/ — `agent start` needs an existing available pane; `wait` / `read` / `prompt` are the track verbs
- Buzz: https://engineering.block.xyz/blog/configuring-agents-in-buzz — agents wake on `@mention`; Model and Effort are first-class; Agent Defaults are the house rules
- Local: `herdr` and `buzz` both installed; `buzz messages send --mention` is the delegate verb
