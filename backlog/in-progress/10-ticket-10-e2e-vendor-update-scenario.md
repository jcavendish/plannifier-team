# Ticket 10: Fix E2E — missing `?scenario=vendor-update` on test page

**Status:** In Progress
**Assignee:** Qamar
**PR:** https://github.com/jcavendish/plannifier/pull/340 (draft)
**Latest Update:** Dev complete — PR #340 open, handed to Qamar for QA
**Severity:** Medium
**Size:** XS
**Type:** Bug Fix
**Depends On:** Ticket 6 (landed)

## Summary

5 E2E tests in `document-import-vendor-update.critical.spec.ts` are failing because the test helper page does not implement `?scenario=vendor-update`. As a result, vendors never receive `catalogMatch` data and the match/update badges never render — causing assertion failures on badge visibility.

No production code is broken. This is a test infrastructure gap exposed by Ticket 6's new UI.

## Root Cause

The test helper page handles some `?scenario=` query params but `vendor-update` was never wired up. Tests that rely on it get an uninitialized state, so `catalogMatch` is always null and the badge conditionals are never truthy.

## Failing Tests

- `document-import-vendor-update.critical.spec.ts` — 5/5 tests failing
- Full repro details in PR #337 comment (Qamar's re-test notes)

## Proposed Fix

Add `vendor-update` scenario handling to the test helper page:
1. Locate the `?scenario=` switch in the test page component
2. Add a `vendor-update` case that seeds vendors with `catalogMatch` populated (use existing matched vendor fixture)
3. Re-run the 5 failing specs to confirm they pass

## Files Likely Changed

- `frontend/tests/helpers/` or similar test page/fixture setup
- Possibly a fixture file for vendor-update scenario seed data

## Acceptance Criteria

- All 5 previously failing tests in `document-import-vendor-update.critical.spec.ts` pass
- No regression in other E2E suites
- No production code changes required
