# Repository Settings File

The single source of truth for the `.claude/team-conventions.local.md` schema. The
`team-conventions`, `convention-check`, and `conventions-init` skills all read this file rather
than restating the schema.

## Location

`.claude/team-conventions.local.md`, in the repository root. Optional.

Commit it to share settings across the team; leave it gitignored (`.claude/*.local.md`) to keep
them local to one machine.

## Defaults when absent

All eight packs active, every rule **error**, `ticket_prefix` unset (ticket references become
optional), `default_branch` is `main`, no exclusions, no deviations, no vocabulary.

## Format

Frontmatter carries the machine-readable keys and must start at line 1. Prose sections below it
carry the human-readable decisions.

```markdown
---
packs: [naming, layout, git, reviews, errors-logging, testing, comments-docs, dependencies]
exclude:
  - src/legacy/**
  - vendor/**
severity:
  TST-10: off
  LAY-6: warning
ticket_prefix: PROJ
default_branch: main
---

## Deviations

- React components use `PascalCase.tsx`, overriding NAM-1 — framework convention, 140 files.

## Vocabulary

- `customer` — never `client`, `user`, or `account`
- `fetch` — for any I/O read; never `retrieve`, `load`, or `pull`
```

## Keys

| Key | Type | Meaning |
|---|---|---|
| `packs` | list | Active rule packs, named by pack filename without extension. Omit a name to disable that pack entirely. Defaults to all eight. |
| `exclude` | list | Glob patterns excluded from all audits — frozen code, vendored trees, generated output. Distinct from a deviation: these paths are not audited at all. |
| `severity` | map | Per-rule overrides keyed by rule ID. One of `error`, `warning`, `off`. |
| `ticket_prefix` | string | Tracker prefix for branch names (`GIT-5`) and commit trailers (`GIT-3`). Omit when there is no tracker; ticket references then become optional rather than required. |
| `default_branch` | string | The protected branch. Used by `GIT-10` and as the diff base for `/convention-check --branch`. |

Both list keys accept inline (`[a, b]`) or block (`- a`) YAML. Prefer block form once a list runs
past one line.

## Sections

### `## Deviations`

Intentional departures from a rule, each with its reason. Treat every entry as binding permitted
practice: never report it as a violation, and never propose reverting it.

A deviation overrides a *rule* for the repository. To exempt a *path* from auditing, use `exclude`
instead — the two are different mechanisms and conflating them makes audits unpredictable.

### `## Vocabulary`

The repository's chosen word for each concept, enforcing `NAM-8`. One bullet per concept: the
chosen word, then the synonyms that must not appear.

`/convention-check` checks new and renamed identifiers against this list and reports a `NAM-8`
violation when a banned synonym is used. Absent this section, `NAM-8` cannot be checked
mechanically and is skipped rather than guessed at.

`/conventions-init` populates this section from the repository's observed domain nouns and writes
the same list into `CONVENTIONS.md`.

## Precedence

1. `exclude` — excluded paths are never audited, regardless of anything else.
2. `## Deviations` — a documented deviation is never a violation.
3. `severity` — overrides the pack's declared severity.
4. `packs` — a disabled pack contributes no rules.
5. Pack defaults — everything not overridden above.
