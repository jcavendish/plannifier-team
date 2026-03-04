# Ticket 14: No user feedback after wedding creation

**Status:** In Progress — Awaiting Merge
**Assignee:** Review: Joao
**Plan Status:** Approved
**Latest Update:** QA passed (Qamar, 2026-03-04) — all acceptance criteria met, 4/4 E2E pass. Ready for Joao review.
**PR:** https://github.com/jcavendish/plannifier/pull/344
**Severity:** High
**Size:** XS
**Type:** Bug Fix
**Depends On:** —

## Summary

After clicking "Create Wedding" in the import review wizard, the button stops loading but nothing else happens — no success toast, no navigation to the new wedding. The wedding may actually be created in the database but the user has no way to know or reach it.

## Problem

In `ImportReviewPageClient.tsx`, the success feedback is gated behind:

```ts
if (result.data?.weddingId) {
  toast.success(t('successTitle'));
  router.push(`/${locale}/planner/weddings/${result.data.weddingId}`);
}
```

If `result.data.weddingId` is falsy (empty string `''` — the default in `ImportResult`), no toast fires and no navigation happens. `finally` sets `setCreating(false)`, so the button returns to normal. Silent failure from the user's perspective.

## Root Cause

`ImportResult.weddingId` defaults to `''` in `wedding-creator.ts`. If the RPC returns but `rpcResult.wedding_event_id` is null/undefined for any reason, `result.weddingId` stays `''`. The client checks `result.data?.weddingId` which is falsy for empty string — so no success path runs.

Additionally, even if the API returns a 200 with a valid `weddingId`, there's no fallback handling for the case where the navigation or toast fails.

## Proposed Fix

1. **Investigate why `rpcResult.wedding_event_id` could be null** — add logging in `wedding-creator.ts` to capture the full `rpcResult` shape before extracting `wedding_event_id`.
2. **Fix the client-side guard** — if `response.ok` and no `result.error`, always show feedback. Navigate to `/planner/weddings/${weddingId}` if we have an ID, or fall back to `/planner/weddings` (weddings list) with a toast.
3. **Add explicit error for empty weddingId** — if `response.ok` but `weddingId` is empty, treat it as an unexpected error (toast error + log), not silent success.

## Acceptance Criteria

- [x] Clicking "Create Wedding" always results in visible feedback (success or error) — never silent
- [x] On success, user is navigated to the newly created wedding page
- [x] If navigation target is unavailable, user is sent to weddings list with a success toast
- [x] Error case shows an error toast with a human-readable message

## Files

- `frontend/src/app/[locale]/planner/imports/[id]/review/ImportReviewPageClient.tsx`
- `frontend/src/lib/services/import/wedding-creator.ts`
- `frontend/src/app/api/import/create-wedding/route.ts`

## Latest Update

QA passed (Qamar, 2026-03-04). See comments file for full test results.
