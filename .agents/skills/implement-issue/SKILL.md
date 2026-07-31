---
name: implement-issue
description: "Implement an OpenAction Linear issue end to end. Use when given a Linear issue ID and asked to build, fix, or deliver it: read the full issue and any technical specification, inspect the relevant repository, implement and test the change, open a GitHub pull request, update Linear, and report the result."
---

# Implement a Linear Issue

Implement one Linear issue through a review-ready pull request. Use an existing
technical specification in the issue as the implementation workflow when one
is present.

## Inputs

Require one Linear issue ID, such as `OPE-123`. Ask only when the issue,
repository, or a product decision cannot be resolved unambiguously.

## Operating rules

- Follow the testing, Git, and CI policy in the workspace root `AGENTS.md`. It
  is authoritative for focused local validation, every push, CI timing, and
  failed-job log access.
- Follow the repository's `AGENTS.md` and local contribution instructions.
- Run from the relevant repository checkout. Preserve unrelated changes, use
  an isolated worktree when the current checkout is not clean, and never push
  directly to `main`.
- Use the GitHub and Linear access available in the local environment. Require
  enough access to read and update the issue, create or update the PR, and
  inspect CI; stop and report a missing capability that prevents completion.

## Workflow

1. Read the issue's title, body, status, labels, links, attachments, and all
   comments. Resolve the relevant repository and any linked pull request.
2. Locate the latest technical specification in the issue body. When present,
   use its implementation context, design, impacts, edge cases, validation,
   scope boundaries, and decisions as issue-specific guidance. Let newer issue
   comments and the current code override stale details; disclose any material
   divergence.
3. When no specification exists, derive the smallest safe implementation from
   the issue and current code. Ask about ambiguity only when it changes product
   behavior or scope. Use the `specify-issue` skill as reference on how to
   do the specification.
4. Move the issue to `Implementation in progres`.
5. Fetch the latest remote `main`, verify the worktree state, and create a
   branch from it using the repository's branch naming convention.
6. Inspect existing patterns and focused tests before editing. Implement the
   complete change, including migrations, permissions, translations,
   observability, or compatibility work when relevant.
7. Add or update appropriate tests. Run only the individual test files or test
   cases involved in the issue, plus narrowly scoped formatters or static
   checks. Never run a full local suite. Fix in-scope failures.
8. Design the validation handoff from the implemented behavior, current source,
   and preview configuration. For every required path, identify the preview
   app, fixture account and permission, and each prerequisite record. Use a
   verified existing fixture only when it is sufficient; otherwise write exact
   safe UI instructions for the validation agent to create run-owned temporary
   data. Do not create test data during implementation merely to support
   preview validation, and never claim that a record exists when it was not
   verified.
9. Review the diff for correctness, security, data safety, unintended scope,
   generated artifacts, and accidental secrets. Reconcile it with the issue
   and specification.
10. Commit with the issue ID, push the feature branch, and open a draft GitHub
   PR. Include the issue ID in its title and use only the exact PR body format
   below. Monitor that pushed head using the workspace CI policy before
   continuing.
11. Use the `review-pr` skill on the draft PR before requesting human review.
    Fix every verified `Blocker` and `Important` issue, rerun the focused
    checks that cover those fixes, commit and push the corrections, monitor
    each new head using the workspace CI policy, and repeat the review loop
    until no `Blocker` or `Important` finding remains. Do not block readiness
    on `Nit` findings unless they reveal product risk.
12. Confirm required CI has succeeded for the latest head using the workspace
    policy. Diagnose failures through its bounded GitHub MCP run, job, and log
    workflow; fix in-scope problems and repeat after every push. For
    `citipo/openaction-europe` only, also wait for the Coolify bot PR comment
    and verify preview URLs using the rules below.
13. When required checks pass, and when Europe preview URLs are verified if the
    repository is `citipo/openaction-europe`, mark the PR ready. Link the PR and
    post exactly one Linear comment using the format below.
14. Move the issue to the `To validate` status. Do not assume an automation-specific
    queue. Return a concise summary with issue and PR links, behavior delivered, tests 
    run, CI state, Europe preview state only when applicable, and any follow-up. For 
    non-Europe repositories, do not mention preview URLs or wait for them.

## GitHub PR body

Write exactly these three top-level sections in this order. Add no preamble,
issue metadata, link list, test section, generated footer, or other text. Put
all technical and validation details inside the third section's code block.

````markdown
## Résumé fonctionnel

<En français non technique, résumer en quelques phrases l'objectif de l'issue
Linear et l'approche fonctionnelle effectivement mise en œuvre. Écrire pour le
client final, sans noms de classes, fichiers, commandes ou détails internes.>

## Parcours de test sur la prévisualisation

### <Nom du parcours>

1. Dans `<application>`, se connecter avec le compte de prévisualisation
   `<fixture, rôle et organisation vérifiés>`.
2. Préparer les données nécessaires à cette exécution :
   - <navigation UI exacte pour créer `<entité>`> ;
   - <valeurs exactes de chaque champ, relation et état requis> ;
   - <valeur non sensible et unique à générer, par exemple
     `Validation OPE-123 <horodatage>`> ;
   - <résultat visible qui confirme que la préparation a réussi>.
3. Accéder à `<navigation précise dans l'interface>`.
4. Effectuer `<interactions UI précises>`.
5. Vérifier `<résultat fonctionnel visible attendu>`.

<Remplacer l'étape 2 par la fixture ou le record existant exact seulement
lorsqu'il a été vérifié dans la configuration de prévisualisation. Ajouter
uniquement les autres parcours nécessaires. Pour chacun, indiquer
l'application, le compte et ses droits, la préparation ou la fixture vérifiée,
la navigation, les interactions et le résultat attendu.>

## Implementation context

```markdown
### Implementation decisions and approach

- Decision: <decision and implementation details>
- Approach: <architecture, data flow, contracts, and relevant code anchors>
- Rationale: <why this approach was selected>

### Deviations

- <deviation from the Linear issue or specification, with rationale>

### Validation

- <test, formatter, or manual check run and its result>
- <relevant CI state, and Europe preview state only when applicable>

### Risks and continuation

- Risk: <remaining risk, limitation, or operational concern>
- Continue with: <specific context and next action for future work>
```
````

Copy the first two sections verbatim into the Linear comment. Keep their
content concise, non-technical, and in French. Make every preview workflow
executable by the validation agent: name the app, navigation path, UI actions,
fixture account and required rights, and visible result. For each prerequisite,
provide either an exact fixture or test record verified in the current preview
configuration, or a complete safe UI procedure to create it for that validation
run. A creation procedure must name the entity, starting navigation, all
required field values and relations, a non-sensitive unique value, and the
visible confirmation that setup succeeded. Derive those instructions from the
current code, fixtures, and implementation; never invent data, rely on an
undocumented shared record, or instruct the validator to use production data.
Do not create preview test data during implementation solely for this handoff.
For other repositories, name only verified safe data or the same explicit
creation procedure and do not reference unavailable Coolify preview URLs.

Write the third section in English for future implementation and review work.
Record every relevant implementation decision, approach, rationale, technical
detail, deviation, validation result, risk, and continuation detail. Omit empty
internal subsections, but do not move technical content outside the raw
Markdown code block. Wrap every line inside that block to at most 80
characters, including commands and code paths. Check the prepared body before
publication; the following command must print nothing:

```bash
awk '
  /^```markdown$/ { in_block=1; next }
  in_block && /^```$/ { in_block=0 }
  in_block && length($0) > 80 { print NR ":" length($0) }
' pr-body.md
```

## Europe Coolify preview URLs

Apply this section only when the resolved repository is
`citipo/openaction-europe`. For every other repository, skip it entirely: do
not wait for Coolify, do not look for missing preview URLs, do not keep the PR
in draft because URLs are absent, and do not include preview URLs in the PR,
Linear, or final summary.

For Europe, read the Coolify bot comment on the current PR and use only URLs
explicitly listed there for that PR and head commit. Match each URL to its named
app and verify that it is reachable. Never derive, copy, or guess a URL.
Require verified URLs for `console`, `public`, `platform`, and `mobilisation`.
If any is missing, ambiguous, unreachable, or not tied to the PR, keep the PR
in draft and Linear `Implementation in progres`, do not post the Linear
comment, and report the missing evidence. Never claim Europe completion with
incomplete or inferred URLs.

## Linear comment

For `citipo/openaction-europe`, post exactly these three top-level sections in
this order. Copy the first two sections and their content verbatim from the PR.
Add no technical summary, issue metadata, CI details, status note, footer, or
other commentary.

```markdown
## Résumé fonctionnel

<Copie exacte de la première section de la PR>

## Parcours de test sur la prévisualisation

<Copie exacte de la deuxième section de la PR>

## Prévisualisations Coolify

- Console : <URL de prévisualisation vérifiée>
- Public : <URL de prévisualisation vérifiée>
- Platform : <URL de prévisualisation vérifiée>
- Mobilisation : <URL de prévisualisation vérifiée>
```

For every other repository, post only the first two sections, copied verbatim
from the PR. Do not add `Prévisualisations Coolify`, an empty replacement
section, or any preview URL note.

## Quality check

Before handing off, verify that:

- the implementation and specification satisfy the latest accepted scope;
- focused tests cover important success, edge, permission, and failure paths;
- every pushed head was handled under the workspace CI monitoring policy, and
  the latest required CI is successful;
- the branch has no unrelated or sensitive edits;
- the PR body has only its three sections and its raw block stays within 80
  columns;
- every required preview path identifies a verified fixture or a complete,
  safe, run-owned data-creation procedure for the validation agent;
- for Europe, the Linear comment has only the two copied French sections and
  four verified Coolify URLs;
- for non-Europe repositories, Linear, PR, and final output does not wait for
  or mention preview URLs.
