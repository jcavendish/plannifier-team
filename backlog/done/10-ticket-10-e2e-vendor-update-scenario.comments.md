# Comments: Ticket 10 — Fix E2E `?scenario=vendor-update`

### Devon — 2026-03-02 13:40

Implementation complete on branch `fix/ticket-6-vendor-update-scenario`.

**What changed** (single file: `frontend/src/app/[locale]/test/import-review/page.tsx`):
- Added `Suspense` + `useSearchParams` (Next.js 15 requires Suspense boundary for searchParams in client components)
- Extracted page body to `ImportReviewTestPageContent` (child component that reads searchParams); outer `ImportReviewTestPage` just wraps it in `<Suspense fallback={null}>`
- Added `catalogMatch: null` to existing mock vendors (required by updated TypeScript schema)
- Added `vendorUpdateCatalogVendors` — two `DatabaseVendor` entries: vendor 0 has `phone: null` + `service_category: ''` (gaps → blue badge), vendor 1 has all fields populated (no gaps → green badge)
- Added `vendorUpdateExtractedData` — spreads `mockExtractedData` but overrides vendors with `catalogMatch` populated
- Added `handleDismissMatch` handler — clears `catalogMatch` and `linkedCatalogVendorId` from the vendor at the given index
- Passed `catalogVendors` and `onDismissMatch` to `ImportReviewScreen` (both were already optional props)
- All existing handlers preserved; no production code changed

**Badge logic verification:**
- Vendor 0: `isAutoMatched=true`, `getVendorFieldsToUpdate → {phone, service_category}` → blue "Will update" badge ✅
- Vendor 1: `isAutoMatched=true`, `getVendorFieldsToUpdate → {}` → green "Already in catalog" badge ✅
- After dismiss: `catalogMatch: null` → `isAutoMatched=false` → badges gone, row editable ✅

**Build checks:** type-check ✅ lint ✅ (0 errors introduced)

Handing to Qamar for QA re-run of the 5 failing specs.

### Qamar — 2026-03-02 13:55

**QA PASSED**

- Type-check ✅ | Lint ✅ | Single test-only file changed ✅
- `document-import-vendor-update.critical.spec.ts`: **7/7 pass** (all 5 previously-failing tests now pass)
- `document-import-review.critical.spec.ts`: **36/36 pass** (regression clean)
- Code review: Suspense boundary correct, fixture data matches badge semantics, dismiss handler correct

Ready to merge (after PR #337 lands).
