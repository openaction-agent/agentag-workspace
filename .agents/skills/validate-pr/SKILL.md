---
name: validate-pr
description: "Validate an OpenAction implementation from a Linear issue ID or GitHub pull request number or URL. Use when asked to test, QA, or validate a PR through its French test journey: immediately mark the Linear issue as Validation in progress, route non-Europe issues to To validate, use Europe Coolify preview URLs for real functional validation, execute the flow with playwright-cli, post matching French GitHub and Linear comments, label passed issues AgentValidé and move them to Ready to merge, move unsuccessful issues to To validate, and report evidence as passed, failed, or blocked."
---

# Validate a Pull Request

Validate Europe implementations against their accepted behavior in the Coolify
preview. For every other repository, skip browser validation because Coolify
preview URLs are not available and move the Linear issue to `To validate`
after the initial validation status transition.

## Inputs

Accept one of:

- a Linear issue ID, such as `OPE-123`;
- a GitHub PR number, such as `#123`, optionally with `owner/repo`;
- a GitHub pull request URL.

Ask the user to choose only when the input resolves to multiple plausible PRs.

## Operating rules

- Follow the testing, Git, and CI policy in the workspace root `AGENTS.md`.
  This validation workflow does not run local suites, commit, or push; use its
  bounded-log rule if CI failure evidence must be inspected.
- Follow the repository's `AGENTS.md` and local contribution instructions.
- Use the GitHub and Linear access available in the local environment. Require
  complete PR and issue context plus the ability to publish the required
  comments and status updates; stop and report a missing capability.
- Load and follow the `playwright-cli` skill for all browser navigation,
  interaction, snapshots, console inspection, and network inspection. Before
  opening a preview, verify that `command -v playwright-cli` succeeds. If it
  does not, report the validation as blocked; do not install it or silently
  substitute `npx`, a browser connector, `curl`, or another automation tool.
- For Europe, this skill updates Linear status, posts one GitHub PR comment,
  posts one Linear comment with the same content, and then moves Linear to
  `Ready to merge` when validation passes or `To validate` when validation
  fails or is blocked. For non-Europe repositories, move Linear to
  `To validate` after the initial status transition and do not post
  preview-validation comments.
- Never test on production or with production data. Only work on the preview 
  environments (which are safe, no risky capability).
- Do not modify implementation code while validating. For Europe validation,
  require a local checkout at the exact PR head before browser testing. Clone
  the repository under `codebases` when absent; fetch the PR head and use a
  clean detached checkout or isolated worktree when the existing checkout is
  dirty or points elsewhere. Preserve unrelated work.

## Workflow

1. Resolve only the Linear issue identity. For a PR input, read the minimum PR
   metadata and verified links needed to find its Linear issue. If no issue can
   be resolved, stop and ask for its ID. Immediately move the resolved issue
   to `Validation in progress` and confirm the update succeeded before loading
   the full validation context or performing any validation.
2. Read the complete Linear issue, comments, links, and attachments. Resolve
   its implementation PR and repository from verified links.
3. Read the PR metadata, complete body, current head SHA, changed files and
   diff, conversation comments, checks, and statuses. Determine whether the
   repository is `citipo/openaction-europe`. Read deployments or Coolify
   comments only when they are relevant. Retrieve complete paginated results.
4. Treat the latest Linear clarification as accepted behavior. Record any
   disagreement among Linear, the PR description, and the current
   implementation before testing.
5. If the repository is not `citipo/openaction-europe`, move the Linear issue
   to `To validate`, report that Coolify validation is not applicable for
   this project, and stop. Do not wait for preview URLs, run a substitute
   browser validation, or mention preview URLs in the summary.

## Europe validation lifecycle

For `citipo/openaction-europe` only:

1. Prepare the exact current-head checkout and discover its preview fixtures
   as described below.
2. Execute the validation workflow below against the current PR head and
   Coolify preview URLs.
3. Post one GitHub PR comment and one Linear comment with exactly the same
   French validation summary.
4. Update the Linear issue according to the overall validation result and
   confirm the update succeeded:
   - `PASSED`: resolve the existing team label named exactly `AgentValidé`,
     re-read the issue's current labels, append `AgentValidé` without removing
     or duplicating any existing label, and move the issue to `Ready to merge`.
     Re-read the issue and confirm both the label and status. If the exact label
     is unavailable or either update cannot be confirmed, report workflow
     finalization as blocked and do not claim external publication completed.
   - `FAILED` or `BLOCKED`: move it to `To validate`; do not add or remove
     `AgentValidé` or alter any other label.
   The comments must make failures and blockers clear.

## Prepare the current-head checkout and fixtures

1. Ensure `citipo/openaction-europe` is checked out under `codebases`, fetch the
   resolved PR head, and verify that the inspected checkout is at the exact
   remote head SHA. Do not use a stale branch, another task's checkout, or the
   GitHub diff as a substitute for current source inspection.
2. Read the repository instructions at that checkout before continuing.
3. Inspect the current preview bootstrap and fixture definitions, including
   `console/bin/setup-preview`, `console/src/DataFixtures/TestFixtures.php`,
   `console/src/DataFixtures/DevFixtures.php`, and any relevant fixture helpers.
   Determine which fixture groups the preview actually loads.
4. Resolve and record internally the non-production account, password source,
   2FA state, organization membership and permissions required by the journey.
   Resolve every named test record from the same fixtures. Prefer a non-2FA
   account with the narrowest sufficient permissions.
5. Treat missing or unusable fixture access as a blocker only after inspecting
   this current-head source and attempting the safe fixture authentication flow
   below. Report the precise missing fixture, role, or bootstrap evidence.

## Recover or design the test journey

1. Locate the latest `## Parcours de test sur la prévisualisation` section in
   the PR body. If needed, compare it with the copy in the linked Linear issue
   or comment, but prefer the current PR body for the current implementation.
2. Use the existing French journey only when it names every required app,
   fixture account and permission, navigation path, UI action, and visible
   expected result. For every prerequisite record, require either an exact
   verified fixture or test record, or complete safe UI instructions to create
   run-owned temporary data. Preserve its sequence and data exactly.
3. If the section is absent or unusable, design a French journey from the
   accepted Linear behavior, PR diff and body, relevant current code, and
   verified safe fixtures. Inspect fixture definitions before naming a record;
   never invent a fixture or substitute production data.
4. Cover the primary changed behavior, its important permission, validation,
   or edge path, and the smallest adjacent regression paths justified by the
   diff. Keep every step observable through the UI and identify the expected
   visible result.
5. If a required fixture, account role, product decision, safe action, or
   complete data-creation instruction remains unknown, mark the affected path
   blocked instead of improvising.

## Resolve the target environment

1. For `citipo/openaction-europe`, read the Coolify bot comment on the resolved
   PR. Accept only URLs explicitly present in a comment for that PR and current
   head. Never derive, edit, copy from another PR, or guess a preview URL.
2. For Europe, match each URL to its named application, such as `console`,
   `public`, `platform`, or `mobilisation`. Use every app required by the
   journey and regression paths; do not require an unrelated app.
3. Mark a path blocked when its required Europe Coolify URL is missing,
   ambiguous, stale, unreachable, or unsafe. Report the exact missing evidence.
4. Treat an unauthenticated Console login page as an expected starting state,
   not as evidence that authorization is unavailable.
5. Use deterministic credentials committed as preview fixture data to log in
   through visible UI controls. Such fixture credentials are authorized only
   for the matching preview; never use them on production or substitute a real
   staff account. They may be supplied to UI automation but must not appear in
   snapshots, screenshots, retained artifacts, GitHub, Linear, or the final
   summary.
6. Use an already authorized browser state when one is safely available, but do
   not require one. Mark authentication blocked only when the current-head
   fixture account is absent, its verified login is rejected, required
   permissions are missing, unavoidable 2FA prevents access, or the preview did
   not load the expected fixtures. Preserve concise evidence of the failed
   attempt without exposing credentials.

## Execute the journey with playwright-cli

1. Create an isolated in-memory named session per app, for example
   `validate-pr-123-console`, and open the exact quoted target URL. Do not use
   a persistent profile unless the user supplies an authorized one. Expect a
   fresh Console session to begin at the login page.
2. Capture the starting URL and a focused snapshot. When login is shown, enter
   the verified preview fixture account through visible controls, submit the
   form, and confirm that the expected organization and fixture records are
   accessible before executing the journey. Lack of pre-existing browser state
   alone is never a blocker.
3. Create any journey-prescribed run-owned temporary data through the stated
   visible UI steps. Use the specified non-sensitive unique value, verify the
   stated setup result, and record the value internally before continuing.
   Do not substitute a shared fixture, guess missing fields, or use an API.
4. Follow the planned navigation, actions, and fixture or run-owned data
   exactly through visible UI controls.
   Do not use JavaScript evaluation, storage edits, request mocking, or direct
   API calls to bypass application behavior.
5. After every meaningful action, verify the expected visible state with a
   focused snapshot or read-only element inspection. Check persistence after
   navigation or reload when it is part of the accepted behavior.
6. Run the relevant adjacent regression paths in the same way. Keep them
   proportional to the changed area; do not broaden validation into a generic
   product tour.
7. After each path, inspect `playwright-cli console` for errors and
   `playwright-cli requests` for failed requests. Inspect relevant request
   details and correlate them with the action. Distinguish harmless or known
   preview noise from evidence of an introduced defect; do not silently ignore
   either.
8. Capture concise evidence: application and URL, fixture or run-owned data,
   key actions, final visible outcome, relevant console errors, failed
   requests, and a focused screenshot or snapshot when it materially supports
   the result.
   Exclude tokens, credentials, personal data, and unrelated internal logs.
9. Do not delete run-owned data without the user's confirmation. Never delete
   a shared fixture or pre-existing record. Record the created data so that a
   confirmed cleanup can target it precisely.
10. Close every named browser session. Do not leave authentication-state files
   or other sensitive artifacts behind.

## Classify the result

Classify every required path and the overall validation:

- `PASSED`: every required path ran on the current target environment with its
  exact fixture or verified run-owned setup, all visible outcomes matched, and
  no relevant introduced console or network failure was observed.
- `FAILED`: a visible outcome is wrong, an adjacent regression is reproduced,
  or a relevant console error or failed request demonstrates a defect.
- `BLOCKED`: a required PR, target URL, fixture, authorization, safe action, or
  `playwright-cli` capability is unavailable, so correctness cannot be
  established. For authenticated preview paths, require current-head fixture
  discovery plus a failed fixture login attempt or a precise reason why that
  safe attempt was impossible; a login page by itself is not a blocker.

Use `FAILED` overall when any required path fails, even if another path is
blocked; list both. Otherwise use `BLOCKED` when any required path is blocked.
Never report `PASSED` from code inspection alone or when authentication,
fixture, URL, or a required path was unavailable.

## Report evidence

Return a concise report containing:

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

## Europe GitHub and Linear comments

Before publishing comments, re-read the remote PR head SHA. If it changed,
refresh context and rerun affected paths before writing.

Post the same French Markdown body as a GitHub PR comment and a Linear comment:

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

Use `Aucun problème détecté.` under `Problèmes détectés` when validation
passes. Keep the comment concise and safe for humans; do not include secrets,
tokens, personal data, raw logs, or long traces.
