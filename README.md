# team-conventions

A Claude Code plugin that holds your team's engineering conventions in one place and puts them to
work three ways: Claude follows them while writing code, `convention-check` audits changes against
them, and `conventions-init` writes them down for the humans.

The rules are **language-agnostic**. They govern naming semantics, structure, error handling,
testing, git process, and review etiquette — not syntax that a formatter already settles.

## What's in it

| Component | Type | What it does |
|---|---|---|
| `team-conventions` | skill (auto) | Loads whenever code is written or reviewed. Applies the rules silently; answers "what's our commit format?" |
| `convention-check` | skill (invoked) | Audits uncommitted changes, a path, or a branch. Reports violations by severity. `--fix` applies safe corrections. |
| `conventions-init` | skill (invoked) | Detects what a repo actually does, reconciles it against the rules, writes `CONVENTIONS.md` and the settings file. |
| `convention-auditor` | agent | Read-only batch auditor. Only fires when `convention-check` delegates a large changeset — never on its own. |

## Rule packs

Eight packs, 78 rules, each with an ID so findings and review comments can cite them.

| Pack | ID | Rules | Covers |
|---|---|---|---|
| Naming | `NAM-*` | 9 | Casing per language, verb prefixes signalling cost, boolean assertions, units in names, abbreviations, magic values |
| Layout | `LAY-*` | 8 | Feature-first structure, single entry points, dependency direction, the config boundary |
| Git | `GIT-*` | 10 | Conventional Commits, branch naming, protected history, secrets |
| Reviews | `REV-*` | 10 | PR size, description contents, comment severity prefixes, merge rules |
| Errors & logging | `ERR-*` | 12 | Never-swallow, cause chains, typed domain errors, structured logging, bounded retries |
| Testing | `TST-*` | 12 | Test-per-behavior, determinism, mocking policy, edge coverage, pyramid shape |
| Comments & docs | `DOC-*` | 9 | Why-not-what comments, public API docs, `TODO` ownership, READMEs, ADRs |
| Dependencies | `DEP-*` | 8 | Justifying additions, lockfiles, pinning, upgrade cadence, vendor boundaries |

Full text lives in `skills/team-conventions/references/`. Edit those files to make the conventions
yours — that's the intended way to customize the plugin.

## Install

Local, for one session:

```bash
claude --plugin-dir /path/to/team-conventions
```

Persistent — this repo doubles as a single-plugin marketplace:

```
/plugin marketplace add /path/to/team-conventions
/plugin install team-conventions@team-conventions-marketplace
```

## Usage

Plugin skills are namespaced, so the full invocation is
`/team-conventions:convention-check`. Claude Code accepts the short form when the name is
unambiguous:

```
/convention-check                    # uncommitted changes (default)
/convention-check --branch           # everything on this branch vs the default branch
/convention-check src/billing/       # a path
/convention-check --all              # the whole repo
/convention-check --fix              # apply the mechanical fixes
/convention-check --pack errors-logging

/conventions-init                    # detect, confirm, generate CONVENTIONS.md
/conventions-init --detect-only      # report what the repo does today, write nothing
/conventions-init --update           # diff the existing doc against reality
```

The `team-conventions` skill needs no invocation — it loads on its own when Claude writes or
reviews code, and when you ask a conventions question. Running `conventions-init` makes this more
reliable: its Step 6 writes a `CLAUDE.md` pointer, which applies the conventions in every session
whether or not the plugin is loaded.

## Per-repo configuration

Drop `.claude/team-conventions.local.md` in a repository root to override the defaults:

```markdown
---
packs: [naming, layout, git, reviews, errors-logging, testing, comments-docs, dependencies]
exclude: [src/legacy/**, vendor/**]
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
```

Anything under `## Deviations` is permitted practice and never flagged; `exclude` drops paths from
auditing entirely. `## Vocabulary` is what makes `NAM-8` (one word per concept) mechanically
checkable — without it the rule is skipped rather than guessed at.

The authoritative schema is `skills/team-conventions/references/local-settings.md`. A fully
annotated example is in `examples/team-conventions.local.md`.

Commit the file to share settings with the team; leave it gitignored to keep them local.

## Design notes

- **Report before fix.** `convention-check` never edits without `--fix`, and even then only applies
  mechanically safe changes. Anything requiring judgment is reported with a suggestion.
- **The repo outranks the plugin.** When a codebase consistently does something different, the
  skills match the codebase and record the deviation. No mass rewrites.
- **Conventions, not code review.** These skills check the rules in the packs. Bugs, security, and
  performance belong to a general code review.
- **No surprise agents.** `convention-auditor` fires only when `convention-check` delegates a
  changeset over ~15 files or ~800 lines, and it is read-only.

## Customizing

1. Edit the pack files in `skills/team-conventions/references/` — that's the source of truth.
2. Keep rule IDs stable; findings, review comments, and settings files reference them.
3. `skills/conventions-init/templates/CONVENTIONS.md` is a condensed summary of the packs. The
   skill rewrites any bullet that disagrees with its pack, but updating the template keeps
   generated documents closer to the source.
4. Bump `version` in `.claude-plugin/plugin.json`.

## License

MIT — see `LICENSE`.
