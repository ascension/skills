# Fresh relay sessions

Use this reference when constructing a spawn intent and its packet. Read the capability gate, the spawn intent, and only the packet section for the current leg. Call the Skill tool with "dispatch" to deliver the intent. Re-check installed model IDs and command syntax from local `--help` or `--list-models`; the environment is their source of truth.

`$name` is Codex skill invocation. It is the same skill as `/name` on Claude or Cursor and `/skill:name` on Pi. `$code-review` maps to `/code-review`; `$implement` maps to `/implement`; `$tdd` maps to `/tdd`. Use the prefix the selected runner requires.

## Capability gate

Every relay leg must have:

- A **fresh session**: a new process, session ID, or subagent with no inherited turns and no resume, continue, or fork behavior.
- A model the adapter resolved from **house rules** or an approved override, present in the host catalog.
- The repository worktree and its `AGENTS.md` or `CLAUDE.md` instructions.
- Read access to the ledger and manifest directory outside the worktree.
- Explicit entry into every user-invoked skill governing that leg. Model discovery is insufficient for skills with `disable-model-invocation: true`.

An implementation or fix leg must resolve `/implement`, `/tdd`, and `/code-review`, mutate files, run checks, and commit. A review leg must resolve `/code-review` and spawn its Standards and Spec subagents in parallel. Test the capability before the first mutation; select another host when any part is absent.

Give every leg a unique name containing the phase, task ID, and attempt. Run mutating legs sequentially in the integration worktree.

## Spawn intent

The control plane emits one of these per leg. Dispatch delivers it.

```text
name:      <phase>-<task-id>-<attempt>
packet:    <body from the matching packet section>
role:      implement | review | verify | recover | probe | rollover
model:     <ledger row; adapter resolves the host string>
effort:    medium | high | xhigh
isolation: fresh
wait:      idle | done | blocked
adapter:   <optional pin: herdr | buzz | native | pi>
issue:     <PRD or ticket number>
branch:    <integration branch>
work:      <short PRD title>
cwd:       <integration worktree>
```

Dispatch returns a receipt. The control plane accepts receipts only.

## Capability probe packet

Run this in a fresh read-only leg before the first mutation; an empty branch cannot probe `/code-review` itself because that skill correctly rejects an empty diff.

```text
PROBE ONLY — preserve HEAD and the worktree exactly.
EXPECTED HEAD: <SHA>
CONTROL DIRECTORY: <ledger and manifest directory>

Report the exact runner, model, fresh session ID, and explicit invocation syntax
and canonical paths for implement, tdd, and code-review. Prove parallel
subagents by launching two read-only agents at once: one returns
STANDARDS-PROBE plus HEAD; the other returns SPEC-PROBE plus HEAD. Read one
manifest entry from CONTROL DIRECTORY. Return all evidence in one probe receipt.
```

The probe passes only when the model resolves from house rules (or an approved override) and appears in the host catalog, all three skills resolve, both subagents return the expected HEAD from distinct contexts, the control directory is readable, and independent inspection confirms unchanged HEAD and a clean worktree.

## Implementation or fix packet

The first line must explicitly invoke the runtime's installed `implement` skill with that runner's prefix (`/implement`, `$implement`, or `/skill:implement`). Then provide only:

```text
TASK: <ticket or finding ref>
TASK SPEC: <ticket URL/path or generated per-finding spec path>
PARENT SPEC: <PRD ref>
WORKTREE / BRANCH: <paths and names>
TICKET_BASE_SHA: <immutable SHA>
DEPENDENCIES: <refs and integrated commit SHAs>
ACCEPTANCE CRITERIA: <the active task's criteria only>
COMPATIBILITY: <none or the review-before-commit adapter below>

Work only this task and fetch source bodies and comments from their refs. Follow
the repository instructions and installed skills. Resolve actionable
ticket-local findings, pass TASK SPEC explicitly to the nested /code-review,
reference TASK in the commits, and leave a clean worktree at the reported HEAD.

Return one compact receipt:
STATUS | HEAD | COMMITS | CHECKS | CRITERION EVIDENCE | REVIEW DISPOSITIONS | BLOCKERS
```

For a review finding, generate a small task-spec file in the control directory containing its axis, cited hunk, cited standard or spec line, source refs, and resolution criterion. Group findings only when one code change resolves the same root cause and every original criterion remains explicit.

Before the first implementation dispatch, inspect the installed `implement` and `code-review` instructions. If `implement` schedules review before commit while `code-review` compares only committed `<fixed-point>...HEAD`, set `COMPATIBILITY` to: "Checkpoint-commit the ticket before the nested review, pass `TICKET_BASE_SHA` as its fixed point and TASK SPEC as its explicit spec source, then commit review fixes." Otherwise set it to: "Pass `TICKET_BASE_SHA` and TASK SPEC explicitly to the nested review." The installed skills remain the source of truth.

## Recovery packet

Use the same explicit skill and task packet as the failed leg, adding only:

```text
ATTEMPT: 2 of 2
CURRENT HEAD / WORKTREE STATUS: <verified state>
FAILURE EVIDENCE: <command, exit/status, and concise error>

Establish the current state before changing it. Recover only the original task.
Return the original receipt fields plus RECOVERY DISPOSITION.
```

The coordinator applies the single retry and terminal-state rules in the main skill's **Failure bounds** section.

## Integration verification packet

This is a fresh read/check-only leg; it does not invoke `/implement` unless a failure is later turned into a fix task.

```text
REPOSITORY / WORKTREE: <path>
EXPECTED HEAD: <SHA>
COMMANDS: <exact integration commands pinned in the ledger>

Run every command at EXPECTED HEAD without editing files. Return:
STATUS | OBSERVED HEAD | COMMAND + EXIT CODE | FAILURES
```

Verification passes only when the observed SHA matches, every command exits successfully, and the worktree remains clean.

## Aggregate review packet

The first line explicitly invokes the installed `code-review` skill with the pinned merge-base SHA, using that runner's prefix (`/code-review`, `$code-review`, or `/skill:code-review`). Then provide:

```text
PR: <URL>
FIXED POINT: <pinned merge-base SHA>
EXPECTED HEAD: <SHA at dispatch>
COMPOSITE SPEC: <review-spec manifest path>
MODE: Read-only aggregate review of the entire PR.

Treat COMPOSITE SPEC as the explicit originating spec for this aggregate
invocation; it takes precedence over issue refs inferred from commit messages.
Fetch every source linked by the manifest. Return REVIEWED HEAD, a source
receipt listing every manifest entry and whether its full source was fetched,
the complete Standards and Spec reports separately, then a compact finding
index that preserves each finding's axis and evidence.
```

## Coordinator rollover packet

The first line explicitly invokes `orchestrate-prd` with the PRD ref, using the runner's prefix. Then pass only the PRD ref, ledger and manifest paths, host, house-rules path, repository/worktree, integration branch, pinned base SHA, PR URL if created, completed review-cycle count, and the next frontier. Reconstruct everything else from the tracker, Git, and the ledger.
