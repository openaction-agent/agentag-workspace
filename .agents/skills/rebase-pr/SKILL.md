---
name: rebase-pr
description: "Rebuild an OpenAction GitHub pull request on the latest main by cherry-picking its commits onto a clean branch, resolving conflicts, validating the result against the PR and any linked Linear issue, and safely updating the original PR branch. Use when asked to rebase, refresh, cleanly replay, or repair a conflicting or outdated PR branch."
---

# Rebase a Pull Request

Replay one pull request onto the latest `main` with cherry-picks, preserving its intended behavior rather than its old
textual diff.

The input is a PR URL, `owner/repository#number`, or a PR number when the current checkout makes the repository
unambiguous. Ask only when the PR cannot be resolved. The PR must target `main`; stop otherwise.

## Rules

- Follow the root `AGENTS.md` and the repository's instructions.
- Work from a checkout of the base repository. Preserve unrelated changes, use an isolated worktree when the checkout
  is not clean, and never reset a user's checkout or reuse a branch with unrelated work.
- Stop and report if the PR or its linked issue cannot be read well enough to validate the result.
- The rebase request authorizes the final `--force-with-lease` push without further confirmation.

## Understand the intent

1. Read the complete PR: title, body, base and head repositories and branches, head SHA, commits oldest to newest,
   diff, conversation, reviews, and checks.
2. Read any linked Linear issue (comments, links, attachments); newer clarifications override an older PR body. Stop
   only if missing issue details prevent validating the behavior.
3. Without an issue, derive the intent from the PR discussion, diff, tests, and code, and stop if a material product
   decision stays ambiguous.
4. Write a short checklist of the expected behavior, contracts, tests, migrations, and out-of-scope work.

## Prepare

1. Identify the base repository and the writable head repository from their URLs (fork PRs are pushed to their head
   repository).
2. Fetch the base `main` and the PR head; record `LATEST_MAIN_SHA`, `ORIGINAL_HEAD_SHA`, and `HEAD_BRANCH`, and check
   that the fetched head matches the PR's head SHA.
3. Create a backup ref `backup/<head-branch>-before-rebase-<timestamp>` at `ORIGINAL_HEAD_SHA`, push it to the head
   repository, and verify the remote backup.
4. Create a clean branch (for example `rebase-pr-<number>`) from `LATEST_MAIN_SHA`.

## Replay the commits

1. Classify each commit of the PR's remote commit list as kept, superseded, empty, or needing manual integration.
2. Cherry-pick kept commits oldest to newest; do not use `git rebase`.
3. Do not cherry-pick merge commits blindly: flatten their unique behavior only when the intended patch is clear,
   otherwise ask.
4. Skip a commit that becomes empty because `main` already has the behavior, and record which change replaced it.
5. Keep meaningful commit boundaries; squash or split only when conflict resolution makes them misleading, and say so.

For each conflict, keep `main`'s behavior outside the PR scope and the PR's behavior inside it, integrating both when
upstream refactoring moved the code. Check call sites, tests, migrations, configuration, templates, and contracts; do
not pick `ours` or `theirs` wholesale without verifying the result, and avoid unrelated refactors or formatting. Ask
when both sides cannot be preserved confidently. Record each conflicted file and how it was resolved.

## Verify

1. The clean branch starts at `LATEST_MAIN_SHA`.
2. Compare the original series with the result (`git range-diff` helps), allowing differences required by `main`.
3. Every changed file maps to the checklist, with no conflict markers or stray artifacts.
4. The PR's claims and the issue's acceptance criteria still hold; document intentional differences.
5. Run focused tests covering the affected behavior, especially conflicted paths and changed contracts.
6. The remote PR head is still `ORIGINAL_HEAD_SHA`; if it moved, stop and restart from the new head.

## Push

Summarize the planned push, backup ref, kept and dropped commits, conflict decisions, and test results, then push:

```bash
git push \
  --force-with-lease=refs/heads/<head>:<original-head-sha> \
  <head-remote> <clean-branch>:refs/heads/<head>
```

Then re-read the PR (head SHA, diff, files, checks), confirm it still targets `main`, that the published diff matches
the validated result, and that the backup is still available. Follow CI to its result: fix failures caused by the
replay, rerun focused tests, and push again with the same safety checks. For infrastructure, flaky, or upstream
failures that cannot be fixed in scope, retry when appropriate and report the blocker without claiming the PR is ready.

Never rewrite without a verified backup, force-push without an exact lease, overwrite a head that moved, drop behavior
because a cherry-pick is hard, or claim success before the remote PR matches the validated result.

## Output

Reply with the PR and head branch; old head, latest `main`, and new head SHAs; the backup ref; commits kept, changed,
skipped, or dropped with reasons; conflicts and their resolution; tests; CI failures and fixes; and the final PR and
check status, or the precise blocker.
