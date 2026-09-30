---
name: release-preprod
description: "Release a new OpenAction version to preproduction: check that the client forks (openaction-ecologistes, openaction-placepublique, openaction-lapres) contain every citipo/openaction-europe main commit, then create the version tag and a GitHub release explaining the changes since the previous version on the latest main of the four repositories, and follow the preprod deployments. Use when asked to release, tag, or deploy a version to preprod/staging, or to publish a new OpenAction version."
---

# Release a Version to Preproduction

Tag the latest `main` of `citipo/openaction-europe`, `citipo/openaction-ecologistes`,
`citipo/openaction-placepublique`, and `citipo/openaction-lapres` with the same version, publish a GitHub release for
each one explaining what changed since the previous version, and follow the preprod deployments they trigger.

## Inputs

The version is required, as bare semantic versioning like the existing tags (`1.28.0`, not `v1.28.0`). If the user
writes a `v` prefix, drop it and say so. Ask for the version when it is missing, and never pick one yourself.

## Rules

- Follow the root `AGENTS.md`, the main reference for forges, Git, and CI; this skill only adds the release workflow.
- Pushing a tag to any of these repositories runs its `preprod` workflow, which builds the release images and deploys
  them to that instance's preproduction. The release request authorizes exactly that for the four repositories, at
  the SHAs checked below, and nothing else.
- Never run the `prod` workflow, and never deploy to production.
- Never synchronize forks, rewrite a branch, or change code here. When a fork is not synced, stop before creating
  anything and tell the user to run `$sync-forks` first.
- Never delete, move, or recreate a tag or release without explicit confirmation, even after a failure.
- Create nothing until every check below passes for all four repositories.

## Check the version

1. For each repository, find the latest published release (`gh release list --exclude-drafts`, compared as semantic
   versions) and its tag SHA. They should all be the same version; if not, report the difference and ask how to
   proceed.
2. Check that the requested tag exists in none of the four repositories (`git ls-remote --tags`), as a tag or as a
   release.
3. The version must be greater than the previous one. If it is not the next patch, minor, or major increment (for
   example `1.27.3` to `1.29.0`), ask for confirmation.

## Check the repositories

1. Clone or fetch the four repositories under `codebases/`, following the `AGENTS.md` rules on existing checkouts, and
   record each `main` SHA, confirmed with `git ls-remote`. These are the SHAs you release.
2. For each fork, add or reuse a remote whose URLs point to Europe, fetch it, and check that Europe's recorded `main`
   is an ancestor of the fork's `main` (`git merge-base --is-ancestor`). If it is not, list the missing Europe commits
   (`git log --oneline <fork>/main..<europe>/main`) and stop: the forks must first be synced with `$sync-forks`.
3. Check that CI passed on each recorded `main` SHA with the workspace's `bin/ci-wait`. If CI failed, stop and report
   the failed jobs; if it is still running, keep following it.
4. Check that each previous release tag is still reachable or recorded: for Europe, the previous tag must be an
   ancestor of `main`. Fork history is rewritten by each sync, so a fork's previous tag usually is not; that is
   expected.

## Write the release notes

Write the notes in English, like the existing releases, one file per repository. Explain the changes from the
previous version's tag to the recorded `main` SHA; never invent or extrapolate a change.

Europe:

1. Generate the list of merged pull requests with
   `gh api repos/citipo/openaction-europe/releases/generate-notes -f tag_name=<version>
   -f target_commitish=<sha> -f previous_tag_name=<previous>`, and keep its `What's Changed` list and
   `Full Changelog` link.
2. Check the list against `git log <previous>..<sha>`: commits pushed without a pull request are missing from it,
   so add them.
3. When the diff contains changes that matter to whoever deploys (database migrations, new or renamed environment
   variables, configuration, new services or workers, dependency or runtime upgrades), add a `Deployment notes`
   section listing them, found from the diff (`git diff --stat <previous>..<sha>` and the files involved), not from
   titles alone.

Each fork:

1. A fork's changes are the Europe changes plus its own. Start with a `Europe changes` section that links the Europe
   release of this version and repeats its list and deployment notes.
2. Add a `<fork> changes` section comparing the fork-specific commits of the two versions: the previous series is
   `$(git merge-base <previous> <europe>/main)..<previous>` (the commits on top of the Europe base of the previous
   tag), and the new one is `<europe-sha>..<sha>`. Use `git range-diff` between them and describe the fork-specific
   commits that were added, changed (for example adapted during a sync), or removed, linking their pull requests
   when there are any. If nothing changed, say so in one line.
3. Add the fork's own deployment notes when its commits bring any, and end with
   `**Full Changelog**: https://github.com/citipo/openaction-<fork>/compare/<previous>...<version>`.

## Publish

1. Just before publishing, re-read each `main` (`git ls-remote`). If one moved, still tag the recorded SHAs, which
   were checked together, and mention it in the result.
2. Publish Europe first, then the forks:

   ```bash
   gh release create <version> -R citipo/openaction-<repository> \
     --target <recorded-sha> --title <version> --notes-file <notes-file>
   ```

3. After each release, check that the tag points to the recorded SHA and that the release is published, not a draft
   or a pre-release. If one fails, stop, and report which tags and releases exist and which do not.

## Follow the deployments

For each repository, find the `preprod` workflow run of the new tag
(`gh run list -R <repository> --workflow preprod.yaml --json databaseId,headBranch,headSha,status,conclusion`, where
`headBranch` is the tag) and wait until it appears. Then follow it to its result with `bin/ci-wait` on the tagged
SHA, whose check runs then include the release and deploy jobs. If a job fails, read its log as `AGENTS.md`
describes, rerun the failed jobs once (`gh run rerun <run-id> --failed`) when the failure is an infrastructure or
flaky one, and otherwise report it. Never claim the version is on preprod before the deploy job passed.

## Output

Reply following the workspace answer-language rule:

```markdown
Status: RELEASED | PARTIAL | BLOCKED | FAILED

Version: <version> (previous: <previous version>)

| Repository | Released SHA | Europe included | Release | Preprod deploy |
| --- | --- | --- | --- | --- |
| europe | `<sha>` | — | <release URL> | <passed | failed: job | pending> |
| ecologistes | `<sha>` | yes | <release URL> | <status> |

Highlights: <the few changes that matter most, and deployment notes>
Blockers: <unsynced forks with missing Europe commits, failed CI or deploy jobs, or none>
```

`RELEASED` means all four releases were published and deployed to preprod; `PARTIAL` that some were;
`BLOCKED` that nothing was created because a check failed (unsynced fork, failed CI, version conflict), with what to
do next; `FAILED` that a release or deployment failed, with the precise error and the tags and releases that exist.
