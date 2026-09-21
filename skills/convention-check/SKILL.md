---
name: convention-check
description: This skill should be used when the user asks to "check conventions", "convention check", "does this follow our conventions", "audit this against our standards", "review my changes against the style guide", "check this file for convention violations", or runs /convention-check. Audits code against the team-conventions rule packs and reports violations by severity; applies safe fixes only when passed --fix.
argument-hint: "[path | glob | branch | --branch | --all] [--fix] [--pack <name>]"
allowed-tools: Read, Grep, Glob, Bash, Edit, Task
---

# Convention Check

Audit code against the team's conventions and report violations. Report only by default; change
files only when `--fix` is present.

## Step 1 — Resolve the target

Parse arguments to determine what to audit. Precedence, first match wins:

| Argument | Target |
|---|---|
| `--all` | Every tracked file in the repository |
| `--branch` | `git diff <default_branch>...HEAD` — everything on this branch |
| A git ref (`main`, a SHA, `origin/main`) | `git diff <ref>...HEAD` |
| A path or glob | Those files in full |
| *(none)* | **Uncommitted changes** — the default |

For the default, run `git diff HEAD --name-only` plus `git ls-files --others --exclude-standard`
to include untracked files. When both are empty, report that there is nothing to audit and stop —
do not silently widen to the whole repository.

Outside a git repository with no path argument, ask which path to audit rather than assuming.

## Step 2 — Load the rules

Read `.claude/team-conventions.local.md` if present for active packs, excluded paths, severity
overrides, `ticket_prefix`, `default_branch`, documented deviations, and the repository vocabulary.
Its schema is `${CLAUDE_PLUGIN_ROOT}/skills/team-conventions/references/local-settings.md`.

When neither that file nor a `CONVENTIONS.md` exists, audit against the defaults and mention once
that `/conventions-init` will record the repository's own decisions. Do not block on it.

Rule packs live at `${CLAUDE_PLUGIN_ROOT}/skills/team-conventions/references/<pack>.md`. Select
packs based on what the target actually contains, and read only those:

| What the target contains | Packs |
|---|---|
| Source files changed | `naming.md`, `errors-logging.md`, `comments-docs.md` |
| Files added, moved, or deleted | `layout.md` |
| Test files touched, or source changed without a test | `testing.md` |
| Commits being audited, or `--branch` | `git.md` |
| `--branch`, or a PR being prepared | `reviews.md` |
| Manifest or lockfile changed | `dependencies.md` |
| `--pack <name>` given | That pack only |

Skip packs disabled in the settings, and skip files matching its `exclude` globs entirely. Never
flag a violation that the settings file documents as an intentional deviation.

Check `NAM-8` (one word per concept) only when the settings file has a `## Vocabulary` section —
without it there is nothing to check against, so skip the rule rather than guessing at synonyms.

## Step 3 — Delegate when the changeset is large

When the target exceeds roughly 15 files or 800 changed lines, dispatch the `convention-auditor`
agent instead of reading everything directly. Split the file list into batches of about 10 related
files and run one agent per batch in parallel.

Give each agent: the absolute file paths in its batch, the absolute paths of the rule packs to
apply — expand `${CLAUDE_PLUGIN_ROOT}` so the agent receives real paths, not the variable — the
severity overrides, the documented deviations, and the required finding format from Step 4.
Collect the returned findings and continue at Step 5.

Below that threshold, read the files directly — a subagent costs more than it saves.

## Step 4 — Find violations

Read each file and check it against the loaded packs. Record each violation as:

```
<file>:<line>  <RULE-ID>  <severity>  <one-line description>
```

Rules for what counts:

- Report a violation only when the specific rule text is breached. Do not invent rules, and do not
  report general code-quality opinions the packs do not cover — that belongs to a general code
  review, not to this skill.
- Report a rule once per distinct occurrence, not once per line it spans.
- When a file has more than ten violations of one rule, collapse them into a single finding with a
  count and the first three locations.
- Pre-existing violations in untouched lines are out of scope unless `--all` was passed. Audit what
  changed.
- Skip generated files, vendored directories, lockfiles, and anything matched by `.gitignore` or by
  the settings file's `exclude` list.

## Step 5 — Report

Print a report in this shape. Order by severity, then by file.

```
Convention check — 7 files, 3 packs (naming, errors-logging, testing)

ERRORS (4)
  src/billing/invoice.ts:42   ERR-1   Empty catch block swallows the parse failure
  src/billing/invoice.ts:88   ERR-3   Rethrow drops the original error; pass it as `cause`
  src/billing/sync.ts:15      NAM-6   `timeout` holds milliseconds — rename to `timeoutMs`
  src/billing/sync.ts:—       TST-1   Behavior changed with no accompanying test

WARNINGS (1)
  src/billing/invoice.ts:1    LAY-6   File is 612 lines; consider splitting by concern

CLEAN
  src/billing/types.ts, src/billing/index.ts

4 errors, 1 warning. Run with --fix to apply the 2 mechanical fixes (ERR-3, NAM-6).
```

The count in the closing line covers only findings on the Step 6 auto-fixable list — in this
example `ERR-3` and `NAM-6`, since `ERR-1`, `TST-1`, and `LAY-6` all require judgment. Naming the
fixable rule IDs keeps the count honest.

Close with a one-line summary. When nothing is found, say so in one line without padding the output.

## Step 6 — Fix, only with `--fix`

Without `--fix`, stop after the report. Never edit files.

With `--fix`, apply only mechanically safe corrections:

- Renames for `NAM-2`, `NAM-3`, `NAM-6` where every reference is inside the audited files
- Renames to the repository vocabulary for `NAM-8`, under the same whole-reference condition
- Extracting magic values to named constants (`NAM-7`)
- Deleting commented-out code (`DOC-4`, `GIT-9`)
- Deleting comments that restate the code (`DOC-1`)
- Adding the cause to a rethrow (`ERR-3`)
- Adding an owner and ticket to a bare `TODO` (`DOC-3`) — only when `ticket_prefix` is configured
  and both the owner and the ticket are known; otherwise leave it and report

Never auto-fix: anything requiring a judgment call about intent, restructuring files (`LAY-*`),
writing tests (`TST-1`), broadening or narrowing a catch (`ERR-2`), or renames whose references
extend outside the audited set. Report those with a suggested change instead.

After fixing, re-run the affected checks, then list what was changed and what was left for manual
handling. Run the repository's test suite if one is configured and the fixes touched source files.

## Notes

- This skill audits conventions, not correctness. Bugs, security issues, and performance problems
  belong to a general code review; say so rather than expanding scope.
- Keep the report terse. A reviewer scans it; prose between findings makes it unusable.
