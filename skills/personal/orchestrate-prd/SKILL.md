---
name: orchestrate-prd
description: Coordinate an approved PRD ticket graph through isolated implementation sessions, PR publication, and a bounded review/fix loop.
argument-hint: "<PRD issue, URL, or path> [base ref]"
disable-model-invocation: true
---

# PRD Relay

This is the AFK continuation of Ask Matt's multi-session flow after `/to-tickets`. A **relay leg** is one task in one fresh process or agent context. The coordinator keeps the ledger; each worker receives one small task packet.

This skill owns orchestration only. The PRD and tickets own requirements, `/implement` owns ticket work, and `/code-review` owns the Standards and Spec review axes.

## 1. Pin the control plane

1. Read `docs/agents/issue-tracker.md`. If it is absent, stop and ask the user to run `/setup-matt-pocock-skills`.
2. Resolve the PRD, all linked descendant tickets and comments, the integration branch, target base, immutable starting base SHA, and the repository's integration-verification commands. Require a clean integration worktree; isolate or stop on unrelated pre-existing changes.
3. Map every in-scope PRD requirement to an approved ticket, including user-supplied missing or incomplete work. If publishing new tickets still needs approval, stop at `/to-tickets`; resume only with an approved dependency graph.
4. Create an orchestration ledger in the OS temporary directory, outside the PR diff. For each task record its source ref, concise acceptance criteria, blockers, status, starting SHA, commits, checks, and criterion evidence. Store raw worker logs beside the ledger, not in coordinator context.
5. Create a review-spec manifest beside the ledger. It is an index of the parent PRD, every in-scope ticket and acceptance criterion, accepted scope changes, and out-of-scope decisions. Use source links rather than copied issue bodies.

The control plane is pinned when the base and starting SHA, integration commands, and clean worktree are verified; the ledger and review-spec manifest are populated; every requirement has a task with explicit blocking edges; and the initial frontier is known.

## 2. Open the relay

Read the capability gate, selected launcher, and current packet section in [FRESH-SESSIONS.md](FRESH-SESSIONS.md).

Dispatch every implementation, recovery, verification, review, and fix relay leg under that reference's capability gate.

Pass pointers plus the active task's criteria; let the worker fetch primary sources. Keep only its compact receipt in coordinator context. The ledger, tracker, and Git history are sufficient to roll the coordinator into a fresh session at any phase boundary.

The relay is open when the read-only capability probe passes without changing HEAD or the worktree, and its exact runner and model are recorded.

## 3. Run implementation legs

Work the dependency frontier with one mutating worker at a time on the integration branch.

For each ready ticket:

1. Record `TICKET_BASE_SHA`, inspect the installed `/implement` and `/code-review` ordering as directed by `FRESH-SESSIONS.md`, then dispatch a fresh session that explicitly enters `/implement` with the ticket ref and implementation packet.
2. Accept the receipt only after independently verifying that observed integration `HEAD` equals reported `HEAD`, `TICKET_BASE_SHA` is its ancestor, every commit and diff in `TICKET_BASE_SHA..HEAD` is reported and task-scoped, the worktree is clean, every criterion has evidence, and required checks passed.
3. Record newly discovered scope as a separate ticket with blocking edges. Keep the active worker on its original acceptance criteria.
4. Treat runner failures as recovery tasks and dispatch them fresh with the recovery packet.

After every ledger task is evidenced and the integration branch is clean, run the pinned integration commands with the verification packet. Turn a code failure into a fresh `/implement` fix leg and re-run verification. Implementation is complete only when the final verification leg is green at the expected HEAD.

## 4. Publish the PR

Push the integration branch normally and create or update one draft PR against the pinned base. Link the parent PRD and every ticket, use the tracker's closing syntax for implementation tickets, preserve the parent PRD, and include verification evidence.

Publication is complete when the PR exists at the expected base and head. On failure, record the exact branch, commit, credential or remote error, and a PR-ready title and body, then stop as `blocked`.

## 5. Close the review loop

Pin the PR merge-base SHA once. A cycle is one valid aggregate `/code-review` report plus remediation of its actionable findings. The initial aggregate review is cycle 1; the maximum is five.

For each cycle:

1. Record `EXPECTED_HEAD`, then start a fresh review session. Explicitly invoke `/code-review <PINNED_BASE_SHA>` and pass the PR plus the review-spec manifest as the composite spec source. Prefer a different approved model family from implementation when available.
2. Accept only a valid two-axis report whose `REVIEWED HEAD` equals `EXPECTED_HEAD` and whose source receipt accounts for every manifest entry. Preserve Standards and Spec as separate axes. Give each finding a ledger row and close it only as `fixed` or `non-actionable`, with evidence.
3. If no actionable findings remain and integration verification passes, mark the draft PR ready, record `review-clean`, and stop.
4. Convert coherent findings into fix tasks. Run every fix through `/implement` in a fresh session, verify its commits, run a fresh integration-verification leg, and push the same PR. When the cycle number is below 5, start the next cycle from the original pinned SHA.

A failed dispatch with no valid two-axis report does not consume a cycle. If cycle 5 finds issues, fix them, require green integration verification, push, then stop as `cap-reached/unconfirmed`: a sixth review would be required to prove the final fixes review-clean.

## Failure bounds

Each relay leg gets one initial attempt and at most one fresh recovery attempt. Reconcile and verify any partial Git state before retrying. A verification failure caused by code becomes one bounded fix task followed by one new verification leg; another red result is `blocked`. Stop as `blocked` on a second failure, a non-recoverable capability or credential failure, uncertain Git state, or a required user decision. A review dispatch retry does not consume a cycle; a second invalid report is `blocked`.

## Completion report

Report the PR URL, base and head SHAs, completed and blocked tasks, integration checks, review-cycle count, Standards and Spec dispositions, and exactly one result: `review-clean`, `cap-reached/unconfirmed`, or `blocked`. Only `review-clean` is successful completion.
