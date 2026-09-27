---
name: review-pr
description: "Review an OpenAction change from either a Linear issue ID or a GitHub pull request number or URL. Use when asked to review, analyze, or check an issue's implementation or PR: resolve the linked issue and repository, inspect the code and GitHub context, run relevant validation, publish actionable findings on GitHub, and return a concise summary."
---

# Review a Linear Issue or Pull Request

Review one implementation in depth, publish the findings as a GitHub review, and summarize the result.

The input is a Linear issue ID (`OPE-123`), a PR number (`#123`, optionally with `owner/repo`), or a PR URL. The
review mode follows from the findings and cannot be overridden.

## Rules

- Follow the root `AGENTS.md`, the main reference for Linear, forges, Git, testing, and CI; this skill only adds
  the review workflow and format.
- Do not modify or push the implementation unless the user separately asks for fixes.

## Workflow

1. Resolve and read the full Linear issue, the PR, and the repository.
2. Record the base revision and head SHA.
3. Inspect code outside the diff as needed: call sites, contracts, migrations, permissions, and patterns.
4. Analyze the change with `references/review-criteria.md`, favoring product-impacting findings over style.
5. Run the focused tests and static checks needed for confidence.
6. Verify each finding against the head revision, cite `path:line` for code findings, and skip issues already raised
   and resolved.
7. Re-read the PR head SHA; if it changed, revalidate every finding.
8. Submit exactly one GitHub review, with inline comments where they make a fix clearer: `request-changes` if there is
   at least one `Blocker`, otherwise `comment`. Never approve.
9. Reply with the review mode, finding counts, validation performed, PR link, and any blocker, without pasting the
   review.

## Review format

```markdown
## Summary
<1-3 lines describing the implementation and overall assessment>

## Findings
### Blocker
- [`path:line`] <problem, impact, and concrete remediation>

### Important
- [`path:line`] <problem, impact, and concrete remediation>

### Nit
- [`path:line`] <optional improvement>

## Questions
- <clarification needed to establish correctness>

## Positive notes
- <effective design, test, or clarity choice>
```

Skip empty sections. With no actionable issue, write `No actionable findings.` under `Findings`. Separate verified
defects from questions, and review the change, not its author.
