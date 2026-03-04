# Plan: Ticket 14 — No user feedback after wedding creation

## Root Cause (confirmed from code)

In `ImportReviewPageClient.tsx`, the success path is:

```ts
if (result.data?.weddingId) {
  toast.success(t('successTitle'));
  router.push(`/${locale}/planner/weddings/${result.data.weddingId}`);
}
```

If `result.data.weddingId` is falsy (empty string `''`, `null`, or `undefined`), **no feedback fires at all** — no toast, no navigation, no error. `finally` resets the button. Silent.

The API returns `NextResponse.json({ data: result })` where `result.weddingId = rpcResult.wedding_event_id`. The RPC (`create_wedding_from_import`) does return `wedding_event_id` in its JSONB:

```sql
RETURN jsonb_build_object('wedding_event_id', v_wedding_id, ...)
```

However, if `rpcResult.wedding_event_id` is null/undefined for any reason (e.g., Supabase client returns null data, RPC returns unexpected shape, schema cache hiccup), `result.weddingId` becomes falsy → silent failure in the client.

**Secondary observation:** `ReviewedWeddingSchema.safeParse` at the API boundary validates the submitted data. If this fails, a 400 is returned — but `!response.ok` handles that case and shows an error toast. The silent failure can only occur on a 200 response with missing `weddingId`.

---

## Files to Change

| File | Change | Reason |
|------|--------|--------|
| `frontend/src/app/[locale]/planner/imports/[id]/review/ImportReviewPageClient.tsx` | Fix success path guard | Always give user feedback on 200 response |
| `frontend/src/lib/services/import/wedding-creator.ts` | Add defensive logging | Diagnose root cause if `weddingId` is ever empty |

No schema changes. No migrations. No new dependencies.

---

## Exact Changes

### 1. `ImportReviewPageClient.tsx` — Fix silent success

**Current code (line ~62):**
```ts
// Redirect to the new wedding
if (result.data?.weddingId) {
  toast.success(t('successTitle'));
  router.push(`/${locale}/planner/weddings/${result.data.weddingId}`);
}
```

**Replace with:**
```ts
// Redirect to the new wedding
const weddingId = result.data?.weddingId;
toast.success(t('successTitle'));
if (weddingId) {
  router.push(`/${locale}/planner/weddings/${weddingId}`);
} else {
  // Wedding was created but weddingId missing — fall back to weddings list
  // This should not happen; log to help diagnose the root cause
  console.error('[create-wedding] API returned 200 but weddingId is empty:', result);
  router.push(`/${locale}/planner/weddings`);
}
```

**Effect:** Toast always fires on success. Navigation always happens (to the new wedding if we have the ID, to the weddings list as fallback). Console log helps us catch root cause in staging.

### 2. `wedding-creator.ts` — Add defensive logging (line ~313)

**Current code:**
```ts
result.weddingId = rpcResult.wedding_event_id
```

**Replace with:**
```ts
result.weddingId = rpcResult.wedding_event_id
if (!result.weddingId) {
  console.error('[wedding-creator] RPC succeeded but wedding_event_id is empty. Full rpcResult:', JSON.stringify(rpcResult))
}
```

**Effect:** If the RPC ever returns without a wedding ID, we'll see the full raw response in logs.

---

## Edge Cases

| Case | Before | After |
|------|--------|-------|
| Wedding created, `weddingId` present | Navigates to wedding ✅ | Same ✅ |
| Wedding created, `weddingId` empty/null | Silent — no feedback ❌ | Success toast + navigate to weddings list ✅ |
| API returns 400 (bad request) | Error toast ✅ | Same ✅ |
| API returns 409 (already processed) | Error toast ✅ | Same ✅ |
| API returns 500 (creation failed) | Error toast ✅ | Same ✅ |
| Network error | Error toast ✅ | Same ✅ |

---

## Tests

The existing E2E test suite doesn't appear to cover the create-wedding happy path end-to-end (that would require a running Supabase). No E2E test to update.

Add one unit/integration test in a new or existing test file:
- `handleCreateWedding` shows toast when API returns 200 with `weddingId`
- `handleCreateWedding` shows toast and navigates to weddings list when API returns 200 with empty `weddingId`
- `handleCreateWedding` shows error toast when API returns `!ok`

These can be written with `msw` or by mocking `fetch` directly.

---

## Risks / Regressions

- **None material.** We're widening the success path, not changing any data path. The only behavioral change is: user now always sees a toast on 200. Previously they'd see nothing on empty `weddingId` — that was a bug, not a feature.
- The `router.push` fallback to `/planner/weddings` is safe — it's where the user would go anyway to find their new wedding.

---

## Size estimate

~15 lines of code changed across 2 files. Genuinely XS.
