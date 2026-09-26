---
name: sync-fork
description: "Synchronize the main branch of one explicitly allowed OpenAction fork—openaction-ecologistes, openaction-placepublique, or openaction-lapres—with the latest citipo/openaction-europe main while preserving fork-specific behavior. Use when asked to update, refresh, replay, or synchronize one of these three forks with its Europe upstream."
---

# Synchronize an OpenAction Fork

Rebuild an allowed fork's `main` from the latest `citipo/openaction-europe:main`, then reapply and verify its
fork-specific commits.

## Allowed repositories

The input is a repository name, `owner/repository`, or URL. Only these forks are allowed:

- `citipo/openaction-ecologistes`;
- `citipo/openaction-placepublique`;
- `citipo/openaction-lapres`.

The upstream is always `citipo/openaction-europe`, and both branches are always `main`. Refuse anything else.

## Rules

- Follow the root `AGENTS.md` and the repository's instructions.
- Work from a checkout with remotes for the fork and Europe. Preserve unrelated changes and use an isolated worktree
  when the checkout is not clean.
- Stop and report if the available context is not enough to classify fork-specific commits safely.
- Cherry-pick fork-only commits by default. Rebase only when the whole fork-only series is linear, cohesive, and
  mechanical to replay, and explain why first.
- The sync request authorizes the final leased push to the fork's `main` without further confirmation.

## Check repository identity

1. Confirm from its remote URL that the requested repository is on the allowlist.
2. Identify the fork and Europe remotes from their fetch and push URLs, not their names.
3. Fetch both, record `ORIGINAL_FORK_SHA` and `UPSTREAM_SHA`, and confirm them with `git ls-remote`. Stop on a missing
   branch, identity mismatch, or concurrent change.

## Identify fork-specific behavior

1. From the merge base, list fork-only commits using both reachability (`git rev-list upstream/main..fork/main`) and
   patch equivalence (`git cherry`), so already-upstreamed patches are not replayed. Treat merge commits and past
   sync commits separately.
2. Inspect each candidate's diff and related PR or issue context. Stop if a commit's intent remains ambiguous.
3. Write a checklist of the fork behavior that must survive: features, branding, configuration, integrations,
   migrations, templates, and tests.
4. Classify each candidate as fork-specific (reapply), already in upstream (omit), replaced by a named upstream change
   (omit), or ambiguous (ask the user). Do not continue while any in-scope commit is ambiguous.

## Back up

Create `backup/main-before-europe-sync-<timestamp>` at `ORIGINAL_FORK_SHA`, push it to the fork, and verify it locally
and remotely before rewriting anything.

## Rebuild

1. Create a sync branch from `UPSTREAM_SHA`.
2. Reapply fork-specific commits oldest to newest. Flatten merge commits only when their custom behavior is fully
   understood.
3. Skip a commit only when its exact upstream replacement is identified and no fork behavior is lost; record the SHA,
   intent, replacement, and reason.
4. Keep meaningful commit boundaries and avoid unrelated cleanup.

Resolve conflicts on your own only when the result is mechanical (equivalent edits, renames, formatting, moved
context). In every case, keep both Europe and fork behavior where both are needed, check call sites, conditions, data
mapping, configuration, migrations, templates, translations, tests, and contracts, and never take one side wholesale
unless the other is provably kept elsewhere. Never guess business or client-specific logic: ask on any functional
conflict. Record each conflicted file and why both sides are preserved.

## Verify

1. The sync branch starts at `UPSTREAM_SHA` and contains only the classified fork series.
2. Compare the original fork behavior with the result (`git range-diff`, patch equivalence, code inspection), not
   commit counts.
3. Every changed file relative to Europe `main` maps to the checklist, with no drift or conflict artifacts.
4. Fork acceptance criteria and relevant upstream behavior still hold.
5. Run focused tests covering fork behavior, relevant upstream behavior, conflicted paths, and changed contracts.
6. Both remote `main` branches are still at `UPSTREAM_SHA` and `ORIGINAL_FORK_SHA`; if either moved, rebuild.

## Push

Summarize the strategy, backup, replay decisions, conflicts, tests, and old and new SHAs, then push:

```bash
git push \
  --force-with-lease=refs/heads/main:<original-fork-sha> \
  <fork-remote> <sync-branch>:refs/heads/main
```

Then confirm the fork's remote `main` equals the sync head and the backup is still available. Follow CI to its result:
fix failures caused by the replay or fork integration, rerun focused tests, and push again with the same backup and
lease checks. For infrastructure, flaky, or upstream failures that cannot be fixed in scope, retry when appropriate
and report the blocker without claiming success.

Never push `main` without a verified backup and an exact lease, discard fork behavior because upstream touched the
same files, continue past an ambiguous functional conflict, or claim completion before the remote `main` and its
required checks are verified.

## Output

Reply with the fork and upstream; the strategy and why; old fork, upstream, and new fork SHAs; the backup ref; commits
reapplied, changed, skipped, or dropped with reasons; conflicts and their resolution; tests and behavior checks; CI
failures and fixes; and the final check status, or the precise blocker.
