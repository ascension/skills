---
name: dispatch
description: Dispatch a spawn intent through a named or detected host adapter into a fresh session, placing it in a matching space. Use when spawning an agent on Herdr, Buzz, native, or Pi; when the user or caller names the adapter; when a pane must land in a space matched by issue, branch, or short work description; when a session approaches the smart zone (~150k) and must continue on a new agent; or when orchestrate-prd needs a probe, implement, review, verify, recover, or coordinator rollover delivered.
---

# Dispatch

Deliver one **spawn intent** through a **host** **adapter** into a **fresh session**, in the **space** that matches the work. Terms: [CONTEXT.md](CONTEXT.md).

`handoff` writes a file that travels. `/compact` compresses this window. This skill spawns.

## 1. Host

If the user or the spawn intent names an **adapter**, that is the **pin**. Load it. Read [HOSTS.md](HOSTS.md) for the adapter file.

If no adapter is named, detect the live host. Zero matches: stop and ask for a pin.

An unnamed dispatch stays on the host this session is already on.

## 2. Space

Read [SPACES.md](SPACES.md). Resolve a space from the intent's issue, branch, and work against the host's active spaces. One match: use it. None: create one. The adapter performs list and create.

## 3. Intent

The caller supplies a spawn intent (for `orchestrate-prd`, its `FRESH-SESSIONS.md`). Required: `name`, `packet`, `role`, `model`, `effort`, `isolation: fresh`, `wait`. Optional: `adapter`, `issue`, `branch`, `work`, `cwd`.

`role` is `implement`, `review`, `verify`, `recover`, `probe`, or `rollover`.

**Rollover** is coordinator continuation at the [smart zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone) (~150k) or any phase boundary where this session must stop and a new one must continue the same work. After spawn succeeds, this session ends.

Approaching the smart zone with work still in flight: treat the intent as rollover on the resolved host.

## 4. Spawn

Read only the adapter for the resolved host. Pass the resolved space. Spawn, wait, and read as that file says. Resolve `model` and `effort` against the host catalog. A name the adapter cannot resolve is a capability failure — stop or ask; the adapter does not pick a substitute model.

A receipt comes back to the caller. For rollover, the receipt is that the new agent started on the resolved host with the packet; then stop.

Dispatch is complete when the caller has a receipt, or when a rollover agent is live on the resolved host and this session has ended.
