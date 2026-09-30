# Fork Sync Procedure

Rebuild one client fork's `main` from the pinned Europe commit, then reapply and verify its fork-specific commits.
This is the procedure a `sync-forks` subagent follows for the single fork it was given.

## Your brief

You receive the fork, the absolute path of its clone, its fork and Europe remotes, the pinned `UPSTREAM_SHA`, and the
absolute path of your progress file. Work only in that clone and on that fork. Never push to Europe, never touch
another fork's clone or progress file, never create tags or releases, and never change `UPSTREAM_SHA`: if Europe's
`main` moved, note it and keep syncing to `UPSTREAM_SHA`.

## Rules

- Follow the root `AGENTS.md`, the main reference for Git, testing, and CI. Your brief authorizes rewriting this
  fork's `main` under its branch-rewrite rules (backup, lease, moved head).
- Cherry-pick fork-only commits by default. Rebase only when the whole fork-only series is linear, cohesive, and
  mechanical to replay, and explain why first.
- Never guess business or client-specific logic. On any functional conflict or ambiguous commit, do not push; follow
  "Escalate" below.

## Report progress

The user cannot see you; the orchestrator relays your progress file. Append one short line to it, prefixed with the
UTC time, at each milestone:

- identity checked, with `ORIGINAL_FORK_SHA` and `UPSTREAM_SHA`;
- classification done: fork-only commits found, to reapply, already upstream, replaced, ambiguous;
- backup ref pushed;
- each conflict, as it is resolved or escalated: commit, files, and a few words on how;
- tests run and their outcome;
- push done with the new SHA, then the CI result;
- any blocker, as soon as it appears.

Keep the lines factual and short. Never write secrets or full logs there.

## Check repository identity

1. Confirm from the remote URLs that the clone is the fork of your brief.
2. Identify the fork and Europe remotes from their fetch and push URLs, not their names.
3. Fetch both, record `ORIGINAL_FORK_SHA`, and confirm it and `UPSTREAM_SHA` with `git ls-remote`. Stop on a missing
   branch, identity mismatch, or concurrent change.

## Identify fork-specific behavior

1. From the merge base, list fork-only commits using both reachability (`git rev-list <upstream>..<fork>/main`) and
   patch equivalence (`git cherry`), so already-upstreamed patches are not replayed. Treat merge commits and past
   sync commits separately.
2. Inspect each candidate's diff and related PR or issue context.
3. Write a checklist of the fork behavior that must survive: features, branding, configuration, integrations,
   migrations, templates, and tests.
4. Classify each candidate as fork-specific (reapply), already in upstream (omit), replaced by a named upstream change
   (omit), or ambiguous (escalate).

## Back up

Push the backup ref `backup/main-before-europe-sync-<timestamp>` at `ORIGINAL_FORK_SHA` to the fork and verify it
remotely before rewriting anything.

## Rebuild

1. Create a local sync branch from `UPSTREAM_SHA`.
2. Reapply fork-specific commits oldest to newest. Flatten merge commits only when their custom behavior is fully
   understood.
3. Skip a commit only when its exact upstream replacement is identified and no fork behavior is lost; record the SHA,
   intent, replacement, and reason.
4. Keep meaningful commit boundaries and the original authors.

Resolve conflicts on your own only when the result is mechanical (equivalent edits, renames, formatting, moved
context). In every case, keep both Europe and fork behavior where both are needed; check call sites, conditions, data
mapping, configuration, migrations (ordering and duplicates included), templates, translations, tests, and
contracts; and never take one side wholesale unless the other is provably kept elsewhere. Record each conflicted
file, what each side intended, and why both are preserved.

## Escalate

When a conflict or commit needs a business or client-specific decision:

1. Keep the commits already replayed on the local sync branch, abort the blocked cherry-pick, and record the exact
   remaining sequence (blocked commit first) in your progress file so the work can resume after the decision.
2. Check the rest of the sequence for further conflicts without applying it (for example with
   `git merge-tree --write-tree` per remaining commit), so the human can decide on all of them at once.
3. Do not push `main`. Leave the fork's remote `main` untouched and the backup ref in place.
4. For each decision, give the commit, files, what Europe's side does, what the fork's side does, the options with
   their consequences, and your recommendation, or say that it is a business choice.

## Verify

1. The sync branch starts at `UPSTREAM_SHA` and contains only the classified fork series.
2. Compare the original fork behavior with the result (`git range-diff`, patch equivalence, code inspection), not
   commit counts.
3. Every changed file relative to `UPSTREAM_SHA` maps to the checklist, with no drift or conflict artifacts (no
   conflict markers, no stray files).
4. Fork acceptance criteria and relevant upstream behavior still hold.
5. Run focused tests covering fork behavior, relevant upstream behavior, conflicted paths, and changed contracts.

## Push

Check that the fork's `main` is still at `ORIGINAL_FORK_SHA` (restart from the new head if it moved), then push:

```bash
git push \
  --force-with-lease=refs/heads/main:<original-fork-sha> \
  <fork-remote> <sync-branch>:refs/heads/main
```

Then follow CI to a pass with the workspace's `bin/ci-wait`, fixing failures caused by the replay or fork
integration (with a new leased push for each fix). Never discard fork behavior because upstream touched the same
files.

## Report

End with this report, in English, for the orchestrator:

```markdown
Fork: citipo/openaction-<fork>
Status: SYNCED | BLOCKED | FAILED
Upstream: <UPSTREAM_SHA> (Europe main moved since: yes/no)
Old main: <ORIGINAL_FORK_SHA>   New main: <sha, or unchanged>
Backup: <ref>
Strategy: <cherry-pick or rebase, and why>

Commits: <reapplied> reapplied, <changed> changed, <skipped> skipped, <dropped> dropped
- <sha> <title>: <reapplied | changed: how | skipped: upstream replacement | dropped: reason>

Conflicts resolved:
- <commit> <file>: <Europe side> / <fork side> → <resolution, and why both behaviors are kept>

Decisions needed:
- <commit> <file(s)>: <Europe intent> vs <fork intent>; options: <...>; recommendation: <...>
Remaining sequence: <sha list, blocked commit first, or none>
Local state: <sync branch name and last replayed commit>

Tests: <commands and results>
CI: <passed | failed: jobs and fixes | not run: why>
Notes: <anything else the orchestrator or human should know>
```
