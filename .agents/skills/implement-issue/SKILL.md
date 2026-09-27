---
name: implement-issue
description: "Implement an OpenAction Linear issue end to end. Use when given a Linear issue ID and asked to build, fix, or deliver it: read the full issue and any technical specification, inspect the relevant repository, implement and test the change, open a GitHub pull request, update Linear, and report the result."
---

# Implement a Linear Issue

Take one Linear issue, such as `OPE-123`, to a review-ready pull request. Ask only when the issue, repository, or a
product decision cannot be resolved unambiguously.

## Rules

- Follow the root `AGENTS.md`, the main reference for Linear, forges, Git, testing, and CI; this skill only adds
  the issue-to-PR workflow.

## Workflow

1. Read the whole issue, then resolve the repository and any linked PR.
2. If the issue body contains a technical specification, use the latest one as implementation guidance; newer
   comments and the current code override stale details, and material divergences must be disclosed. Otherwise,
   derive the smallest safe implementation from the issue and the code, using the `specify-issue` skill as a
   reference, and ask only about ambiguities that change product behavior or scope.
3. Move the issue to `Implementation in progres`.
4. Create a branch from the latest `main` following the repository's naming convention.
5. Implement the complete change, following existing patterns, including migrations, permissions, translations,
   observability, or compatibility work when relevant.
6. Add or update tests, run the focused tests and checks for the change, and fix in-scope failures.
7. Prepare the preview test journeys (see below).
8. Review the diff for correctness, security, data safety, unintended scope, generated artifacts, and secrets, and
   reconcile it with the issue and specification.
9. Commit with the issue ID, push, and open a PR (not a draft) whose title contains the issue ID and whose body
   follows the format below.
10. Follow CI to a pass.
11. Run the `review-pr` skill on the PR. Fix every verified `Blocker` and `Important` finding, push, follow CI to a
    pass, and repeat until none remain. `Nit` findings do not block unless they reveal product risk.
12. Post one Linear comment in the format below.
13. Move the issue to `To validate` and reply with the issue and PR links, delivered behavior, tests run, CI state,
    and any follow-up.

## Preview test journeys

Each journey must be executable by the validation agent: the app, the fixture account and its rights, the navigation,
the UI actions, and the visible expected result. For each prerequisite record, give either a fixture or record
verified in the current preview configuration, or complete safe UI steps to create it during the validation run
(entity, starting page, every required field and relation, a unique non-sensitive value, and the visible
confirmation). Derive them from the current code and fixtures; never invent data, rely on undocumented shared records,
or point to production data. Do not create preview data yourself for this handoff.

## GitHub PR body

Write exactly these three sections, with nothing before or after:

````markdown
## Résumé fonctionnel

<En français non technique, résumer en quelques phrases l'objectif de l'issue
Linear et l'approche fonctionnelle mise en œuvre. Écrire pour le client final,
sans noms de classes, fichiers, commandes ou détails internes.>

## Parcours de test sur la prévisualisation

### <Nom du parcours>

1. Dans `<application>`, se connecter avec le compte de prévisualisation
   `<fixture, rôle et organisation vérifiés>`.
2. Préparer les données nécessaires à cette exécution :
   - <navigation UI exacte pour créer `<entité>`> ;
   - <valeurs exactes de chaque champ, relation et état requis> ;
   - <valeur non sensible et unique, par exemple
     `Validation OPE-123 <horodatage>`> ;
   - <résultat visible qui confirme la préparation>.
3. Accéder à `<navigation précise dans l'interface>`.
4. Effectuer `<interactions UI précises>`.
5. Vérifier `<résultat fonctionnel visible attendu>`.

<Remplacer l'étape 2 par la fixture ou le record existant seulement s'il a été
vérifié dans la configuration de prévisualisation. Ajouter uniquement les autres
parcours nécessaires.>

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
- <relevant CI state>

### Risks and continuation

- Risk: <remaining risk, limitation, or operational concern>
- Continue with: <specific context and next action for future work>
```
````

The first two sections are concise, non-technical French. The third is English and records every relevant decision,
approach, rationale, deviation, validation result, risk, and continuation detail for future work; omit empty
subsections but keep all technical content inside its code block. Wrap every line of that block at 80 characters or
less; this check must print nothing:

```bash
awk '
  /^```markdown$/ { in_block=1; next }
  in_block && /^```$/ { in_block=0 }
  in_block && length($0) > 80 { print NR ":" length($0) }
' pr-body.md
```

## Linear comment

Copy the first two PR sections verbatim, with nothing else.

## Before handing off

Check that the change satisfies the latest accepted scope; tests cover the important success, edge, permission, and
failure paths; the latest required CI passed; the branch has no unrelated or sensitive edits; the PR body has only its
three sections within 80 columns in the raw block; every preview journey is executable as described above; and the
Linear comment matches its format.
