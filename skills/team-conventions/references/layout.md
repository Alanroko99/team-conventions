# File & Directory Layout

Rule IDs: `LAY-*`. Severity: **error** unless marked otherwise.

## LAY-1 — Organize by feature, not by technical layer

Prefer:

```
src/
  billing/
    invoice.ts
    invoice.test.ts
    subscription.ts
    index.ts          # the feature's public surface
  auth/
  shared/             # genuinely cross-feature only
```

over `controllers/`, `services/`, `models/`, `utils/` at the top level. Layer-first layouts force a
four-directory edit for every one-feature change and hide the actual coupling.

Exception: frameworks with a mandated layout (Rails, Django apps, Next.js `app/`) follow the
framework. Organize by feature *within* whatever the framework dictates.

## LAY-2 — Each directory exposes one public entry point

A feature directory has an `index.*` (or `__init__.py`, `mod.rs`) that re-exports its intended
surface. Code outside the feature imports only from that entry point, never from a file nested
inside it. This is what makes internal refactors non-breaking.

## LAY-3 — No `utils`, `helpers`, `common`, or `misc`

These names carry no information and become dumping grounds with no owner. Name the module after
what it actually does: `date-formatting.ts`, `retry.ts`, `currency.ts`. If a module genuinely holds
unrelated things, that is the signal to split it, not to name it vaguely.

`shared/` is permitted for code with two or more real consumers across features, and only when each
file inside it is itself specifically named.

## LAY-4 — Tests live beside the code they test

`invoice.ts` and `invoice.test.ts` in the same directory. Parallel `tests/` trees drift, get
forgotten in moves, and make it non-obvious whether a file has coverage at all.

Exception: integration, end-to-end, and fixture-heavy suites live in a top-level `tests/` or `e2e/`
directory, because they test the system rather than a file.

## LAY-5 — Dependencies point inward, never in a cycle

Domain logic depends on nothing. Application logic depends on domain. Adapters (HTTP, DB, CLI, UI)
depend on application. Nothing in the inner layers imports an adapter.

Circular imports between modules are an error, not a style preference — break them by extracting the
shared piece or inverting the dependency, never by moving the import inside a function.

## LAY-6 — File size is a smell, not a rule (severity: warning)

Past roughly 400 lines, or more than one exported concept with unrelated reasons to change, split
the file. Do not split a cohesive 500-line module just to satisfy a number.

## LAY-7 — Configuration is read at one boundary

All environment variable and config-file access happens in a single module that validates and
exports a typed config object. No `process.env` / `os.environ` reads scattered through business
logic — they defeat testing and hide the deployment contract.

## LAY-8 — Generated code is marked and never hand-edited

Generated files carry a header comment naming the generator and the command that regenerates them,
and are either committed under a clearly named directory (`generated/`, `*.gen.*`) or gitignored.
