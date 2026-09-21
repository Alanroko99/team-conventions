# Comments & Documentation

Rule IDs: `DOC-*`. Severity: **error** unless marked otherwise.

## DOC-1 — Comments explain why, never what

The code states what it does. A comment restating it is noise that goes stale.

```
// increment i          <- delete this
// Retry once more than the gateway's own limit so a single gateway
// retry never surfaces as a user-visible failure.   <- keep this
```

Comment the non-obvious: the reason for an unusual approach, the constraint that forced it, the
link to the bug report, the invariant a reader must not break.

## DOC-2 — Public API surfaces carry doc comments

Every exported function, class, and module has a doc comment in the language's standard format
(JSDoc/TSDoc, docstrings, godoc, rustdoc) covering: purpose, parameters with units and constraints,
return value, and the errors or exceptions it can produce. Private internals do not need one.

## DOC-3 — `TODO` and `FIXME` require an owner and a ticket

```
// TODO(alice, PROJ-1234): remove once the v1 endpoint is retired
```

A bare `TODO` is permanent. Unattributed markers are flagged in review and must be resolved,
ticketed, or deleted.

## DOC-4 — Delete dead code instead of commenting it out

Git holds the history. Commented-out blocks confuse readers about what is live.

## DOC-5 — Every repository has a README that gets someone running

Required sections: what this is and who uses it, prerequisites with versions, install, run locally,
run the tests, and where configuration lives. Test the instructions on a clean machine before
claiming they work.

## DOC-6 — Non-obvious decisions get an ADR

Architecture Decision Records live in `docs/adr/NNNN-short-title.md` and record: context, the
decision, alternatives considered, and consequences. Write one when choosing a datastore, a
protocol, an auth model, or anything that will be expensive to reverse. ADRs are immutable —
supersede rather than edit.

## DOC-7 — Docs change in the same PR as the code

Documentation updated "later" is documentation that is wrong. README, API docs, runbooks, and
`CONVENTIONS.md` are part of the change, not a follow-up.

## DOC-8 — Runbooks for anything that pages

An on-call runbook states: what the alert means, how to confirm impact, first mitigation, escalation
path. Written so someone unfamiliar with the service can act at 3am.

## DOC-9 — Prefer making code self-explanatory to explaining it

A comment clarifying a confusing block is usually a sign to extract a well-named function instead.
Reach for a comment when the clarity cannot come from naming — an external constraint, a workaround,
a performance trade-off.
