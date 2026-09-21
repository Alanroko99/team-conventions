# Error Handling & Logging

Rule IDs: `ERR-*`. Severity: **error** unless marked otherwise.

## ERR-1 — Never swallow an error silently

An empty `catch {}` / `except: pass` / ignored error return is an error, not a style choice.
Every caught error is handled in one of exactly four ways:

1. **Recover** — take a documented alternative path.
2. **Enrich and rethrow** — add context, preserve the cause.
3. **Convert** — translate to a domain error at a boundary, preserving the cause.
4. **Log and continue** — only in a loop or background job where one failure must not stop the rest,
   and only with a log line that identifies the item that failed.

Deliberately ignoring an error requires a comment saying why.

## ERR-2 — Catch narrowly

Catch the specific error type expected at that point. A bare `except Exception` / `catch (e)` around
a multi-statement block hides bugs from unrelated statements. Where a broad catch is unavoidable
(top-level request handler, worker loop), it must log the full error and stack trace.

## ERR-3 — Preserve the cause chain

Never discard the original error when wrapping. Use the language's built-in mechanism:
`throw new Error("msg", { cause: err })`, `raise X("msg") from err`, `fmt.Errorf("msg: %w", err)`.
A rethrow that drops the cause destroys the only useful debugging information.

## ERR-4 — Fail fast at boundaries, be tolerant inside

Validate all external input — HTTP bodies, CLI arguments, environment config, file contents,
third-party responses — at the entry point, and reject with a precise message. Once past the
boundary, code may assume the data is well-formed. Do not re-validate the same shape at six layers.

## ERR-5 — Error messages name the failure, the subject, and the next step

Bad: `"Error"`, `"Something went wrong"`, `"Invalid input"`.
Good: `"Cannot parse invoice 4821: due_date is empty; expected ISO-8601 date"`.

User-facing messages say what to do next. Operator-facing messages carry the identifiers needed to
find the record. Neither contains a stack trace shown to an end user.

## ERR-6 — Domain errors are typed, not stringly matched

Define error types/classes per failure mode (`InvoiceNotFound`, `PaymentDeclined`). Never branch on
`err.message.includes("not found")` — message text is not an API and will change.

## ERR-7 — Use structured logging with consistent levels

Log as key-value fields, not interpolated prose, so logs are queryable.

| Level | Use for | Paged on |
|---|---|---|
| `error` | Failed operation needing human attention | yes |
| `warn` | Degraded but handled; retry succeeded, fallback used | trend only |
| `info` | Significant state change: started, completed, config loaded | no |
| `debug` | Developer detail, off in production | no |

`error` is for things someone must act on. Logging `error` for an expected validation failure
trains people to ignore the level.

## ERR-8 — Never log secrets or personal data

Tokens, passwords, API keys, full card numbers, and personal identifiers stay out of logs. Redact at
the logging helper, not at each call site, so it cannot be forgotten. Log a user ID, never an email
body.

## ERR-9 — Every log line carries correlation context

Request ID, trace ID, job ID, or tenant ID on every line emitted while handling a unit of work.
Propagate it across service calls. Logs without correlation are unusable at volume.

## ERR-10 — Log at the point of handling, not the point of raising

Logging an error and then rethrowing it produces duplicate entries for a single failure. Enrich on
the way up; log once, where the decision is made.

## ERR-11 — Retries are bounded, backed off, and only for transient failures

Retry network timeouts, 5xx, and lock contention — never validation errors or 4xx. Use exponential
backoff with jitter and a hard attempt cap. Non-idempotent operations require an idempotency key
before they may be retried.

## ERR-12 — Clean up deterministically

Release files, connections, locks, and timers with the language's scope-bound mechanism
(`try/finally`, `with`, `defer`, RAII), never by relying on a line at the end of the happy path.
