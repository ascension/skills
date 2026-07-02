---
name: triage-content
description: Evaluate an external link, tweet, article, or repo against the Hive codebase, docs, and ADRs, then decide whether it's worth adopting and where it lands. Use when the user pastes a URL/tweet/thread and asks "is this useful?", "should we use this?", "does this apply to us?", or wants a relevance/adoption verdict for external content.
---

# Triage Content

Turn an external signal (link, tweet, thread, repo, doc) into a grounded **adoption verdict** for Hive: does it add depth, solve a real problem, improve a workflow, reduce complexity, add clarity, refine a pattern, or improve architecture - and if so, exactly where does it land?

Bias toward a decision, not a summary. The user has an inbox of links; your job is to say **adopt / adapt / watch / discard** with a concrete next step and evidence.

## Quick start

Input: a URL, tweet, or pasted text. Then:

1. **Ingest** - fetch the URL with the available web/browser fetch tool (extract the core claim/technique, not marketing fluff). If the user pasted raw tweet/thread text, use that directly.
2. **Distill** - one paragraph: what is the actual idea, and what would adopting it require?
3. **Map to Hive** - find the surface(s) it touches (workspace, doc, ADR, domain term). See [REFERENCE.md](REFERENCE.md) for the surface map and how to search.
4. **Score** - run the 7-lens rubric below; keep only lenses where you can name a *specific* file/module/decision it affects.
5. **Verdict** - output the decision block (below) with a concrete next action.

## The 7 lenses

Score each **High / Some / None**, and for anything above None cite the exact Hive surface:

- **Depth** - lets a module hide more behind the same interface (see `docs/ENGINEERING_STANDARDS.md`)
- **Solves a problem** - maps to a known bug/ADR/issue/TODO or a recurring gotcha in a CLAUDE.md
- **Improves a workflow** - dev loop, CI, deploy, review, or an operator/Eve flow
- **Reduces complexity** - deletes a wrapper, collapses a layer, retires an ad-hoc conditional
- **Adds clarity** - better naming (check `CONTEXT.md` domain language), docs, or mental model
- **Refines a pattern** - sharpens an existing convention already used across workspaces
- **Improves architecture** - load-bearing enough to justify a new ADR under `docs/adr/`

## Verdict (always output this block)

```text
SIGNAL: <one-line what it is> - <source url/handle>
IDEA: <the transferable idea in 1-2 sentences>
RELEVANCE: <Adopt | Adapt | Watch | Discard>
WHY: <lenses that fired, each with the specific Hive surface it hits>
LANDS IN: <workspace / doc / ADR-000X / CONTEXT term - or "nowhere yet">
NEXT STEP: <PR to <path> | new ADR "<title>" | note in <doc> | GH issue | file in .context/ | none>
RISK/COST: <migration, coupling, boundary violation, maintenance - or "low">
```

## Rules

- **No verdict without a surface.** If you can't name a file/module/ADR it touches, it's `Watch` or `Discard`, not `Adopt`.
- **Respect boundaries.** Flag anything that would violate a hard rule (HiveLink<->Supabase trust boundary, ~1000-line ceiling, org-id pattern, capabilities gating). A clever idea that breaks a boundary is `Discard` or `Adapt`, never `Adopt`.
- **Check for prior art first** - an existing ADR may already have decided this; say so instead of re-litigating (`docs/adr/`).
- **Adapt != Adopt.** Most useful signals need translation to Hive's stack/domain - say what changes.
- Deep dives, the surface map, search recipes, and worked examples live in [REFERENCE.md](REFERENCE.md).
