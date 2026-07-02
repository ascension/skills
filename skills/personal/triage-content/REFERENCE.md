# Triage Content - Reference

Detailed rubric, surface map, search recipes, and examples. Read this when a signal is non-trivial or the SKILL.md verdict block needs grounding.

## Surface map - where an idea can land

| Surface | Path | Adopt here when the idea is... |
|---|---|---|
| Application code | `apps/*` (api, web, console, hive-link, pulse, worker, mobile, ...) | a concrete technique/library/bugfix for a running app |
| Shared package | `packages/*` (types, shared-utils, bambu-node, ui-primitives, ...) | reusable across workspaces - put it in the canonical layer, not one app |
| Domain language | `CONTEXT.md` | a naming/mental-model improvement for an existing concept |
| Architecture decision | `docs/adr/000X-*.md` | load-bearing; changes a trade-off or introduces a durable constraint |
| Engineering standards | `docs/ENGINEERING_STANDARDS.md` | a general depth/legibility heuristic worth codifying |
| Design system | `DESIGN.md` | visual/interaction/brand - never adopt UI ideas without checking this |
| Capabilities/protocol | `CAPABILITIES_SYSTEM.md` | affects hive-link protocol versioning |
| Workspace guide | `apps/*/CLAUDE.md`, `packages/*/CLAUDE.md` | a local gotcha, command, or convention |
| Dev/ops workflow | `DEVELOPMENT.md`, CI config, `COMMIT_CONVENTIONS.md` | improves the build/test/deploy/review loop |
| Not yet actionable | GH issue or `.context/` note | promising but no home yet - capture, don't force-fit |

## Search recipes - prove the surface exists

Before claiming a signal "lands in X", verify X:

- **Concept / where it lives** - prefer the graphify graph if `graphify-out/graph.json` exists:
  `graphify query "<concept from the signal>"` or `graphify explain "<symbol>"`. Falls back to Grep/Explore.
- **Prior art in decisions** - `ls docs/adr` then grep titles; an existing ADR may already own this call.
- **Domain term already named?** - grep `CONTEXT.md` before proposing new vocabulary.
- **Recurring pain it might solve** - grep CLAUDE.md files and open GH issues (`gh issue list --search "<keyword>"`).
- **Boundary check** - if the idea touches uploads, credentials, or hive-link, re-read `docs/adr/0011-hivelink-service-trust-boundary.md` and the CLAUDE.md cross-cutting rules.

For broad "does this concept exist anywhere" sweeps, launch an `Explore` agent rather than reading files one by one.

## The 7 lenses - what "High" actually means

A lens only scores above **None** if you can point at a specific Hive artifact. Vague resonance ("this is good hygiene") is None.

- **Depth** - would let an existing module absorb behavior behind its current interface (Ousterhout deep-module sense). High = you can name the module and the complexity it would hide.
- **Solves a problem** - there is a live bug, ADR "considered option", TODO, or documented gotcha it addresses. High = you can link the issue/line.
- **Improves a workflow** - measurably faster/safer dev loop, CI, deploy, review, or operator/Eve flow. High = names the current friction.
- **Reduces complexity** - passes the deletion test: adopting it removes a wrapper/layer/conditional. High = names what gets deleted.
- **Adds clarity** - improves naming, docs, or the shared mental model. High = names the confusing term/doc it replaces (cross-check `CONTEXT.md`).
- **Refines a pattern** - sharpens a convention already used in >=2 workspaces. High = names the pattern and where it's used.
- **Improves architecture** - changes a load-bearing trade-off; worth an ADR. High = you can draft the ADR title.

Roll-up -> verdict:
- **Adopt** - >=1 High lens with a named surface, no boundary violation, low/known cost.
- **Adapt** - useful core but needs translation to Hive's stack/domain, or trims a risky part.
- **Watch** - real but premature (no home, immature, or speculative). Capture in `.context/` or an issue.
- **Discard** - no surface, duplicate of an existing ADR, or violates a hard rule with no safe adaptation.

## Hard-rule tripwires (auto-downgrade)

If the signal implies any of these, it cannot be a clean **Adopt** - call it out explicitly:

- Adds Supabase/service credentials or direct DB/storage writes to **hive-link** runtime (ADR-0011).
- Pushes a file past the **~1000-line ceiling** or bolts nullable modes onto an unrelated flow.
- Bypasses `getOrganizationIdOrThrow` for the org-id resolution.
- Adds a required hive-link feature without `protocolVersion` gating.
- A UI/visual change that contradicts `DESIGN.md`.

## Worked examples

**Example 1 - a tweet about a Zod v4 discriminated-union perf trick**
- IDEA: faster union parsing via a keyed lookup instead of sequential tries.
- LANDS IN: `packages/bambu-types` (Zod schemas) and hot MQTT parse paths in `packages/bambu-node`.
- Lenses: Reduces complexity (Some - collapses a try-chain), Improves workflow (Some - parse throughput). Depth None.
- VERDICT: **Adapt** - validate against real Bambu payloads first. NEXT STEP: spike PR in bambu-types + benchmark.

**Example 2 - a blog post: "stop putting business logic in socket handlers"**
- IDEA: extract a domain service behind the transport layer.
- LANDS IN: `apps/pulse` (Socket.IO) - matches an in-flight direction (see PulseClient split).
- Lenses: Depth (High - service seam), Refines pattern (High - already splitting Pulse), Architecture (High).
- VERDICT: **Adopt** as reinforcement; NEXT STEP: note in the pulse-client-split plan / consider an ADR if it changes the seam.

**Example 3 - a slick animated dashboard component on X**
- IDEA: pretty real-time chart micro-interactions.
- Tripwire: UI - must reconcile with `DESIGN.md` first.
- Lenses: mostly aesthetic; no code surface yet.
- VERDICT: **Watch** - file screenshot + link in `.context/`; revisit during a design-review pass.

**Example 4 - a repo offering a "universal printer telemetry key-value store" for hive-link**
- Tripwire: would give hive-link a direct data store -> violates ADR-0011 trust boundary.
- VERDICT: **Discard** (or **Adapt** only as an API/Pulse-owned contract). Say why explicitly.

## Output discipline

- Lead with the `SIGNAL/IDEA/RELEVANCE/...` block from SKILL.md; prose is secondary.
- Every fired lens cites a path or symbol. No path -> downgrade.
- If prior art already decided it, cite the ADR and stop - don't re-argue a settled call.
