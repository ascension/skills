---
name: review-pr-comments
description: Review automated PR review comments (Cursor BugBot, CodeRabbit, etc.), verify each issue exists in the actual code, fix confirmed bugs, and push fixes. Use when user wants to address PR review comments, fix BugBot issues, review cursor comments, or resolve automated code review feedback on a pull request.
---

# Review PR Comments

Fetch automated review comments from a PR, verify each flagged issue in the code, fix real bugs, and push.

## Workflow

1. **Identify the PR** — determine the PR number from:
   - User-provided number
   - Current branch (`gh pr view --json number` or GitHub MCP)
   - Ask user if ambiguous

2. **Fetch review comments** — use GitHub MCP `pull_request_read` with `get_review_comments` method. Paginate if needed (`perPage: 100`).

3. **Filter to automated reviewers** — focus on comments from bot authors (e.g., `cursor`, `coderabbitai`, `github-actions`). Skip human review comments unless user asks.

4. **For each comment, verify the issue exists**:
   - Read the exact file and line range cited in the comment
   - Determine if the bug is real, a false positive, or already fixed
   - Categorize: **fix** (real bug), **skip** (false positive or low-value), **defer** (real but out of scope)
   - Explain your assessment briefly before acting

5. **Fix confirmed issues**:
   - Make minimal, targeted fixes — don't refactor surrounding code
   - Run type checking (`tsc --noEmit`) after fixes to confirm no regressions
   - Group related fixes into a single commit when possible

6. **Commit and push**:
   - Stage only the fixed files
   - Use conventional commit: `fix: address BugBot review comments on PR #<number>`
   - Include a brief description of each fix in the commit body
   - Push to the current branch

7. **Report summary** — list each comment with its disposition:
   - Fixed (with one-line description of fix)
   - Skipped (with reason: false positive, pre-existing, etc.)
   - Deferred (with reason: out of scope, needs discussion, etc.)

## Important

- **Always verify before fixing** — automated reviewers have false positives. Read the actual code.
- **Don't fix pre-existing issues** unless user asks — only fix issues introduced in the PR's diff.
- **Type check after fixes** — run the project's type checker to ensure fixes don't break anything.
- **Use GitHub MCP tools** for all GitHub operations (not `gh` CLI).
