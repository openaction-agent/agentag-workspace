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

- Follow the root `AGENTS.md`, the main reference for browser validation (including the result statuses); this skill
  only adds the preproduction workflow. Use the `playwright-cli` skill for browser commands.
- Stay on the supplied host and the same-environment redirects or sibling apps reached through its navigation; never
  follow a link whose environment is uncertain.
- Do not modify code, repositories, Linear issues, or PRs.
- Log in only on the supplied preproduction console, as `agent@openaction.eu` with the password stored in
  `/root/openaction-preprod-password.txt`.
- Send email only to `agent@openaction.eu`. Do not export CRM data, pay, publish content, or contact real users.

## Run

1. Turn the plan into ordered paths, each with its starting state, fixture, actions, expected visible result, and
   cleanup, keeping the user's order and data. With the default plan, adapt labels to the UI but skip no required
   path.
2. Find existing fixtures through read-only UI inspection; never assume a record exists. Mark a path `BLOCKED` when a
   module, permission, fixture, safe recipient, or product decision is missing.
3. Use a run ID such as `VP-YYYYMMDD-HHMMSS` in every created record, email subject, and kept artifact.
4. Open the exact URL in a session named after the run (for example `validate-preprod-YYYYMMDD-HHMMSS`), capture the
   starting URL and a snapshot, log in through the form, and check the identity and environment.
5. Run the paths in order:
   - for searches and filters, check both the narrowed results and the reset, not just the controls;
   - for email, check the exact recipient before sending, put the run ID in the subject, and verify the visible
     queued or sent confirmation. Claim inbox delivery only if the plan provides mailbox access and it was checked.
6. Clean up run-owned data through the UI once confirmed. A failed cleanup does not change the functional result but
   is reported as needing manual action.
7. Remove any sensitive local artifact.

## Result

Use the statuses of the root `AGENTS.md`, plus `NOT APPLICABLE` when the UI clearly shows a path is outside this
environment's product scope; missing access or an unexpected absence is `BLOCKED`.

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
