---
name: land-pr
description: Land a stalled or forgotten pull request — confirm it isn't stale or already superseded, resolve merge conflicts, clear every open review thread and red check, then merge. Use when the user wants to get a PR across the line, revive an old/forgotten PR, or resolve a PR's merge conflicts.
---

# Land PR

Drive one pull request from open-with-problems to merged. The work splits into two questions, in order: **should** this land (is it still wanted, or has it gone **stale** — already **superseded** by a later merge?), and only then **can** it land (conflicts resolved, every thread closed, checks **green**). Answer them out of order and you risk pouring conflict-resolution effort into a PR that should just be closed.

## 1. Identify the PR

Resolve the PR from a user-supplied number, the current branch (`gh pr view --json number,headRefName,baseRefName,state,mergeable,isDraft`), or ask once if ambiguous.

Pull its full picture in one go: title and body, linked issue, base branch, age (`createdAt`), commits (`gh pr view <n> --json commits`), changed files, review threads, and check runs (`gh pr checks <n>`).

**Done when:** you hold the PR number, base branch, the linked issue (or have confirmed there is none), and the list of files it touches.

## 2. Stale check — should this land?

The gate. An old PR is guilty until proven live: a forgotten branch is often already **superseded**. Investigate before touching code:

- **Linked issue** — still open? A closed issue usually means the need was met elsewhere.
- **Superseded by base** — checkout the base branch and look for the PR's feature already present. `git log <base> --since=<PR createdAt> -- <changed files>` surfaces later merges to the same files; read them. If the base already does what this PR does, it's superseded.
- **Superseded by a sibling** — search other merged/open PRs for the same feature (`gh pr list --search`). Two branches solving one problem is the classic forgotten-PR trap.
- **Still wanted** — if the feature is ambiguous or the codebase has moved on, the change may simply be obsolete.

Return one **verdict**:

- **LAND** — still wanted, not superseded → continue to step 3.
- **CLOSE** — superseded or obsolete → stop. Recommend closing with a one-line reason and a pointer to what superseded it. Do not resolve conflicts on a dead PR.
- **ASK** — genuinely unclear whether it's still wanted → surface the evidence and ask the user before spending effort.

**Done when:** you have stated LAND, CLOSE, or ASK with the evidence that decided it. On CLOSE or ASK, the skill ends here.

## 3. Sync the branch & resolve conflicts

Bring the branch up to its base and clear conflicts so it merges clean.

- Update base (`git fetch origin`), then rebase the PR branch onto it (`git rebase origin/<base>`), or merge base in if the repo's history style or a shared/long-lived branch calls for it — match the repo's convention.
- Resolve each conflict **faithfully**: preserve the PR's intent *and* the base's newer changes. A conflict often means the base changed the very thing this PR touches — understand both sides before choosing; never blindly keep one side.
- After resolving, run the repo's build, typecheck, and tests. Conflict resolution that compiles can still be wrong — the tests are the check.

**Done when:** the branch contains every base commit, no conflict markers remain, and build + typecheck + tests are **green**.

## 4. Clear all feedback

Every open review thread and red check is a blocker. Gather them all — bot and human review comments, review summaries, and failing checks (`gh pr checks <n>`).

Verify and act on each with the same rigor as **`review-pr-comments`** (verify the issue in the real code, then fix / skip / defer with a reason) — follow that skill rather than re-deriving it. For an unattended polling loop instead of one pass, that's **`watch-pr-comments`**.

**Done when:** every review thread has a disposition (fixed with the commit, or skipped/deferred with a stated reason), no must-fix item is open, and all checks are green. Push the fixes.

## 5. Land it

Confirm the end state is genuinely mergeable: `gh pr view <n> --json mergeable,reviewDecision,state` reports mergeable, required approvals satisfied, checks green, branch current with base.

Merge per the repo's convention (squash/merge/rebase). If branch-protection requires a human approval the agent can't supply, stop and surface that the PR is ready and what's needed — that's the one blocker the skill can't clear itself.

**Done when:** the PR is merged, or you've reported it green-and-ready with the exact remaining human step.

## Report

Close with: the stale-check verdict and why; what was rebased/conflicts resolved; each feedback item and its disposition; and the final outcome (merged, or ready + blocker).
