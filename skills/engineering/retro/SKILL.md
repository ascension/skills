---
name: retro
description: After a session that shipped or opened a PR, find every place the agent had to stop and encode it so the next run doesn't.
disable-model-invocation: true
---

A **stop** is any moment the agent could not continue without a human. A question it asked. A correction it received. A permission pause. Context the repo should have held. A human review comment on the PR. The retro names every stop in this session and its PR, picks the highest rung that can hold it, and encodes it so the next agent works through that constraint without interruption.

The unit is the **class** of stop, never the incident. The incident is evidence the class exists.

## The ladder

Rank every stop, highest first. Take the first rung that fully holds it.

In order of value:

1. categorically eliminate the problem through better architecture or choice of data structures
2. turn it into a lint rule or test so CI catches it
3. turn it into a skill or rule
4. have humans review the code to catch it - capture the feedback from PRs so we can learn and turn those into more things like 1-3. If humans are providing feedback we can analyze that feedback and turn it into more rules/skills to help guide future agents/runs.

Rung 4 is a source, not a destination. A human comment that can live on 1-3 gets promoted this pass.

`ratchet` encodes rungs 2 and 3, and types on the way to 1. Call the Skill tool with "ratchet" once per class that belongs there. Architecture on rung 1 is a shape change. A lint or a CLAUDE.md line that leaves the shape in place has under-encoded it.

## Steps

1. **Pin the window.** This session, plus the PR if one exists (the user named it, `gh pr view` on the current branch, or the PR just opened). Done when you can name the session and the PR URL, or "no PR".

2. **Collect stops.** Walk the session for questions, corrections, pauses, missing-repo context, and work that had to be redone. Pull human PR review comments and issue-thread replies. Keep comments from humans. Each item is one stop with a one-line evidence quote. Done when every human intervention in the window is on the list, including "none".

3. **Rank.** For each stop, name the class ("every X must Y", or the shape that must replace Z) and the highest rung that fully holds it. A stop you would still need a human for, after the best encoding, stays on 4 as a promotion candidate. Done when every stop has a class and a rung.

4. **Write the retro.** Reply in this shape, nothing else above it:

   ## Stops
   One bullet per class: evidence, rung, encoding (or why it stays on 4).

   ## Encoded
   What you changed this pass.

   ## Still stopping
   What still needs a human, and the next promotion.

   Skip empty sections.

5. **Encode.** Reversible encodings proceed without asking. On this branch when the stop was in this work and the change is local. A follow-up when the shape change is bigger than this PR. For each ranked class:

   - Rung 1: change the architecture or data structure so the stop cannot occur. Local and reversible: do it. Otherwise name the shape.
   - Rung 2 or 3: Call the Skill tool with "ratchet" for that class.
   - Rung 4: keep the human comment in **Still stopping** as a promotion candidate. The reply is the retro. Encodings land in the repo.

   Done when Encoded names the actual files, or Still stopping names the leftover classes.

A retro that only reports, and encodes nothing it could have encoded, failed.
