---
name: land-pr
description: "Land a pull request through a quality-gated polling loop: confirm it is still wanted, keep its branch current, run /code-review against the originating spec, resolve verified feedback and failing checks, and merge only when that reviewed head is fully green. Use when the user asks to get a PR across the line, revive an old PR, or monitor a newly opened PR through merge."
---

# Land PR

Run a **landing loop** until one pull request is merged or reaches a blocker only a human can clear. Automation removes waiting and repetition; every code change still earns its way through review and verification.

## 1. Pin the pull request

Resolve the PR from a supplied number or the current branch. Capture its URL, head and base branches, head SHA, title/body, linked issue, age, commits, changed files, draft state, reviews, unresolved threads, and checks.

Read the repository's contribution rules, branch protection, required commands, merge method, and reviewer conventions. Treat invoking this skill as authorization to update and merge this PR, not to widen its feature scope.

**Complete when:** the PR, its intended outcome, its exact head SHA, and its landing requirements are known.

## 2. Decide whether it should land

For newly opened work, the user's current request establishes intent. For an old or forgotten PR, investigate before editing:

- Is its linked issue still open and its outcome still wanted?
- Does the base branch already contain the behavior? Inspect later commits touching the same files.
- Has another open or merged PR superseded it?

Choose one verdict:

- **LAND** — wanted and not superseded; continue.
- **CLOSE** — obsolete or superseded; stop with the evidence and proposed closing note. Close only when authorized.
- **ASK** — product intent is genuinely ambiguous; surface the decision and stop before changing code.

**Complete when:** LAND is supported by evidence, or the run ends with CLOSE/ASK.

## 3. Establish the quality baseline

Fetch the remote and update the PR branch from its base using the repository's history convention. Rebase only when rewriting this branch is safe; use `--force-with-lease` for a rewritten remote branch. For conflicts, invoke `/resolving-merge-conflicts` so both sides' intent is preserved.

Run the repository's required local checks. Then invoke `/code-review` against the base branch, passing the linked issue or PRD as the spec source, and resolve every must-fix Standards or Spec finding. Keep changes inside the PR's intended outcome.

`/code-review` is the required review for landing. Consume external bot findings when they exist. A missing, skipped, or usage-limited bot review — Cursor Bugbot, Pi Code Review, or similar — is not part of this baseline.

Commit coherent fixes and push only after the affected checks pass. Record the pushed head SHA; all later review and merge evidence belongs to that SHA.

**Complete when:** the remote head contains the current base, `/code-review` has no must-fix finding on that SHA, and required local checks pass at the recorded SHA.

## 4. Run the landing loop

Repeat **OBSERVE → CLASSIFY → ACT → POLL**. Default to a 60-second poll while checks or reviews are pending, adjusting only for provider limits. Elapsed time and a quiet poll are never completion criteria.

### OBSERVE

Fetch fresh PR state for the current head SHA: mergeability, base freshness, check runs and logs, review decision, new reviews/comments, and every unresolved thread. Paginate; a partial comment list is not evidence of a clean PR.

### CLASSIFY

Put every blocker in exactly one state:

- **FIX** — draft state after the baseline, base drift, a reproducible conflict, a failing check, or a valid finding introduced by this PR.
- **RESPOND** — false positive, already fixed item, or deliberate tradeoff that needs an evidence-backed reply.
- **WAIT** — an in-progress required check or a requested human review/approval with no agent action available. A missing, skipped, or usage-limited external bot review is not WAIT.
- **ESCALATE** — missing permission, no eligible reviewer, persistent external outage, or product decision the agent cannot safely make.
- **TERMINAL** — merged, closed, or proven superseded.

### ACT

Work every FIX and RESPOND item before waiting:

- Verify each comment against the current diff. Fix confirmed issues minimally; explain false positives or scope decisions with code/test evidence. Consume bot findings that actually arrived; a bot that never started is not a blocker.
- If the quality baseline passed while the PR is draft, mark it ready for review and observe again.
- If the base advanced, return to step 3 to sync it and re-establish the quality baseline.
- Inspect failing-check logs and reproduce failures locally. Retry only when evidence indicates infrastructure or flakiness; an unexplained red check remains a blocker.
- After any code change, rerun affected local checks and `/code-review`, commit coherently, push, and record the new head SHA. That push invalidates prior check and review evidence; begin OBSERVE again.
- Reply with the fixing commit SHA. Resolve a thread only after its fix is pushed or its reviewer accepts the disposition; leave genuine disagreement visible.

### POLL

TERMINAL ends the loop: verify and report an external merge, closure, or superseding change without applying the merge gate. If only WAIT items remain, wait and observe again. Request required reviewers through repository conventions when none are assigned. ESCALATE only when continued polling cannot change the state; report the exact human action needed. If the runtime must end, leave a checkpoint containing the PR URL, head SHA, completed evidence, blockers, and next poll action rather than declaring success.

**Complete when:** a fresh observation of one unchanged head SHA satisfies the merge gate, or a TERMINAL outcome is verified and reported.

## 5. Pass the merge gate

Merge only when all of these are true for the same head SHA:

- The PR is open, ready for review, mergeable, and current with base.
- Required human approvals are present, no changes are requested, and no actionable thread is unresolved.
- Every required repository check passes and no other check is red; skipped or cancelled checks have an evidence-backed disposition consistent with repository policy. A missing, skipped, or usage-limited external bot review is not a required check.
- Required local checks pass and `/code-review` against the base has no must-fix finding on this SHA. That two-axis report is the required review. If a repo readiness script fails only because Cursor Bugbot or Pi Code Review is absent, treat the `/code-review` report as the substitute and continue.

Immediately re-fetch the PR, head SHA, base state, reviews, threads, and checks. Any change returns to the landing loop. Otherwise merge with the repository's configured method, then verify the server reports the PR merged.

**Complete when:** the PR is verified merged. A human-only blocker is a checkpoint, not a successful landing.

## Report

Report the LAND/CLOSE/ASK evidence, branch updates, `/code-review` Standards and Spec dispositions, local checks, each external feedback disposition, polling outcome, and final merge commit or exact human blocker.
