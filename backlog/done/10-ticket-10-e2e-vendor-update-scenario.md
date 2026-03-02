# Ticket 10: Fix E2E — missing `?scenario=vendor-update` on test page

**Status:** Done
**Assignee:** —
**Severity:** Medium | **Size:** XS | **Type:** Bug Fix
**Depends On:** Ticket 6 (landed)
**PR:** https://github.com/jcavendish/plannifier/pull/340
**Latest Update:** Merged to staging 2026-03-02 (PR #340).

## Summary

5 E2E tests in `document-import-vendor-update.critical.spec.ts` were failing because the test helper page does not implement `?scenario=vendor-update`. As a result, vendors never receive `catalogMatch` data and the match/update badges never render — causing assertion failures on badge visibility.

## Fix

Test infrastructure only — no production code changed. Added `?scenario=vendor-update` query parameter support to the import review test helper page, with two vendors populated with `catalogMatch` data to exercise the badge rendering paths.

## QA Results

- Lint: ✅ 0 errors
- Type-check: ✅ 0 errors  
- Unit tests: ✅ 959/959
- E2E: ✅ 7/7 pass (`document-import-vendor-update.critical.spec.ts`)
