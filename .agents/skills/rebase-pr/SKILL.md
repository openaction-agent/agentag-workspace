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

- Follow the root `AGENTS.md`, the main reference for Linear, forges, Git, testing, and CI; this skill only adds
  the replay workflow. Its branch-rewrite rules (backup, lease, moved head) apply to the final push.
- Work from a checkout of the base repository.

## Understand the intent

1. Read the complete PR, with its commits oldest to newest.
2. Read any linked Linear issue; newer clarifications override an older PR body. Stop only if missing issue details
   prevent validating the behavior.
3. Without an issue, derive the intent from the PR discussion, diff, tests, and code, and stop if a material product
   decision stays ambiguous.
4. Write a short checklist of the expected behavior, contracts, tests, migrations, and out-of-scope work.

## Prepare

1. Identify the base repository and the writable head repository from their URLs (fork PRs are pushed to their head
   repository).
2. Fetch the base `main` and the PR head; record `LATEST_MAIN_SHA`, `ORIGINAL_HEAD_SHA`, and `HEAD_BRANCH`, and check
   that the fetched head matches the PR's head SHA.
3. Create the backup ref `backup/<head-branch>-before-rebase-<timestamp>` at `ORIGINAL_HEAD_SHA` in the head
   repository.
4. Create a clean branch (for example `rebase-pr-<number>`) from `LATEST_MAIN_SHA`.

## Replay the commits

1. Classify each commit of the PR's remote commit list as kept, superseded, empty, or needing manual integration.
2. Cherry-pick kept commits oldest to newest; do not use `git rebase`.
3. Do not cherry-pick merge commits blindly: flatten their unique behavior only when the intended patch is clear,
   otherwise ask.
4. Skip a commit that becomes empty because `main` already has the behavior, and record which change replaced it.
5. Keep meaningful commit boundaries; squash or split only when conflict resolution makes them misleading, and say so.

For each conflict, keep `main`'s behavior outside the PR scope and the PR's behavior inside it, integrating both when
upstream refactoring moved the code. Check call sites, tests, migrations, configuration, templates, and contracts, and
do not pick `ours` or `theirs` wholesale without verifying the result. Ask
when both sides cannot be preserved confidently. Record each conflicted file and how it was resolved.

## Verify

1. The clean branch starts at `LATEST_MAIN_SHA`.
2. Compare the original series with the result (`git range-diff` helps), allowing differences required by `main`.
3. Every changed file maps to the checklist, with no conflict markers or stray artifacts.
4. The PR's claims and the issue's acceptance criteria still hold; document intentional differences.
5. Run focused tests covering the affected behavior, especially conflicted paths and changed contracts.

## Push

Summarize the planned push, backup ref, kept and dropped commits, conflict decisions, and test results, then push:

```bash
git push \
  --force-with-lease=refs/heads/<head>:<original-head-sha> \
  <head-remote> <clean-branch>:refs/heads/<head>
```

Then re-read the PR, confirm it still targets `main` and its published diff matches the validated result, and follow
CI to a pass, fixing failures caused by the replay. Never drop behavior because a cherry-pick is hard.

## Output

Reply with the PR and head branch; old head, latest `main`, and new head SHAs; the backup ref; commits kept, changed,
skipped, or dropped with reasons; conflicts and their resolution; tests; CI failures and fixes; and the final PR and
check status, or the precise blocker.
