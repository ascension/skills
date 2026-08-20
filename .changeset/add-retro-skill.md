---
"mattpocock-skills": minor
---

Add **`retro`**, a user-invoked after-ship look-back that finds every place an agent had to stop, ranks it on a four-rung ladder, and encodes it so the next run works through the same constraint without interruption.

The unit is a **stop**: a question the agent asked, a correction it received, a permission pause, context the repo should have held, or a human review comment on the PR. Each stop is generalised to a class and placed on the highest rung that can hold it:

1. Eliminate it through architecture or data structures.
2. Catch it in CI with a lint rule or test.
3. Steer the next agent with a skill or rule.
4. Leave it to human review, and capture that PR feedback so it can be promoted into 1-3.

Rungs 2 and 3 (and types on the way to 1) go through `/ratchet`. Rung 1 is a shape change, not a new rule. Rung 4 is a source, not a destination. The retro is the reply; encodings land in the repo. There is no retro file.

Wired as a promoted Engineering skill. Plugin entry, top-level + Engineering READMEs under **User-invoked**, a docs page at `docs/engineering/retro.md`, and a Codebase health route in `ask-matt` as the after-ship look-back that feeds `ratchet`.
