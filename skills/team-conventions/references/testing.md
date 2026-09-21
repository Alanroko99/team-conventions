# Testing

Rule IDs: `TST-*`. Severity: **error** unless marked otherwise.

## TST-1 — Every behavior change ships with a test

A bug fix includes a test that fails before the fix and passes after. A new feature includes tests
for its success path and its documented failure paths. A PR that changes behavior with no test
change needs an explicit reason in the description.

Pure refactors are the exception: the existing tests passing unchanged is the evidence.

## TST-2 — Test observable behavior, not implementation

Assert on return values, emitted events, and persisted state. Do not assert that a private method
was called, or on internal call ordering that the contract does not promise. Tests coupled to
implementation block refactoring — the thing they were meant to enable.

## TST-3 — Name tests as statements about behavior

`test_rejects_invoice_with_past_due_date`, not `test_invoice_2`. The name is what a reader sees in
a CI failure; it must identify the broken contract without opening the file.

Structure the body in three visible phases — arrange, act, assert — separated by blank lines.

## TST-4 — One behavior per test, and assert on it

Multiple assertions are fine when they describe one behavior. Testing three unrelated behaviors in
one function means the first failure hides the rest.

## TST-5 — Tests are deterministic and order-independent

No dependence on wall-clock time, timezone, locale, random seeds, network access, or the execution
order of other tests. Inject the clock and the random source. A test that passes in isolation but
fails in a suite is a broken test.

## TST-6 — No sleeps

Never `sleep(2)` to wait for async work. Await the promise, poll a condition with a timeout, or use
the framework's fake timers. Sleeps make suites slow and flaky simultaneously.

## TST-7 — Quarantine or fix flaky tests immediately

A test that fails intermittently is worse than no test: it teaches the team to re-run CI without
reading failures. Fix it, or mark it skipped with a linked ticket, the same day it is identified.

## TST-8 — Mock only what the team does not own

Mock external HTTP services, third-party SDKs, and the clock. Do not mock first-party modules to
test other first-party modules — use the real one. Heavy internal mocking tests the mocks.

Prefer an in-memory or containerized real database over a mocked repository layer.

## TST-9 — Each test builds its own data and cleans up

No shared mutable fixtures across tests. Use factories or builders with sensible defaults so a test
specifies only the fields it cares about. Tests leave no residue in shared state.

## TST-10 — Coverage guides, it does not certify (severity: warning)

Use coverage to find untested branches, not as a target to hit. Meaningful tests on the error paths
of critical code beat uniform line coverage. If the repo enforces a threshold, record the number and
its rationale in `CONVENTIONS.md`.

## TST-11 — Shape the suite as a pyramid

Many fast unit tests, fewer integration tests at real boundaries, a thin layer of end-to-end tests
covering critical user journeys only. The full unit suite runs in seconds; if it does not, that is
a defect to fix.

## TST-12 — Test the edges

For every unit under test, cover: empty input, a single element, the boundary value, one past the
boundary, null/undefined where permitted, and the documented error conditions. The happy path alone
is not coverage.
