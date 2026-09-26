# Default Critical-Path Test Plan

Use this plan when the user supplies none. Run every required path in order. Stay read-only until the contact
lifecycle path creates a synthetic record owned by the run.

## Test data

- One run ID: `VP-YYYYMMDD-HHMMSS`.
- Email recipient: `agent@openaction.eu` only.
- Synthetic contact email: `agent+<run-id-lowercase>@openaction.eu` if the form accepts it; otherwise the authorized
  mailbox, with the run ID in an external ID, tag, or name field.
- Synthetic name: first name `Validation`, last name `<run-id>`.
- No real person's name, address, phone number, membership, payment, or political data.

## 1. Reachability and login (required)

1. Open the URL and check that it is clearly a non-production environment.
2. Check that the login form works without an application error.
3. Log in with the credentials defined in `SKILL.md`.
4. Check the authenticated landing page, the expected account or organization, and the absence of a login loop.
5. Reload once and check that the session persists.

## 2. Main navigation (required)

1. Open the dashboard or home, CRM/contacts, and email/campaigns modules from the main navigation, then return to the
   starting module.
2. Check that each page renders its heading or main content and the navigation remains available.

Expected: no broken routing, blank screen, or relevant console or network failure.

## 3. CRM list (required)

1. Open the CRM or contacts module.
2. Check the contact list, its columns or cards, the result count, pagination or scrolling, and the search and filter
   controls.
3. Open one existing contact read-only, note only a non-sensitive identifier, and return without editing.

If there are no contacts, record it and continue with the synthetic contact path.

## 4. Contact search (required)

1. Search for the run's synthetic contact if it exists, otherwise for a visible non-sensitive value from the list, and
   check that the results match.
2. Search for `NORESULT-<run-id>` and check for a clear empty state.
3. Clear the search and check that the original list or count returns.

## 5. CRM filters (required when filters exist)

`BLOCKED` if the CRM should offer filters but they are missing.

1. Apply one low-risk criterion (status, tag, subscription, creation date, organization) whose effect can be checked
   on visible rows or the count, and check every visible row matches.
2. Combine it with a second criterion when available and check the results stay consistent.
3. Remove one criterion and check the other persists.
4. Clear all filters and check the baseline returns.
5. If the UI is expected to persist filters, reload with a harmless filter and check the behavior.

Evidence must come from rows, counts, or an empty state, not from filter chips alone.

## 6. Synthetic contact lifecycle (required when contact creation is available)

`NOT APPLICABLE` if the module is intentionally read-only; `BLOCKED` if permissions or behavior are unclear.

1. Check that the form rejects a missing required field, then create the contact with the synthetic data and the
   minimum required fields.
2. Open it and check the saved values.
3. Find it in the CRM list by searching its run ID.
4. Edit one harmless field (first name or a test tag), save, reload, and check persistence.
5. Do not merge, subscribe, donate for, or otherwise affect an existing contact.

## 7. Email sending (required when the email module and an authorized sender are available)

1. Start a new email or campaign draft named `Validation preprod <run-id>`, with that subject and a short plain-text
   body saying it is an automated preproduction test that can be ignored.
2. Select only the authorized mailbox as recipient, preferably with a direct test-email feature. If only audience
   sending exists, use a segment shown in the UI to contain exactly the run's contact and nobody else.
3. Before sending, check that the final recipient count is exactly one authorized address; otherwise stop and mark
   the path `BLOCKED`.
4. Send, then check the visible queued, sent, or accepted confirmation and the draft status. If the environment routes
   mail to a visible mail sink, check it too; otherwise do not claim delivery.

## 8. Cleanup (after user confirmation)

1. Capture the final evidence first, then ask the user to confirm deleting the run's records. Without confirmation,
   keep them and report their identifiers.
2. Once confirmed, delete or archive only the run's email draft or campaign, when this is clearly safe and keeps the
   send evidence, and only the run's synthetic contact, when deletion is scoped to that record.
3. Never delete a pre-existing contact, shared audience, template, or campaign. If safe cleanup is not possible, keep
   the data and report its identifiers.

A cleanup problem alone does not fail the validation.

## 9. Logout (required)

1. Log out through the UI and check the return to an unauthenticated page without error.
2. Close the browser session and check that no authentication state or persistent profile remains.
