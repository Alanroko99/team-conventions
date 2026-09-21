# Naming Conventions

Rule IDs: `NAM-*`. Severity: **error** unless marked otherwise.

## NAM-1 — Follow the host language's casing, not a personal preference

Use whatever the language's dominant standard is, and use it consistently across the repo:

| Language | Files | Types | Functions / vars | Constants |
|---|---|---|---|---|
| JS/TS | `kebab-case.ts` | `PascalCase` | `camelCase` | `SCREAMING_SNAKE` |
| Python | `snake_case.py` | `PascalCase` | `snake_case` | `SCREAMING_SNAKE` |
| Go | `lowercase.go` | `PascalCase` | `camelCase` / `PascalCase` (exported) | `PascalCase` |
| Rust | `snake_case.rs` | `PascalCase` | `snake_case` | `SCREAMING_SNAKE` |
| Shell | `kebab-case.sh` | — | `snake_case` | `SCREAMING_SNAKE` |

Mixing two conventions inside one repo is the violation, not the choice itself. When a repo already
uses a non-standard convention consistently, match the repo and record the deviation in
`CONVENTIONS.md` rather than introducing a second style.

## NAM-2 — Names state what a thing is, not what type it is

Drop type suffixes that the type system already carries: `userList`, `configObj`, `dataMap`,
`IUserService`, `AbstractBaseFactoryImpl`. Prefer `users`, `config`, `usersById`, `UserService`.

Exception: a suffix that disambiguates two representations of the same concept is useful —
`userIds` vs `users`, `rawBody` vs `body`.

## NAM-3 — Booleans read as assertions

Prefix with `is`, `has`, `can`, `should`, or `will`: `isActive`, `hasPermission`, `canRetry`.
Never name a boolean after its negative (`notReady`, `disableCache`) — double negatives at call
sites are a recurring source of inverted logic.

## NAM-4 — Functions lead with a verb; the verb signals cost and effect

- `get*` / `read*` — cheap, no I/O, no mutation
- `fetch*` / `load*` — performs I/O, may fail or block
- `compute*` / `build*` — expensive, pure
- `set*` / `update*` / `delete*` — mutates
- `ensure*` — idempotent, creates only if absent
- `to*` / `as*` — conversion, no side effects

Never name an I/O-performing function `get*`. Callers treat `get` as free and will put it in a loop.

## NAM-5 — Abbreviations: only the universal ones

Allowed: `id`, `url`, `http`, `api`, `db`, `io`, `ui`, `max`, `min`, `num`, `ms`, `src`, `dest`.
Everything else spells out: `req`→`request`, `res`→`response` (except in framework handler
signatures where the framework's idiom dominates), `usr`→`user`, `cfg`→`config`, `tmp`→`temporary`
for anything that outlives one statement.

Single letters are acceptable only for loop indices (`i`, `j`), coordinates (`x`, `y`), and
generic type parameters (`T`, `K`, `V`).

## NAM-6 — Units and currency belong in the name

`timeout` is ambiguous. `timeoutMs`, `delaySeconds`, `sizeBytes`, `priceCents`, `distanceKm` are not.
Any numeric holding a physical quantity, a duration, or money carries its unit in the identifier.

## NAM-7 — No unexplained magic values

A literal other than `0`, `1`, `-1`, `""`, or an obvious empty collection gets a named constant at
module scope. This applies to timeouts, retry counts, size limits, port numbers, and status strings.

## NAM-8 — One concept, one word, repo-wide (severity: warning)

Pick `fetch` or `retrieve`, `user` or `account`, `delete` or `remove` — and use only that one across
the codebase. Synonym drift makes search unreliable and implies distinctions that do not exist.
Record the chosen vocabulary in `CONVENTIONS.md`.

## NAM-9 — Name length scales with scope

A variable alive for three lines can be short. A module-level export, a public API field, or a
database column is read by people with no surrounding context and must be self-describing.
