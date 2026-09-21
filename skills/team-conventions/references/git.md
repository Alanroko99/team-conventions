# Git: Commits & Branches

Rule IDs: `GIT-*`. Severity: **error** unless marked otherwise.

## GIT-1 — Conventional Commits for every subject line

```
<type>(<scope>): <subject>
```

Types: `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `build`, `ci`, `chore`, `revert`.

- `<scope>` is the feature directory or package name; optional but preferred.
- `<subject>` is imperative mood, lowercase, no trailing period, ≤72 characters.
  Write "add retry to invoice sync", not "added", "adds", or "Adding".
- Breaking changes append `!` after the scope — `feat(api)!: drop v1 endpoints` — and explain the
  break in the body under a `BREAKING CHANGE:` line.

## GIT-2 — The body explains why, not what

The diff already shows what changed. The body states the motivation, the alternatives rejected, and
any non-obvious consequence. Wrap at 72 columns. Omit the body only when the subject is genuinely
self-explanatory (`chore(deps): bump lodash to 4.17.21`).

## GIT-3 — Reference the ticket, do not rely on it

Put `Refs: PROJ-1234` (or `Closes: #123`) in a trailer. The commit message must still stand alone —
a reader without tracker access should understand the change.

## GIT-4 — One logical change per commit

A commit compiles, passes tests, and does exactly one thing. Do not mix a refactor with a behavior
change; split them so the behavior diff is reviewable. Do not bundle unrelated fixes because they
happened in the same sitting.

Formatting-only changes go in their own commit, labeled `style:` or `refactor:`, so they never
obscure a behavioral diff.

## GIT-5 — Branch names

```
<type>/<ticket>-<short-slug>
```

Examples: `feat/PROJ-1234-invoice-retry`, `fix/PROJ-1290-null-customer`, `chore/bump-node-22`.

Type matches the Conventional Commit types. The slug is kebab-case, 2–5 words. Omit the ticket
segment only when no ticket exists.

## GIT-6 — Never rewrite shared history

`push --force` to a shared branch is forbidden. On a personal branch, use `--force-with-lease`, never
bare `--force`. `main` (and any release branch) is protected and receives changes only through a
reviewed pull request.

## GIT-7 — Never commit secrets, and rotate if it happens

Credentials, tokens, private keys, and `.env` files with real values stay out of the repository.
A secret that reaches a commit is compromised even after the commit is removed — rotate it, then
purge the history. Commit `.env.example` with placeholder values instead.

## GIT-8 — Rebase the feature branch, merge into the default branch

Keep a feature branch up to date with `rebase` so its history stays linear and reviewable. Integrate
into `main` with a merge commit or squash, per the repo's configured setting — consistently, not
per-author.

## GIT-9 — No commented-out code in a commit

Version control is the history. Delete it. The only exception is code commented out with an adjacent
explanation and a ticket reference for why it must remain visible.

## GIT-10 — Never commit directly to the default branch

Every change reaches `main` through a pull request, including one-line fixes and documentation.
