# Pull Requests & Code Review

Rule IDs: `REV-*`. Severity: **error** unless marked otherwise.

## REV-1 — Keep pull requests small

Target under 400 changed lines of substantive diff. Past that, review quality collapses and approval
becomes rubber-stamping. Split large work into a stack: refactor first, behavior second.

Generated files, lockfiles, and pure formatting changes do not count toward the budget, but must be
isolated in their own commits so reviewers can skip them.

## REV-2 — The description answers four questions

1. **What** changed — one or two sentences.
2. **Why** — the problem, the ticket, the user impact.
3. **How to verify** — the exact commands, URLs, or steps a reviewer runs.
4. **Risk** — what could break, and how to roll back.

A PR body that only restates the title is incomplete.

## REV-3 — Self-review before requesting review

Read the full diff as a reviewer would before assigning anyone. Remove debug statements, stray
`TODO`s without tickets, commented code, and accidental whitespace churn. Leave inline comments on
the PR explaining any decision a reviewer would otherwise question.

## REV-4 — CI must be green before review is requested

Do not assign reviewers to a red build. Their time is spent on logic, not on failures CI already reports.

## REV-5 — Reviewer response within one business day

Acknowledge within one business day, even if the acknowledgment is "I can look at this tomorrow
afternoon." Blocked PRs age badly and accumulate merge conflicts.

## REV-6 — Comment severity is explicit

Prefix every review comment so the author knows what is required:

- **blocking:** — must be addressed before merge
- **suggestion:** — improvement the author may take or decline
- **nit:** — trivial, never blocks merge
- **question:** — seeking understanding, not requesting change
- **praise:** — worth saying out loud

An unlabeled comment defaults to *suggestion*.

## REV-7 — Review the change, not the person

Comment on code: "this allocates on every iteration", not "you always do this". State the problem
and, where possible, a concrete alternative. Approve when the code is better than what was there —
not when it matches how the reviewer would have written it.

## REV-8 — Disagreements escalate, they do not stall

Two rounds without convergence means a synchronous conversation or a third opinion. Record the
outcome in the PR thread so the reasoning survives. Never resolve a disagreement by merging past it.

## REV-9 — The author merges

The author merges once approved and green — they know whether anything else is pending. Delete the
branch on merge.

## REV-10 — Approvals expire on substantive change

Pushing new logic after approval requires re-review. Rebases, comment fixes, and typo corrections do
not.
