# Dispatch

Personal-skill glossary for `/dispatch`. Not the promoted plugin glossary.

## Language

**Dispatch**:
The act of delivering a spawn intent through this session's host adapter into a fresh session.
_Avoid_: handoff, compact, launcher

**Host**:
The environment this session is sitting in — Buzz, Herdr, or a native harness.
_Avoid_: runtime, launcher, harness

**Adapter**:
The host-specific wiring that turns a spawn intent into that host's verbs.
_Avoid_: launcher

**Spawn intent**:
The portable handoff from a caller to dispatch: name, packet, role, model, effort, isolation, wait, and optional adapter, issue, branch, work, cwd.
_Avoid_: launcher recipe

**Fresh session**:
A new process, session, or subagent with no inherited turns. Required of every dispatch.
_Avoid_: standing agent, resumed context

**Pin**:
An explicit adapter named by the user or the spawn intent.
_Avoid_: launcher flag

**Space**:
The host's work container for a pane, matched or created from issue, branch, and short work description.
_Avoid_: workspace (except when quoting Herdr), tab, pane

**Rollover**:
A dispatch that continues this work in a new agent on the same host, then ends this session.
_Avoid_: handoff, compact

## Relationships

- **Dispatch** delivers one **Spawn intent** per call
- An **Adapter** belongs to one **Host**
- Every dispatch is a **Fresh session** on the same **Host**
- **Rollover** is a **Dispatch** that ends this session
- A **Pin** names the **Adapter**
- A **Space** holds the new pane on that **Host**
