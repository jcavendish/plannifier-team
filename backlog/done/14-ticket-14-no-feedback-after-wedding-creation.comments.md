# Comments: Ticket 14

### Devon — 2026-03-04 10:48

Started Ticket 14. Read all relevant code: `ImportReviewPageClient.tsx`, `wedding-creator.ts`, `create-wedding/route.ts`, `extraction-schemas.ts`.

Root cause confirmed: success path in `ImportReviewPageClient.tsx` is gated on `result.data?.weddingId` being truthy. If the RPC returns but `weddingId` is empty/null, no toast and no navigation fires — `finally` just stops the spinner.

Plan is ready. Fix is 15 lines across 2 files:
1. Client: always fire toast on 200, navigate to weddings list if `weddingId` missing
2. Server: add defensive log to capture the raw RPC result when `wedding_event_id` is empty

Awaiting Joao's approval before implementing.

### Devon — 2026-03-04 11:00

Implementation complete. PR #344 open as draft targeting staging.

Changes:
- `ImportReviewPageClient.tsx`: toast.success now always fires on 200. Navigate to wedding if weddingId present, fall back to /planner/weddings if empty.
- `wedding-creator.ts`: defensive console.error log when wedding_event_id is missing from RPC result.

Lint: 0 errors. Type-check: clean. Build: compiled and pages generated (trace step times out on sandbox — pre-existing, same as prior tickets).

Handing to Qamar for QA.

### Qamar — 2026-03-04 12:11

## QA Testing Results

**Status:** ✅ PASS

**Branch:** `fix/ticket-14-wedding-creation-feedback`
**PR:** https://github.com/jcavendish/plannifier/pull/344

---

### Code Review

Diff is clean: 2 files, 13 insertions, 4 deletions — exactly what Devon described.

**`ImportReviewPageClient.tsx`** — logic flow is correct:
- 200 + `result.error` truthy → error toast (via catch) ✓
- 200 + no error + weddingId present → success toast + navigate to wedding ✓
- 200 + no error + weddingId empty → success toast + console.error + navigate to list ✓
- Non-200 → error toast (from catch) ✓

**`wedding-creator.ts`** — defensive log is correct and non-invasive. ✓

**i18n:** `ImportCreation.successTitle` verified present in both `en.json` ("Wedding created successfully!") and `pt-BR.json` ("Casamento criado com sucesso!"). All error keys present. ✓

**Lint:** 0 errors on changed files. ✓

**Build:** OOM crash during trace step — pre-existing issue (Ticket 8), not introduced by this change. Same as all prior tickets. ✓

---

### Flows Tested (all on real `/en/planner/imports/[id]/review` route)

| Flow | Description | Result |
|------|-------------|--------|
| 1 | Happy path: success toast fires + navigates to wedding page | ✅ PASS |
| 2 | Fallback path: 200 + empty weddingId → toast fires + navigates to list | ✅ PASS |
| 3 | Error path: API 500 → error toast, stays on review page, button re-enabled | ✅ PASS |
| 4 | Loading state: button disabled + spinner during creation | ✅ PASS |

---

### Acceptance Criteria

- [x] Clicking "Create Wedding" always results in visible feedback — never silent
- [x] On success, user navigated to the new wedding page
- [x] If weddingId missing, user sent to weddings list with success toast
- [x] Error case shows error toast with human-readable message

---

### E2E Tests Written

`frontend/tests/document-import-creation-feedback.critical.spec.ts` — 4 tests, 4/4 pass (35.7s)

These tests use the real review page route (not the mock test helper), which is required to validate toast/navigation behavior. Committed to branch: `46fd0b3`.

---

### Notes

- No Critical/High issues found.
- The fix is minimal and surgical — no unrelated changes.
- The OOM build issue is pre-existing (Ticket 8) and does not block this ticket.

Ready for Joao review.
