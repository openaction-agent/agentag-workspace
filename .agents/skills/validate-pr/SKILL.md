---
name: validate-pr
description: "Functionally validate an OpenAction implementation, from a Linear issue ID or a GitHub pull request number or URL, in its Coolify preview with playwright-cli, then publish the result and update the Linear workflow. Use when asked to test, QA, recette, or validate a PR or issue implementation."
---

# Validate a Pull Request

Validate `citipo/openaction-europe` implementations against their accepted behavior in the Coolify preview. Other
repositories have no preview: only move their Linear issue to `To validate`.

The input is a Linear issue ID (`OPE-123`), a PR number (`#123`, optionally with `owner/repo`), or a PR URL.

## Rules

- Follow the root `AGENTS.md`, the main reference for Linear, forges, Git, and browser validation (including the
  result statuses); this skill only adds the preview validation workflow.
- Use the `playwright-cli` skill for browser commands. Do not modify implementation code, commit, or push.

## Workflow

1. Resolve the Linear issue (for a PR input, from the PR's links); if none can be found, ask for its ID. Immediately
   move it to `Validation in progress` and confirm the update before doing anything else.
2. Read the complete issue and its PR.
3. If the repository is not `citipo/openaction-europe`, move the issue to `To validate`, report that preview
   validation does not apply, and stop.
4. Prepare the checkout and fixtures, the journey, and the preview URLs (sections below), then run the journey.
5. Post the same French summary as one GitHub PR comment and one Linear comment.
6. Update Linear and confirm it:
   - `PASSED`: add the existing team label `AgentValidé` to the current labels (without removing or duplicating any),
     move the issue to `Ready to merge`, then re-read it to confirm both. If either fails, report finalization as
     blocked.
   - `FAILED` or `BLOCKED`: move the issue to `To validate` without changing labels.

## Checkout and fixtures

1. Check out `citipo/openaction-europe` at the exact PR head SHA, in a clean detached checkout or worktree. Never
   use a stale checkout or the GitHub diff instead of the source.
2. Read the repository instructions, then the preview bootstrap and fixtures (`console/bin/setup-preview`,
   `console/src/DataFixtures/TestFixtures.php`, `console/src/DataFixtures/DevFixtures.php`, and related helpers) to
   find which fixtures the preview loads.
3. From them, resolve the account, password, 2FA state, organization, and permissions needed, preferring a non-2FA
   account with the narrowest sufficient rights, and every named test record.

## Test journey

1. Use the latest `## Parcours de test sur la prévisualisation` section of the PR body (the Linear copy can help, but
   the PR is authoritative) when it names every app, account and rights, navigation, action, and expected result, and
   each prerequisite is either a verified fixture or complete creation steps. Follow it exactly.
2. Otherwise, design a French journey from the accepted behavior, the diff, the code, and verified fixtures. Never
   invent fixtures.
3. Cover the main changed behavior, its important permission, validation, or edge path, and the closest regression
   paths justified by the diff, each with an expected visible result.
4. If a fixture, role, product decision, safe action, or creation step remains unknown, mark that path blocked.

## Preview URLs and login

- Use only URLs from the Coolify bot comment for this PR and head, never derived, edited, or copied from another PR.
  Match each to its app (`console`, `public`, `platform`, `mobilisation`) and use only those the journey needs. A
  missing, ambiguous, stale, unreachable, or unsafe URL blocks its path.
- A fresh Console session starts on the login page; that is expected. Log in with the preview fixture credentials,
  which are authorized only for that preview.
- Authentication is blocked only if the fixture account is absent, its login is rejected, permissions are missing,
  2FA cannot be avoided, or the preview did not load the fixtures. Keep evidence of the failed attempt.

## Running the journey

1. Name sessions after the PR and app (for example `validate-pr-123-console`).
2. After login, confirm access to the expected organization and records.
3. Create any run-owned data through the prescribed UI steps with its unique value and verify the result. Do not
   replace it with shared fixtures or guess fields.
4. Follow the journey exactly, then run the adjacent regression paths the same way, proportionate to the change.
5. Record created data so it can be cleaned up later.

## Result

Use the statuses of the root `AGENTS.md`. A blocked authenticated path additionally requires fixture discovery and a
failed login attempt (or a precise reason why none was possible). If the PR head SHA changed before publishing, rerun
the affected paths.

Reply with:

```markdown
Status: PASSED | FAILED | BLOCKED

- Input: <Linear issue and/or PR link>
- PR head: `<SHA>`
- Journey: <existing PR section | designed from verified context>
- Target: <Europe Coolify app URL>

| Path | App and fixture | Actions and visible result | Console/network | Status |
| --- | --- | --- | --- | --- |
| <name> | <app, fixture> | <concise evidence> | <relevant evidence> | <status> |

Regressions: <paths checked and result>
Defects or blockers: <reproduction, impact, and missing evidence>
Artifacts: <safe local snapshot, screenshot, or trace paths when retained>
External publication: GitHub PR comment and Linear comment posted
```

GitHub and Linear comment (concise, without secrets, personal data, raw logs, or long traces):

```markdown
## Résultat de validation

Statut : <Réussi | Échoué | Bloqué>

<Résumé court du résultat fonctionnel, en français non technique.>

## Parcours vérifiés

- <Parcours, application, fixture, actions principales, résultat observé>

## Problèmes détectés

- <Bug, régression, erreur console/réseau pertinente, ou blocage avec impact>

## Éléments de preuve

- PR : <URL>
- Commit testé : `<SHA>`
- Prévisualisation : <app et URL Coolify utilisée>
- Artefacts : <capture, snapshot ou trace conservée, si utile et sans secret>
```

When validation passes, write `Aucun problème détecté.` under `Problèmes détectés`.
