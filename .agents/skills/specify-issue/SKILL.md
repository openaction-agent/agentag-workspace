---
name: specify-issue
description: Prepare and save an implementation-ready technical specification for one or more Linear issues. Use when asked to specify, scope, or technically analyze Linear issue IDs, including reading issue discussions and the relevant OpenAction codebase, updating the issue body, and moving it to "Spec to review".
---

# Specify Linear Issues for Implementation

Turn one or more Linear issues (`OPE-123` or `OPE-123,OPE-456`) into implementation-ready specifications saved in the
issue body. Ask only when an issue or repository cannot be resolved unambiguously.

## Rules

- Follow the root `AGENTS.md` and the repository's instructions.
- This workflow is read-only for code: do not edit, run tests, commit, or push. Preserve unrelated local changes.
- Stop and report if Linear or GitHub access is not enough to read and update every issue.

## Workflow

For each issue:

1. Read the title, body, status, labels, links, attachments, and all comments; later clarifications win.
2. Move the issue to `Specification in progress`.
3. Resolve the repository and fetch the latest `main` without discarding local work.
4. Inspect the code on `origin/main`: the affected paths, adjacent behavior, tests, migrations, configuration, and
   conventions.
5. Write a concise English specification with the template below. It complements the issue: add implementation
   facts and decisions (code paths, boundaries, contracts, constraints, risks, edge cases, focused tests) without
   repeating the issue's problem, requested behavior, acceptance criteria, or discussion. Ask the user only when a
   decision materially changes scope or product behavior. Keep lines near 80 characters.
6. Re-read the issue and code to verify every file reference and technical claim.
7. Keep the existing issue body. Replace a previously generated specification, or append the new one in a fenced
   Markdown block. Do not post it as a comment.
8. Once the body update succeeds, move the issue to `Spec to review`.

Reply with the issue links, repository, important decisions, and open questions.

## Specification template

Use the verified `origin/main` commit as the baseline. `Implementation design`, `Validation`, and `Implementation
context` are required; include the other sections only when they add concrete information.

```markdown
# Implementation design

* `<component or layer>`: <intended change and why it fits the existing
  architecture>
* <Describe required data flow, state transitions, contracts, and invariants.
  Name likely files or symbols when verified, without prescribing incidental
  low-level edits.>

# Impacts and constraints

* <Describe affected API/events, persistence, migrations, permissions,
  security/privacy, performance, observability, compatibility, deployment, or
  rollback constraints. Include only relevant items.>

# Edge cases

* <State a concrete boundary, failure mode, concurrency/idempotency case, or
  legacy-data condition and the expected implementation behavior.>

# Validation

* `<individual test file or test case>`: <specific scenario and expected
  assertion to add or update>
* `<focused command>`: <what it validates; include only commands verified for
  this repository>
* <Mention Coolify preview validation only for `citipo/openaction-europe`.>

# Scope and open decisions

* Out of scope: <adjacent change that should deliberately remain untouched>
* Decision: <non-obvious technical choice and rationale>
* Open question: <unresolved implementation uncertainty, impact, and who or
  what can resolve it>

# Implementation context

Repository: <owner/repository>

* `<path>` — `<symbol, module, route, schema, or test>`: <verified current
  behavior, dependency, convention, or invariant relevant to this change>
* <Describe the current execution/data flow only as needed to explain the
  implementation design.>
```

Leave out the generic branch, PR, CI, and Linear workflow, which the `implement-issue` skill owns, and never prescribe
a full test suite. The specification must be grounded in the current code, separate verified behavior from proposals
and open questions, and contain no sensitive or irrelevant details.
