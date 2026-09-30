---
name: sync-forks
description: "Synchronize the main branch of the three OpenAction client forks—openaction-ecologistes, openaction-placepublique, and openaction-lapres—with the same latest citipo/openaction-europe main commit, in parallel, one subagent per fork, while preserving fork-specific behavior and escalating functional conflicts to a human. Use when asked to update, refresh, replay, or synchronize the forks (or one of them) with Europe, or before a preprod release when the forks are behind Europe."
---

# Synchronize the OpenAction Forks

Rebuild the `main` branch of each client fork from one pinned `citipo/openaction-europe:main` commit, then reapply
and verify its fork-specific commits. You orchestrate: you prepare one clone per fork, run one subagent per fork in
parallel, relay their progress, verify their results, and write the final report. The subagents do the per-fork
work described in [the fork sync procedure](references/fork-sync.md).

## Scope

The forks are `citipo/openaction-ecologistes`, `citipo/openaction-placepublique`, and `citipo/openaction-lapres`; the
upstream is always `citipo/openaction-europe`, and every branch involved is `main`. Sync all three unless the user
names a subset of them. Refuse any other repository or branch.

## Rules

- Follow the root `AGENTS.md`, the main reference for forges, Git, testing, and CI; this skill only adds the fork
  sync workflow. The sync request authorizes rewriting each fork's `main`, under the branch-rewrite rules (backup,
  lease, moved head), and nothing else: never push to Europe, never create tags or releases.
- Never guess business or client-specific logic. A functional conflict blocks only its fork: the other forks still
  finish, and the human gets every open decision at once in the final report.
- Do not do a fork's work yourself while its subagent is running, and do not redo a subagent's work. If a subagent
  fails or cannot be spawned, stop and report; do not replace the subagents with a sequential run
  without saying so.

## Prepare the clones

Work under `codebases/`, which stays in the thread's workspace for follow-ups.

1. Clone or fetch `codebases/openaction-europe`, then resolve `UPSTREAM_SHA`, the current Europe `main`, confirmed
   with `git ls-remote`. Every fork is rebuilt on this exact commit, even if Europe moves during the run, so that
   the forks stay consistent with each other for the next release.
2. For each fork, prepare its own clone, `codebases/openaction-<fork>`. Clone it with
   `--reference-if-able ../openaction-europe --dissociate` to reuse Europe's objects. If the clone already exists,
   fetch it and follow the `AGENTS.md` rules on unrelated local changes (use an isolated worktree when it is not
   clean).
3. In each fork clone, add Europe as the `europe` remote (or reuse a remote whose fetch and push URLs point to
   `citipo/openaction-europe`), fetch it, and check that `UPSTREAM_SHA` is present. Check that `origin` fetches and
   pushes the fork itself.
4. Create `codebases/sync-forks-<timestamp>/`, the run directory, with an empty `<fork>.progress.md` per fork.

Subagents never share a working tree: each fork has its own clone and its own progress file, and no subagent writes
to the Europe clone.

## Spawn the subagents

Spawn the three subagents in the same step so they run in parallel, one per fork, named `sync_<fork>`. Start each
with a fresh context (in Codex, `fork_turns: "none"`) and do not override its model or reasoning effort, so it runs on
the same route as you. Its brief must be self-contained:

- the fork (`citipo/openaction-<fork>`), the absolute path of its clone, the names of its fork and Europe remotes,
  and `UPSTREAM_SHA`;
- the absolute path of its progress file;
- the absolute path of the workspace root, which holds `AGENTS.md` and `bin/ci-wait`;
- the instruction to read the root `AGENTS.md` and
  `.agents/skills/sync-forks/references/fork-sync.md` (absolute paths), to follow that procedure for this fork only,
  and to end with the report format it defines;
- any user instruction specific to this fork (commits to drop, known decisions on earlier conflicts).

## Follow the subagents

The user does not see the subagents. While they run:

1. Wait for them with a bounded timeout (a few minutes), then read the progress files.
2. When a file gained a meaningful entry since your last update (classification done, backup pushed, a conflict
   resolved or escalated, push, CI result), post one concise combined update, one line per fork. Do not post when
   nothing changed, and never paste the progress files whole.
3. Repeat until every subagent has returned its final report.

## Verify

Do not trust the reports alone. For each fork a subagent reports as pushed, check that:

- the fork's remote `main` (`git ls-remote`) equals the reported new SHA;
- `UPSTREAM_SHA` is an ancestor of it (`git merge-base --is-ancestor`);
- the reported backup ref exists remotely at the reported old SHA;
- CI passed on the new SHA (`bin/ci-wait`).

For a blocked fork, check that its remote `main` is unchanged. Finally, compare Europe's remote `main` with
`UPSTREAM_SHA`: if it moved, say so and offer another sync; do not start it on your own.

## Follow-ups

When the user answers an escalated decision in this thread, resume that fork from the state and remaining sequence
its subagent recorded in the progress file and report, applying the user's decision (with a new subagent, or
yourself when only this fork remains), then verify it as above.

## Output

Reply following the workspace answer-language rule:

```markdown
Status: SYNCED | PARTIAL | BLOCKED | FAILED

Europe: `citipo/openaction-europe@<UPSTREAM_SHA short>` (<Europe main moved since: yes/no>)

| Fork | Status | Old → new `main` | Backup ref | Reapplied / skipped | Conflicts resolved | CI |
| --- | --- | --- | --- | --- | --- | --- |
| ecologistes | SYNCED / BLOCKED / FAILED | `<sha>` → `<sha>` | `<ref>` | <n> / <n> | <n> | <status> |

## <fork>
- Strategy and why; skipped or dropped commits with reasons.
- Conflicts resolved: <file> — <commit>: <what both sides did and how both behaviors were kept>.
- Tests run and behavior checks.

## Decisions needed
1. <fork> — <commit sha and title>, <file(s)>: <Europe intent> vs <fork intent>. Options: <A (consequence)>,
   <B (consequence)>. Recommendation: <option and why>, or none if it is a business choice.
```

`SYNCED` means every requested fork was pushed and CI passed; `PARTIAL` that some were; `BLOCKED` that none could be
pushed because of pending decisions; `FAILED` that a fork broke for another reason (with the precise error). Leave
`Decisions needed` out when there are none.
