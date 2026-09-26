# Review Criteria

Start from the accepted behavior in the linked Linear issue and its latest comments, and check both the changed lines
and the surrounding code.

## 1. Functional correctness

- The change solves the problem stated in the PR and the Linear issue.
- Control flow, null/empty handling, and boundary conditions are correct.
- Adjacent behavior and call sites do not regress.
- Migrations and configuration changes are safe for existing environments.

## 2. Security and data safety

- Injection, unsafe deserialization, and authorization or permission gaps.
- Handling of secrets, tokens, and personal data in code and logs.
- Input validation and output escaping.
- Dangerous defaults, overly broad permissions, and missing safeguards.

## 3. Performance and scalability

- N+1 queries, unbounded loops, and repeated expensive work.
- Network, file, or database calls in hot paths.
- Caching and pagination when data volume can grow.
- Algorithmic complexity changes.

## 4. Reliability and operations

- Failure paths, retries, and timeouts.
- Idempotency of jobs and commands that may re-run.
- Useful logs, metrics, and error context.
- Health, readiness, or background task behavior when relevant.

## 5. Readability and maintainability

- Clear naming, structure, and abstractions; no duplicated logic or hidden coupling.
- No unnecessary few-line private methods: prefer inline logic unless extraction clearly improves reuse or
  readability.
- Backward compatibility of public contracts and interfaces.
- Docs and comments updated for non-obvious behavior.

## 6. Testing

- Tests cover happy paths, edge cases, and failure paths, with meaningful assertions that are not overly tied to the
  implementation.
- Missing tests are called out when the risk is non-trivial.
- CI has no failing or skipped critical jobs.

## Severity

- Blocker: must be fixed before merge; risk of bugs, data loss, or security issues.
- Important: should be fixed before merge; high-confidence quality risk.
- Nit: optional polish with no material product risk.
- Question: clarification needed before confidence is high.

Do not inflate severity. Report a finding only when a concrete execution path supports it; otherwise ask a question.
