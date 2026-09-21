---
name: team-conventions
description: This skill should be used before writing or editing any code — adding or changing a function, class, variable, file, module, test, error handler, log statement, commit message, branch name, PR description, or dependency — and when the user asks "what are our conventions", "what's our commit format", "how do we name things here", "what's our branch naming", "how should I structure this file", or "what's our team style guide". Provides the team's language-agnostic conventions for naming, file layout, git commits, pull requests, error handling, logging, testing, comments, and dependencies.
---

# Team Conventions

Apply these conventions to all code written, edited, or reviewed. They are language-agnostic: they
govern structure, naming, error handling, and process rather than syntax that a formatter already
settles.

## How to use this skill

1. **Check for repository overrides first.** Read `.claude/team-conventions.local.md` in the
   repository root if it exists. It declares which rule packs are active, changes severities,
   excludes paths, and records deviations and vocabulary. Overrides there always win over the
   defaults below. The schema lives in `references/local-settings.md`.
2. **Load only the relevant pack.** Each rule pack is a separate file in `references/`. Read the
   one that matches the work at hand rather than all eight.
3. **Apply silently while writing.** Do not narrate rule IDs in ordinary work. Write conforming
   code. Cite a rule ID only when explaining a correction, answering a conventions question, or
   producing an audit report.
4. **Defer to the existing repository.** When a repo consistently uses a different convention,
   match the repo and note the deviation — consistency inside one codebase outranks these defaults.
   Do not mass-rewrite existing code to match these rules; apply them to code being touched.

## Rule packs

| Pack | Covers | File |
|---|---|---|
| `NAM-*` | Casing per language, verb prefixes, booleans, units, abbreviations, magic values | `references/naming.md` |
| `LAY-*` | Feature-first structure, entry points, dependency direction, config boundary | `references/layout.md` |
| `GIT-*` | Conventional Commits, branch names, history rules, secrets | `references/git.md` |
| `REV-*` | PR size and description, review etiquette, comment severity, merge rules | `references/reviews.md` |
| `ERR-*` | Never-swallow, cause chains, typed errors, structured logging, retries | `references/errors-logging.md` |
| `TST-*` | Test-per-behavior, determinism, mocking policy, naming, edge cases | `references/testing.md` |
| `DOC-*` | Why-not-what comments, public API docs, TODO ownership, READMEs, ADRs | `references/comments-docs.md` |
| `DEP-*` | Adding dependencies, lockfiles, pinning, upgrade cadence, vendor boundaries | `references/dependencies.md` |

Packs live at `${CLAUDE_PLUGIN_ROOT}/skills/team-conventions/references/<pack>.md`. From within
this skill, a relative path such as `references/naming.md` resolves.

## Which pack applies to which task

- Writing or renaming any identifier, file, or module → `naming.md`
- Creating files, moving code, deciding where something belongs → `layout.md`
- Writing a commit message or creating a branch → `git.md`
- Opening a PR, writing its description, or leaving review comments → `reviews.md`
- Any `try`/`catch`, error return, log statement, or retry → `errors-logging.md`
- Writing or changing tests, or deciding whether a change needs one → `testing.md`
- Writing comments, doc comments, READMEs, or ADRs → `comments-docs.md`
- Adding, upgrading, or removing a package → `dependencies.md`

## The rules that matter most

These recur across nearly every change. Apply them without needing to open a pack:

- **Never swallow an error** (`ERR-1`). Every caught error is recovered from, enriched and
  rethrown, converted at a boundary, or logged with the identifier of what failed. An empty catch
  block requires a comment explaining why.
- **Preserve the cause when wrapping** (`ERR-3`). Use `cause:`, `raise ... from`, or `%w`.
- **Comments explain why, not what** (`DOC-1`). Delete any comment that restates the code.
- **Behavior changes ship with a test** (`TST-1`). A bug fix includes a test that fails before it.
- **Conventional Commits, imperative subject** (`GIT-1`): `feat(billing): add invoice retry`.
- **Units in names** (`NAM-6`): `timeoutMs`, not `timeout`.
- **No `utils`, `helpers`, `common`, or `misc` modules** (`LAY-3`). Name the module for what it does.
- **Booleans read as assertions and are never negative** (`NAM-3`): `isEnabled`, not `notDisabled`.
- **No secrets in commits or logs** (`GIT-7`, `ERR-8`). A leaked secret is rotated, not just removed.

## Severity

Each rule carries a severity used by audits and review comments:

- **error** — must be fixed before merge. Rules default to this, except where a pack marks otherwise.
- **warning** — flagged, judgment applies. Marked explicitly in the pack.
- **off** — disabled for this repository via the settings file.

## Repository overrides

`.claude/team-conventions.local.md` customizes these defaults through five keys — `packs`,
`exclude`, `severity`, `ticket_prefix`, `default_branch` — plus two prose sections,
`## Deviations` (rule departures that are never flagged) and `## Vocabulary` (the repo's chosen
word per concept, which is what makes `NAM-8` checkable).

Read `references/local-settings.md` for the full schema, precedence rules, and a worked example.
Absent the file, all eight packs are active, every rule is an **error**, `ticket_prefix` is unset,
and `default_branch` is `main`.

## Related skills

- **`convention-check`** — audits uncommitted changes, a path, or a branch against these packs and
  reports violations. Invoke it when the user asks whether code follows the conventions.
- **`conventions-init`** — generates a repository's `CONVENTIONS.md` and the settings file, and
  writes a `CLAUDE.md` pointer that makes these conventions apply even without this plugin.
  Invoke it when onboarding a repository or when the user asks to write down the conventions.

## Answering questions about the conventions

When asked what a convention is, read the relevant pack and answer with the rule ID, the rule, and
the reasoning behind it. When the repository has a documented deviation, lead with the deviation
rather than the default.
