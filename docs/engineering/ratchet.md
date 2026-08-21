Quickstart:

```bash
npx skills add mattpocock/skills --skill=ratchet
```

```bash
npx skills update ratchet
```

[Source](https://github.com/mattpocock/skills/tree/main/skills/engineering/ratchet)

## What it does

`ratchet` converts a repeated fix, correction, or piece of head-knowledge into infrastructure — a type, lint rule, test, CI check, hook, or a co-located CLAUDE.md/REVIEW.md rule — so the whole class of issue is prevented forever instead of re-fixed every time it appears.

It never stops at the instance in front of it, and it holds the repo to one bar: **zero additional context** — any agent or new contributor should be able to work productively from the repo's files alone, with nothing beyond the prompt. Every question an agent had to ask a human is treated as a missing piece of infrastructure, not a normal part of the workflow.

## When to reach for it

Type `/ratchet`, or the agent reaches for it automatically when a task fits — it fires on "make sure this never happens again", on requests for a lint rule, check, or guardrail, when the agent recognises it is fixing an issue it has fixed before, or when you ask it to audit or improve an area's CLAUDE.md, REVIEW.md, or docs.

Reach for it whenever the same mistake or busywork shows up twice, when a review rejection reveals a rule that lived only in someone's head, or after a session where you had to supply context the repo should have held. For diagnosing *why* something is broken in the first place, use [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) instead — `ratchet` starts after the fix, when the question is how to make the class impossible.

## The enforcement ladder

The skill's core move is picking the right rung on a ladder ranked by determinism: type → lint rule → test → CI check → hook → CLAUDE.md/REVIEW.md rule → skill → code comment. A rule is encoded at the highest rung that can fully hold it. The dividing question is whether a script could decide violations without human judgment: if yes, the rule belongs on a mechanical rung, and leaving it as prose is under-encoding.

## Co-location — rules live beside their code

A mechanical rung enforces itself everywhere; a prose rule only works if the agent loads it at the moment it applies. So `ratchet` places prose by **altitude**: a rule goes in the nearest CLAUDE.md or REVIEW.md above the code it governs — creating a nested one if none exists — and the top-level file keeps only genuinely repo-wide rules. Agents drilling down the file tree then surface each rule exactly where it matters, instead of skimming past a bloated top-level dump. A rule parked at the wrong altitude is itself a class worth fixing.

## The gap audit

Beyond reacting to a repeated fix, `ratchet` has a proactive mode: sweep an area for knowledge that still lives in heads — questions agents had to ask, corrections written nowhere, conventions visible in code but stated in no file an agent would load, directories with non-obvious constraints and no CLAUDE.md. Each gap becomes a class, encoded and proven like any other. The audit is done when an agent dropped into the area could work from the repo's files alone.

## It's working if

- The output names a class ("every X must Y"), never just a patched instance.
- The chosen rung is mechanical whenever a script could decide violations — prose rungs only carry judgment.
- Prose rules land in the nearest CLAUDE.md/REVIEW.md above the code they govern, not the top-level file.
- It pastes the failing output from reintroducing the original mistake (or names the load path for a prose rule).
- Existing violations are fixed in the same pass, so the new check passes on the current codebase.

## Where it fits

`ratchet` is a reach-for-it-anytime standalone that compounds: every click makes the codebase safer and more legible for the next agent and the next contributor. It pairs naturally with [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) — after a hard bug is fixed, `ratchet` asks whether the whole class can be locked out — with [code-review](https://aihero.dev/skills-code-review), whose repeated findings are exactly the classes worth encoding, and with [retro](https://aihero.dev/skills-retro), which looks back across a session and a PR to find the classes `ratchet` then clicks. When you're unsure which skill fits, [ask-matt](https://aihero.dev/skills-ask-matt) routes you.
