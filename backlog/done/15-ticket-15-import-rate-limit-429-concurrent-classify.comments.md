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

### Devon — 2026-03-04 13:17

QA test steps updated (Joao feedback). Review screen showing entities is NOT sufficient — the real test is post-creation.

**Corrected QA steps for PR #348:**
1. Upload 2+ PDFs → trigger import → verify no 429 in console/logs
2. Verify all files classified + extracted (review screen populated)
3. Create wedding
4. **Navigate to wedding → Budget tab:**
   - Budget lines present with correct descriptions
   - Each budget line has installments (payment schedule rows)
   - Payments present
5. **Navigate to wedding → Vendors tab:**
   - Vendors present
   - Vendor contracts linked (if source docs had contract data)

Pass = all entities exist in DB after creation, not just in review screen preview. Updated PR #348 description accordingly.
