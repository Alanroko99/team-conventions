# Dependencies

Rule IDs: `DEP-*`. Severity: **error** unless marked otherwise.

## DEP-1 — Adding a dependency is a decision, not a reflex

Before adding one, confirm: the standard library cannot do it in reasonable code; the package is
maintained (released within ~12 months, open issues being answered); the license is compatible; the
transitive tree and install size are acceptable; and the cost of vendoring the needed 30 lines is
genuinely higher than the cost of owning the dependency.

New runtime dependencies are called out explicitly in the PR description with this reasoning.

## DEP-2 — Lockfiles are committed and respected

Commit `package-lock.json` / `yarn.lock` / `poetry.lock` / `go.sum` / `Cargo.lock`. CI installs from
the lockfile with the frozen flag (`npm ci`, `poetry install --sync`, `go mod verify`) and fails if
the lockfile is stale. Never regenerate a lockfile as a drive-by change in an unrelated PR.

## DEP-3 — Pin exact versions for applications, use ranges for libraries

Applications pin exact versions so every environment is byte-identical. Published libraries declare
compatible ranges so consumers are not forced into conflicts.

## DEP-4 — Separate runtime from development dependencies

Test frameworks, linters, and build tooling belong in the dev section and must not reach the
production image. Check this when adding anything.

## DEP-5 — Upgrade on a schedule, not on an incident

Run dependency updates on a regular cadence in dedicated PRs, grouped by risk: patch and minor
together, each major on its own with the changelog reviewed. Security advisories are the exception
and are handled immediately.

## DEP-6 — Wrap volatile third-party APIs at a boundary

When a third-party client is called from many places, put a thin adapter module in front of it.
This makes replacement a one-file change and keeps the vendor's types out of the domain layer.
Do not wrap stable, ubiquitous libraries — the indirection costs more than it saves.

## DEP-7 — No unvetted install-time scripts or unpinned remote sources

Do not add packages that run arbitrary post-install scripts without review, and never
`curl | sh` from an unpinned URL in a build. Pin container base images by digest, not by a
mutable tag like `latest`.

## DEP-8 — One tool per job

A repo has one HTTP client, one date library, one test runner, one assertion style. Two libraries
doing the same job is a defect to resolve, not a preference to accommodate.
