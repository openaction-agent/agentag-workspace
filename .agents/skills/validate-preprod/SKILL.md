---
name: validate-preprod
description: "Validate an OpenAction preproduction environment from a supplied URL, with an optional user-provided test plan. Use when asked to test, QA, smoke-test, recette, or validate a preprod/staging environment independently of a Pull Request or Linear issue. Execute the supplied journey when present; otherwise run the bundled critical-path console test plan covering authentication, navigation, CRM access, contact search and filters, contact lifecycle, and a safe email send."
---

# Validate an OpenAction Preproduction Environment

Validate a preproduction URL through its visible UI and return a concise, evidence-backed result. No PR, repository,
or Linear issue is involved, and nothing is published or changed in external workflows.

## Inputs

A preproduction URL is required. The user may also give a test plan (in the prompt or as an attachment) and specific
fixtures, expected results, apps, or exclusions.

A supplied plan is authoritative: add only the smoke checks needed to log in, reach the feature, and catch an obvious
adjacent regression. Without a plan, run [the default critical-path plan](references/default-test-plan.md) in full.
Ask only when the URL is missing or the plan has an ambiguity the UI cannot resolve safely.

## Rules

- Follow the root `AGENTS.md`.
- Use the `playwright-cli` skill for every browser operation. If `command -v playwright-cli` fails, report `BLOCKED`;
  do not install it or use another tool.
- Stay on the supplied host and the same-environment redirects or sibling apps reached through its navigation. Never
  derive a target, go to production, or follow a link whose environment is uncertain.
- Use visible UI controls only, without JavaScript evaluation, storage edits, request mocking, or API calls.
- Do not modify code, repositories, Linear issues, PRs, or deployments.
- Log in only on the supplied preproduction console, as `agent@openaction.eu` with the password stored in
  `/root/openaction-preprod-password.txt`. Keep credentials out of messages, reports, screenshots, snapshots, traces,
  filenames, and saved authentication state, and do not persist a browser profile.
- Send email only to `agent@openaction.eu`. Do not export CRM data, pay, publish content, or contact real users.
- Never alter or delete pre-existing records. Create only uniquely named, synthetic test data, and delete it only
  after capturing evidence and getting the user's confirmation; otherwise keep it and report its identifier.
- Stop any action whose scope, recipient, environment, or effect is uncertain, and mark its path `BLOCKED`.

## Run

1. Turn the plan into ordered paths, each with its starting state, fixture, actions, expected visible result, and
   cleanup, keeping the user's order and data. With the default plan, adapt labels to the UI but skip no required
   path.
2. Find existing fixtures through read-only UI inspection; never assume a record exists. Mark a path `BLOCKED` when a
   module, permission, fixture, safe recipient, or product decision is missing.
3. Use a run ID such as `VP-YYYYMMDD-HHMMSS` in every created record, email subject, and kept artifact.
4. Open the exact URL in one isolated in-memory named session (for example `validate-preprod-YYYYMMDD-HHMMSS`),
   capture the starting URL and a snapshot, log in through the form, and check the identity and environment.
5. Run the paths in order, verifying the visible state after each meaningful action:
   - for searches and filters, check both the narrowed results and the reset, not just the controls;
   - for created or edited data, check persistence after navigation or reload;
   - for email, check the exact recipient before sending, put the run ID in the subject, and verify the visible
     queued or sent confirmation. Claim inbox delivery only if the plan provides mailbox access and it was checked.
6. After each path, check `playwright-cli console` and `playwright-cli requests`, correlate errors with actions, and
   tell environment noise from real defects.
7. Keep concise evidence, preferably snapshots and targeted screenshots, without secrets or unneeded personal data.
8. Ask before cleaning up run-owned data; if confirmed, clean up through the UI. A failed cleanup does not change the
   functional result but is reported as needing manual action.
9. Close the session and remove any sensitive local artifact.

## Result

- `PASSED`: every required action ran on the preprod, outcomes matched, and no relevant console or network defect
  appeared.
- `FAILED`: an outcome is wrong, a regression is reproduced, or a console error or failed request shows a defect.
- `BLOCKED`: a required URL, module, fixture, authorization, safe action, or browser capability is unavailable.
- `NOT APPLICABLE`: the UI clearly shows the path is outside this environment's product scope. Missing access or an
  unexpected absence is `BLOCKED`.

Overall, any failed path means `FAILED`; otherwise any blocked required path means `BLOCKED`; `PASSED` requires every
required path to pass. Never infer success from code or page reachability.

Reply following the workspace answer-language rule:

```markdown
Status: PASSED | FAILED | BLOCKED

- Target: <preproduction URL>
- Journey: <user-provided | default critical-path plan>
- Run ID: `<identifier>`
- Cleanup: <completed | retained records and reason | not needed>

| Path | Fixture | Actions and visible result | Console/network | Status |
| --- | --- | --- | --- | --- |
| <name> | <safe fixture> | <concise evidence> | <relevant evidence> | <status> |

Defects or blockers: <reproduction, impact, and missing evidence>
Email: <recipient redacted as authorized test mailbox, subject/run ID, UI send result>
Artifacts: <safe absolute paths to retained screenshots or traces>
```

Leave out credentials, tokens, contacts' personal data, raw logs, and long traces, and say clearly when an email was
queued but its delivery was not checked.
