# Orchestrate PRD

Personal-skill glossary for `/orchestrate-prd`. Not the promoted plugin glossary.

## Language

**Control plane**:
This skill — the ledger, frontier, packets, receipts, failure bounds, and review loop.
_Avoid_: agnostic control plane, orchestration layer

**Host**:
The environment the coordinator is sitting in — Buzz, Herdr, or a native harness.
_Avoid_: runtime, launcher, harness

**Dispatch**:
The skill that delivers a spawn intent through this session's host adapter into a fresh session.
_Avoid_: handoff, compact, launcher

**Spawn intent**:
The portable handoff from the control plane to dispatch: name, packet, role, model, effort, isolation, wait.
_Avoid_: launcher recipe

**Role**:
The kind of relay leg a model and effort are assigned to — implement, review, verify, recover, probe, or rollover.
_Avoid_: agent name, standing agent

**Fresh session**:
A new process, session, or subagent with no inherited turns. Required of every relay leg on every host.
_Avoid_: standing agent, resumed context

**Effort**:
The portable thinking level on a spawn intent — `medium`, `high`, or `xhigh`.
_Avoid_: reasoning level, thinking budget

**House rules**:
The user-level map from each role to a default model and effort.
_Avoid_: agent defaults, repo config

**Pin**:
An explicit host choice the user supplies when detect cannot tell which host is live.
_Avoid_: launcher flag

**Suggestion**:
A coordinator-proposed model override, raised only while pinning the control plane, that does not apply until the user approves it.
_Avoid_: silent override, model shopping, mid-run ask

## Relationships

- The **Control plane** emits a **Spawn intent**
- **Dispatch** delivers each **Spawn intent** on this **Host**
- A **Spawn intent** carries one **Role**, one model, and one **Effort**
- Every relay leg is a **Fresh session**
- **House rules** supply the default model and **Effort** for a **Role**
- A **Pin** names a **Host** when detect cannot
- A **Suggestion** becomes a per-task override only after approval
