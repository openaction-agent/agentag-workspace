---
name: openaction-instance-operations
description: Operate a live OpenAction instance through its configured MCP server. Use for requests to retrieve, search, create, update, or administer instance data or configuration, including CRM contacts, segments, tags, emails, campaigns, forms, content, events, payments, automations, analytics, users, roles, and settings; also use for information that may exist only in an OpenAction instance.
---

# OpenAction Instance Operations

Use the OpenAction MCP tool surface first for live product state, statistics,
and supported product actions. Prefer it over repositories, browser automation,
or other fallbacks when it exposes the required operation. This skill
complements the global OpenAction MCP policy in `AGENTS.md`.

## Workflow

1. Inspect the OpenAction MCP tools currently exposed to the agent. Read the
   relevant tool description and input schema before choosing an operation. If
   the MCP provides a catalog/discovery operation, use it before deciding that
   an operation is unavailable.
2. Resolve the target server. Treat these as user-facing aliases:

   - `Place Publique` / `pp` → Place Publique;
   - `Les Écologistes` / `ecolos` → Les Écologistes;
   - `L'Après` / `lapres` → L'Après;
   - `Europe` / `europe` → Europe.

   Map the alias to the configured server's displayed name and endpoint, not
   to a hard-coded tool prefix. If multiple OpenAction instances are available
   and the user did not name one, ask them which instance to use before calling
   an instance tool. Do not infer the target from a repository or earlier work.
3. Start with the narrowest safe operation. Answer direct status questions and
   simple counts, totals, averages, rates, or comparisons through a relevant
   read/statistics operation. For a write, resolve and show the intended target
   internally before performing it; use idempotent actions or a single retry
   when the MCP supports them. A read-only request never authorizes a write.
4. Perform only explicit, in-scope writes. Confirm destructive, externally
   consequential, and scope-expanding actions, as well as material ambiguity
   in a bulk operation. Never transfer or merge data between instances unless
   the user explicitly directs it. Apply the same rule to organizations within
   an instance: never copy, transfer, merge, or treat data across organizations
   without explicit user direction.
5. Verify a successful change with a focused read when the MCP permits it.
   Report the selected instance, action or query scope, and concise outcome.
6. If no suitable MCP operation exists after exploring the available tool
   surface, use the UI or another permitted interface as a fallback and report
   both the limitation and the fallback used.

## Confirmed MCP surface

All configured OpenAction instances expose the same direct typed operations,
rather than a catalog tool. Inspect the selected operation's schema, then use
the matching family:

- access and project configuration: `access_read`, `projects_read`,
  `projects_change`;
- CRM: `contacts_read`, `contacts_change`, `crm_configuration_read`,
  `crm_configuration_change`, `duplicates_read`, `duplicates_change`;
- campaigns: `campaigns_read`, `campaigns_change`, `campaigns_delete`,
  `campaigns_delivery`, `campaigns_results`;
- payments: `payments_read`, `payments_change`, `payments_export`,
  `payment_filters_read`, `payment_filters_change`;
- website: `site_content_read`, `site_content_change`,
  `site_categories_read`, `site_categories_change`, `statistics_read`;
- documents and team: `documents_read`, `documents_change`, `team_change`.

## High-impact operations

The `*_change` families combine several sub-actions. Inspect the requested
sub-action before executing it; a harmless update and a deletion may share the
same tool. Ask for confirmation before operations that delete, merge,
transfer, refund, terminate, cancel, or otherwise irreversibly alter records.

For `campaigns_delivery`, confirm any missing delivery or scheduling detail
and the recipient scope before a send. Treat `campaigns_results`,
`payments_export`, `site_content_read` exports, and document downloads as
potentially personal-data-bearing: retrieve only what is necessary and return
a concise summary unless the user explicitly asks for the export itself.

## Boundaries

- Do not use `codex mcp list` as the operational interface. It can diagnose
  configuration but does not replace the MCP tools available in the session.
- Do not expose credentials, OAuth state, raw tokens, or unnecessary personal
  data in the response.
- Do not replace an MCP operation with repository inspection, a database
  assumption, or browser automation until the available MCP capability has
  been explored.
- If no matching OpenAction MCP is exposed in the current session, report that
  limitation and use another permitted interface only when it can perform the
  requested operation safely.
