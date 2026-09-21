---
# Active rule packs. Omit a name to disable that pack entirely.
packs:
  - naming
  - layout
  - git
  - reviews
  - errors-logging
  - testing
  - comments-docs
  - dependencies

# Globs excluded from all audits. Distinct from a deviation: these paths are never read.
exclude:
  - src/legacy/**
  - vendor/**
  - "**/*.gen.ts"

# Per-rule severity overrides, keyed by rule ID. One of: error, warning, off.
severity:
  TST-10: off       # no coverage threshold enforced here
  LAY-6: warning    # large files exist and are not worth splitting yet
  DOC-2: warning    # internal service, public API docs are aspirational

# Tracker prefix used in branch names and commit trailers. Omit when there is no tracker.
ticket_prefix: PROJ

# Protected branch. Used by GIT-10 and as the diff base for /convention-check --branch.
default_branch: main
---

## About this file

An annotated example of the repository settings the `team-conventions` plugin reads. Copy it to
`.claude/team-conventions.local.md` in a repository root and edit it — the frontmatter above must
stay at line 1 for the skills to parse it.

Everything here overrides the plugin defaults. Delete the keys and sections that do not apply; an
absent file means all eight packs active with every rule an error.

The authoritative schema, including precedence between these keys, is
`skills/team-conventions/references/local-settings.md`.

Commit this file to share the settings with the team, or leave it gitignored via
`.claude/*.local.md` to keep them local to one machine.

## Deviations

Intentional departures from a *rule*, each with its reason. `/convention-check` treats everything
listed here as permitted and will not flag it. To exempt a *path* from auditing entirely, use the
`exclude` key above instead — the two are different mechanisms.

- **React components use `PascalCase.tsx`**, overriding `NAM-1`. The framework convention won;
  changing it now would churn every import across 140 files.
- **Integration tests live in `tests/integration/`**, not beside the source, overriding `LAY-4`.
  They exercise the system rather than a file, which the rule explicitly permits.
- **Commit scopes are omitted**, relaxing `GIT-1`. The repo is a single service; a scope on every
  commit adds noise without adding information.

## Vocabulary

One word per concept, repo-wide. This section is what makes `NAM-8` mechanically checkable —
without it, `/convention-check` skips the rule rather than guessing at synonyms.

- `customer` — never `client`, `user`, or `account`
- `fetch` — for any I/O read; never `retrieve`, `load`, or `pull`
- `delete` — never `remove`, `destroy`, or `purge`
