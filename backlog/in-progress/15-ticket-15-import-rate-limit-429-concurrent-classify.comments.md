# Comments: Ticket 15

### Devon — 2026-03-04 12:55

Implementation complete. Two changes:

1. `route.ts`: `MAX_CONCURRENCY = 2 → 1` — serializes classify+extract calls so no two files compete for the 50K tokens/min window simultaneously. Each file's full pipeline (including any retries) completes before the next starts.

2. `error-handling.ts`: Added ±25% jitter (`1 + (Math.random() * 0.5 - 0.25)`) to the exponential backoff delay — defensive improvement so concurrent retries (in any future scenario) don't fire in lockstep.

- Lint: 0 errors (88 warnings, pre-existing)
- Type-check: clean
- Unit tests: 967/967 pass
- PR #348 open as draft targeting staging

Tradeoff: multi-file imports are slightly slower (sequential). For 2 files: classify+extract file1 → classify+extract file2. Acceptable — reliability over throughput.

QA steps: upload 2+ PDFs, trigger import, verify no 429 in console, verify review screen shows vendors/installments/budget items populated, create wedding.
