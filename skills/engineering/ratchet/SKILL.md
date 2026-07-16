---
name: ratchet
description: Ratchet a repeated fix, correction, or piece of head-knowledge into infrastructure — a type, lint rule, test, CI check, hook, or a co-located CLAUDE.md/REVIEW.md rule — so the whole class of issue is prevented forever. Use when the user says "make sure this never happens again" or asks for a lint rule, check, or guardrail; when you fix an issue you recognise having fixed before; when you had to ask the user for context the repo should have held; or when the user wants an area's CLAUDE.md, REVIEW.md, or docs audited or improved.
---

# Ratchet

A **ratchet** turns one way: once it clicks, the class of issue behind it can never come back. Fixing an instance by hand spends tokens every time and misses cases; encoding the class as infrastructure fixes it forever and moves the repo toward the bar this skill serves: **zero additional context** — any agent, and any new contributor, works productively in the codebase with nothing beyond the prompt. Every question you had to ask the user, every correction received, every review rejection for a rule that lived in someone's head is evidence of a missing click.

The unit of work is the **class**, never the instance. The instance — the bug just fixed, the correction just received, the question just asked — is only evidence the class exists. A class is either a **mistake** (something that must never happen again) or a **routine** (busywork performed the same way repeatedly); mistakes ratchet into guards, routines into scripts and skills.

## The enforcement ladder

Encode the rule at the **highest rung that can fully hold it** — most deterministic first:

1. **Type** — make the invalid state unrepresentable; the compiler enforces it on every build.
2. **Lint rule** — the mistake is a recognisable code pattern.
3. **Test** — the rule is observable behaviour.
4. **CI check / script** — a repo-wide invariant a script can verify (generated files in sync, naming conventions, dependency bans), or a routine a script can perform.
5. **Hook** — must fire at a workflow moment (pre-commit, before a tool call).
6. **CLAUDE.md / REVIEW.md rule** — judgment the agent needs while authoring (CLAUDE.md) or reviewing (REVIEW.md), loaded with the directory it governs.
7. **Skill** — a process or judgment loaded on demand, when the situation matches.
8. **Code comment** — a caveat that only matters at one site; co-locate it there.

Rungs 1–5 are **mechanical**: a machine decides violations with zero judgment. Rungs 6–8 are **prose**: they steer judgment. The dividing question — *could a script decide violations without human judgment?* Yes → a mechanical rung; a mechanically checkable rule written as prose is under-encoded.

## Placement — co-locate with the code

A mechanical rung enforces itself everywhere; a prose rung only works if the agent loads it at the moment it applies. Agents load context by drilling down the file tree, so prose is placed by **altitude**:

- A rule lives in the **nearest CLAUDE.md or REVIEW.md above the code it governs** — create a nested one beside that code if none exists. The top-level file holds only genuinely repo-wide rules.
- A corner-specific rule parked at the top level taxes every session and gets skimmed past; the same rule co-located with its directory loads exactly when an agent works there.
- The same altitude test places docs and comments: knowledge sits beside the code it explains, so drilling down surfaces it without the prompter's help.

A rule at the wrong altitude — top-level but governing one corner — is itself a class: pushing it down to its directory is a click.

## Gap audit

The proactive entry point, for when the ask is "improve this area's CLAUDE.md / REVIEW.md / docs" or a session ends with the user having supplied context the repo should have held. Sweep the area for classes:

- Questions an agent had to ask a human (this session, past sessions, PR threads) — each answer is an unencoded rule.
- Corrections and review rejections whose reason is written nowhere.
- Conventions visible in the code but stated nowhere an agent would load.
- Rules at the wrong rung (prose a lint rule could hold) or wrong altitude (top-level but corner-specific).
- Directories with non-obvious constraints and no CLAUDE.md above them.

Each gap names a class; run the steps on each. Done when an agent dropped into the area could work from the repo's files alone.

## Steps

1. **Name the class.** Generalise from the instance to a one-sentence rule ("every X must Y", "Z is never used for W") with the instance as its worked example. Done when the rule would have caught the instance *and* at least one hypothetical sibling.
2. **Check it's worth a click.** Encode when the class has appeared twice, or once via a human correction or a question the repo couldn't answer — either means the knowledge already failed to transmit, which is a failure of automation. A genuine one-off gets a fix, not infrastructure.
3. **Pick the rung — then the altitude.** Apply the dividing question and take the highest rung that can fully hold the rule. On a prose rung, place it by the co-location rules above; on a mechanical rung, follow the repo's existing conventions for where that mechanism lives (lint config, CI workflow).
4. **Encode.** Write the type, lint rule, test, check, hook, or rule. On prose rungs, keep the worked example beside the rule — a rule with its example transmits better than a bare imperative.
5. **Prove the click.** Reintroduce the original instance (or a minimal reproduction) and watch the mechanism catch it, then revert. On a prose rung, verify placement instead: confirm an agent working in the governed code loads that file on its way down. Done when you can paste the failing output — or, for prose, name the load path.
6. **Engage clean.** Sweep for other live instances the new mechanism now flags and fix them — a check that is born failing gets ignored or deleted. Done when the mechanism passes on the current codebase.

Report the class, the rung and altitude, and the evidence from step 5.
