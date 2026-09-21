---
name: conventions-init
description: This skill should be used when the user asks to "set up our conventions", "generate a CONVENTIONS.md", "write down our coding standards", "onboard this repo to our conventions", "document our team conventions", "create a style guide for this repo", or runs /conventions-init. Detects a repository's existing practices, generates CONVENTIONS.md from the team's rule packs, and writes the per-repo settings file.
argument-hint: "[path-to-repo] [--update] [--detect-only]"
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(git:*)
---

# Conventions Init

Generate and maintain a repository's `CONVENTIONS.md` and its `.claude/team-conventions.local.md`
settings file, grounded in what the repository actually does rather than in aspiration.

## Step 1 — Detect what the repository already does

Never write the document from the template alone. Inspect first, and record evidence for each
finding. Detect:

**Toolchain** — read the manifest (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`,
`Gemfile`) plus config files for formatter, linter, test runner, and package manager. Note the
scripts used to run tests and lint.

**Languages** — file extension counts across tracked files, ignoring vendored and generated paths.

**Casing in practice** — sample 20–30 source files and observe actual file, type, function, and
constant casing. Report what dominates, not what should dominate.

**Directory strategy** — feature-first or layer-first. Note where tests live relative to source,
and whether there is a config module or scattered environment reads (grep for `process.env`,
`os.environ`, `getenv`).

**Vocabulary** — the recurring domain nouns, and which synonyms appear for the same concept
(`fetch` vs `retrieve`, `customer` vs `client`). Count each so Step 3 can recommend a winner.

**Commit style** — `git log --oneline -100`. Measure what fraction match Conventional Commits, and
extract the ticket prefix from any `[A-Z]{2,}-[0-9]+` pattern in subjects, bodies, or branch names.

**Branch naming** — list local and remote branches, plus merged branch names from history.
Identify the default branch with `git symbolic-ref refs/remotes/origin/HEAD`.

**Logging** — which logger is imported, and whether calls are structured or interpolated strings.

**Existing docs** — find any current `CONVENTIONS.md`, `CONTRIBUTING.md`, `STYLE.md`, `CLAUDE.md`,
`.editorconfig`, or `docs/adr/`. Their contents take precedence over detection and over defaults.

With `--detect-only`, print the detection report and stop.

## Step 2 — Reconcile detection against the defaults

Read the rule packs from `${CLAUDE_PLUGIN_ROOT}/skills/team-conventions/references/<pack>.md` and
compare each detected practice to the corresponding rule. Classify every difference:

- **Match** — the repo already follows the default. Fill the template normally.
- **Consistent deviation** — the repo does something different, but does it everywhere. Keep the
  repo's practice, record it under **Deviations** with the reason, and add a severity override.
- **Inconsistent** — the repo does both. Do not pick silently. Surface it as an open question with
  the counts on each side and a recommendation.

Consistency inside a codebase outranks these defaults. Never propose a repo-wide rewrite to match
the template.

## Step 3 — Confirm the open questions

Present every inconsistency and every unknown as a short numbered list with a recommendation, and
ask the user to confirm or correct. Typical items: which casing wins, whether to adopt Conventional
Commits when history is mixed, the coverage policy, the ticket prefix, and which synonym wins for
each contested concept in the vocabulary.

Ask once, in one batch. Do not write the file before the answers arrive. If the user declines to
decide, write the item under **Open questions** in the generated document rather than guessing.

## Step 4 — Generate `CONVENTIONS.md`

Fill `templates/CONVENTIONS.md` and write it to the repository root. Replace every placeholder:

| Placeholder | Source |
|---|---|
| `{{PROJECT_NAME}}` | Manifest name, or directory name |
| `{{DATE}}` | Today, ISO-8601 |
| `{{LANGUAGES}}`, `{{FORMATTER}}`, `{{LINTER}}`, `{{TEST_RUNNER}}`, `{{PACKAGE_MANAGER}}` | Step 1 |
| `{{DEFAULT_BRANCH}}`, `{{TICKET_PREFIX}}` | Step 1 |
| `{{CASING_SUMMARY}}` | Observed casing, stated concretely per language |
| `{{VOCABULARY}}` | The words confirmed in Step 3, one bullet per concept |
| `{{E2E_DIR}}`, `{{CONFIG_MODULE}}`, `{{ADR_DIR}}`, `{{LOGGER}}`, `{{TEST_COMMAND}}` | Step 1, or the convention being adopted |
| `{{COVERAGE_POLICY}}` | The user's answer; write "no enforced threshold" when there is none |
| `{{DEVIATIONS}}` | Step 2 consistent deviations, each with its reason |
| `{{OPEN_QUESTIONS}}` | Unresolved items, or "None." |

**No placeholder may survive into the written file.** Where a value is unknown, do not substitute a
fallback string into the middle of a sentence — `TODO(none configured-1234)` and "config reading
happens in `none configured`" are worse than saying nothing. Instead:

- **Rewrite the containing bullet** to state the rule without the missing value. With no tracker,
  `GIT-5` already sanctions the ticket-free form, so the branch bullet becomes `type/short-slug`
  and the `Refs:` trailer bullet is dropped.
- **Delete the bullet** when the rule cannot be stated without the value — no ADR directory means
  no ADR bullet.
- A whole section that does not apply is trimmed entirely. A repo with no services needs no runbook
  line.

Only the standalone placeholders — `{{DEVIATIONS}}`, `{{OPEN_QUESTIONS}}`, `{{COVERAGE_POLICY}}` —
take a literal fallback such as "None." or "no enforced threshold".

**The packs are authoritative over the template.** The template's bullets are a condensed summary
of `${CLAUDE_PLUGIN_ROOT}/skills/team-conventions/references/`. Where a bullet and its pack
disagree — which happens as soon as a team edits a pack — rewrite the bullet from the pack text.
Never carry stale template wording into a generated document.

## Step 5 — Write the settings file

Create `.claude/team-conventions.local.md` following the schema in
`${CLAUDE_PLUGIN_ROOT}/skills/team-conventions/references/local-settings.md`. Populate it from the
decisions made above:

- `packs` — omit any pack the repository cannot honor.
- `exclude` — globs for frozen, vendored, or generated trees found in Step 1.
- `severity` — one override per consistent deviation, so `/convention-check` does not flag it.
- `ticket_prefix`, `default_branch` — from Step 1, confirmed in Step 3.
- `## Deviations` — the same entries written into `CONVENTIONS.md`, with their reasons.
- `## Vocabulary` — the confirmed word list. Write it here as well as in `CONVENTIONS.md`; without
  it `/convention-check` cannot check `NAM-8` at all.

Add `.claude/*.local.md` to `.gitignore` only if the settings should stay local. When the whole
team should share them, commit the file and say so.

## Step 6 — Wire it in

- Add or update a short section in the repository's `CLAUDE.md` pointing at `CONVENTIONS.md` and
  naming the `team-conventions` skill, so the conventions apply in every session whether or not the
  plugin is installed. Create `CLAUDE.md` if absent; append rather than overwrite if present.
- Add a line to `CONTRIBUTING.md` referencing `CONVENTIONS.md` when that file exists.

## Step 7 — Report

List the files created or changed, the deviations recorded, and the open questions left unresolved.
Suggest running `/convention-check --all` to see where the existing code stands against the newly
written document — and warn that on an established repo the first full audit will be long, which is
expected rather than a call to fix everything at once.

## Updating an existing document

With `--update`, or when `CONVENTIONS.md` already exists:

1. Re-run detection and diff it against the current document.
2. Report drift: rules the document states that the code no longer follows, and practices the code
   has adopted that the document does not mention.
3. Propose specific edits and apply only the ones confirmed.
4. Update the `Last reviewed` date.

Preserve all hand-written content. Never regenerate the file from the template over an existing
document — that discards decisions the team made deliberately.

## Resources

- **`templates/CONVENTIONS.md`** — the document skeleton with placeholders.
- **`${CLAUDE_PLUGIN_ROOT}/skills/team-conventions/references/`** — the eight rule packs, which are
  the authoritative text to reconcile against, plus `local-settings.md` for the settings schema.
