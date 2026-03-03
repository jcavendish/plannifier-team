# Ticket 13: Fix vendor financial empty state not rendering

**Status:** In Progress
**Assignee:** Qamar
**Severity:** Medium
**Size:** S
**Type:** Bug Fix
**Depends On:** —
**Sprint:** S06

## Summary

The "vendor financial empty state" component does not render when a vendor has no contracts. The `data-testid="vendor-financial-empty"` element is absent from the DOM even when expected — confirmed by E2E test `vendor-track-derived`: "vendor financial empty state renders on all vendor cards" failing with count = 0 (expected > 0).

## Problem

```
vendor-track-derived — "vendor financial empty state renders on all vendor cards"
  data-testid="vendor-financial-empty" count = 0 (expected > 0)
```

Fails consistently across multiple test runs. Pre-existing bug — not introduced by Sprint S05.

## Root Cause

Unknown — needs investigation. Likely candidates:
- Empty state branch guarded by a condition that is never falsy (e.g. checking `contracts.length === 0` but `contracts` defaults to `[]` and the guard is inverted or short-circuits)
- Component renders a different empty state UI without the correct `data-testid`
- Conditional rendering depends on a loading state that never resolves to "empty" (e.g. loading spinner stays visible instead of transitioning to empty state)

## Proposed Fix

1. Locate the vendor financial section component (likely in `frontend/src/components/vendors/`)
2. Find the empty state render path — trace the condition that should show `data-testid="vendor-financial-empty"`
3. Fix the condition or add the missing `data-testid` attribute
4. Verify with `vendor-track-derived` E2E test suite

## Acceptance Criteria

- [ ] `vendor-track-derived` E2E test "vendor financial empty state renders on all vendor cards" passes
- [ ] Empty state renders visually when a vendor has no contracts
- [ ] No regression on vendor financial section when contracts exist

## Complexity

S — likely a one-liner guard fix or missing `data-testid`, but needs investigation to confirm.

## Notes

- Discovered by Qamar during Sprint S05 E2E regression sweep (2026-03-03)
- Unrelated to import feature — all 6 import test files green
- Vendor financial feature was shipped in PRs #325, #326 (Sprint S04/VENDOR-FIN series)
