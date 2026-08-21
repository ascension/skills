## What it does

`retro` looks back at a [session](https://www.aihero.dev/ai-coding-dictionary/session) that has shipped or opened a PR and names every **stop**: a moment the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) could not continue without a human. A question it asked. A correction it received. Context the repo should have held. A human review comment on the PR. It then encodes each stop at the highest rung that can hold it, so the next session runs through the same constraint without interruption.

It never treats the incident in front of it as the unit of work. The incident is evidence that a class of stop exists. A retro that only reports, and leaves the class unencoded, has not done the job.

## When to reach for it

You invoke this by typing `/retro`. The agent won't reach for it on its own.

Reach for it when a session has opened a PR, or just shipped, and you want the next agent to hit fewer walls. Mid-build is too early. Encoding one class you already have in hand is [ratchet](https://aihero.dev/skills-ratchet).

| Your situation | Reach for |
| --- | --- |
| The session (or its PR) just landed, and you want the stops named and encoded | `retro` |
| You already know the class ("every X must Y") and want it locked out | [ratchet](https://aihero.dev/skills-ratchet) |
| The whole codebase has drifted, not one session | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) |
| You want the diff reviewed against standards and the spec | [code-review](https://aihero.dev/skills-code-review) |

## The ladder

Every stop is ranked, highest first, and encoded at the first rung that fully holds it.

| Rung | What it does |
| --- | --- |
| Architecture or data structures | Makes the stop impossible. The invalid shape cannot be represented. |
| Lint rule or test | CI catches the class on every run. |
| [Skill](https://www.aihero.dev/ai-coding-dictionary/skill) or rule | Steers the next agent's judgment, co-located with the code it governs. |
| Human review | Humans caught it this time. The comment is captured so the next retro can promote it. |

Human review is a source, not a destination. A PR comment that can live as a type, a test, or a rule gets promoted this pass rather than filed under "we'll catch it next time". Architecture sits above a lint. If a better shape would delete the class, a new rule that leaves the shape in place has under-encoded it.

Rungs that are a lint, a test, a type, or a co-located rule go through [ratchet](https://aihero.dev/skills-ratchet). The retro finds and ranks. `ratchet` clicks.

## Stops come from the session and the PR

The window is this session plus the PR if one exists. Human review comments are first-class stops, the same as a question asked in chat. Each stop keeps a one-line evidence quote so you can see why it made the list. Review comments authored by people count.

The retro is the reply, in three sections: **Stops**, **Encoded**, **Still stopping**. Encodings land in the repo, on this branch when the change is local to the work, as a follow-up when the shape change is bigger than this PR. There is no retro file. A markdown note about a lint that should exist is the class of failure this skill exists to prevent.

## Common questions

**How is this different from ratchet?**

`ratchet` encodes one class you already have. `retro` looks back across a session and a PR to *find* the classes, including the ones that only showed up as a human comment. When a stop belongs on a lint, a test, a type, or a co-located rule, `retro` hands that class to `ratchet`. Reach for `ratchet` alone when you already know the sentence ("every X must Y") and want it clicked now.

**Does it change the repo, or just write a report?**

It encodes. Reversible work (a lint, a test, a local shape change, a co-located rule) proceeds in the same pass. A retro that only lists stops has failed. The one thing it will not do is write a `RETRO.md` and call that the fix.

**Do I run it after every PR?**

Run it when the session had to stop. A clean PR with no questions, no corrections, and no human review comments produces an empty list, which is a successful retro and a cheap one. Skipping it after a noisy session is how the same question gets asked next time.

## It's working if

- The output names classes ("every X must Y"), not one-off incidents.
- Architecture won when a shape change would have deleted the stop, instead of a new lint leaving the shape in place.
- Human PR review comments appear on the list, not only chat corrections.
- **Encoded** names files that now exist, or **Still stopping** names the leftover classes and their next promotion.
- The next session does not hit the same stop.

## Where it fits

`retro` is **periodic maintenance**: the after-ship look-back on [Codebase health](https://aihero.dev/skills-ask-matt), not a chain step. It feeds [ratchet](https://aihero.dev/skills-ratchet), which encodes a class, and sits next to [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture), which surveys the whole codebase rather than one session. Repeated [code-review](https://aihero.dev/skills-code-review) findings and human PR comments are the stops it is built to mine. When you're unsure which skill fits, [ask-matt](https://aihero.dev/skills-ask-matt) routes you.
