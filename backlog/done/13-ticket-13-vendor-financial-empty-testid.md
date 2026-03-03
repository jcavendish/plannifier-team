# Ticket 13: Fix vendor-financial-empty data-testid on empty state

**Status:** Done
**Assignee:** Review: Joao
**Severity:** Medium | **Size:** XS | **Type:** Bug Fix
**PR:** https://github.com/jcavendish/plannifier/pull/343
**Latest Update:** QA PASS (Qamar, 2026-03-03) — 12/12 vendor-track-derived tests pass. Ready for Joao review.

## Summary

`VendorFinancialSummary.tsx` empty state branch rendered `data-testid="vendor-financial-summary-{id}"` instead of the expected `data-testid="vendor-financial-empty"`. This caused 3 tests in `vendor-track-derived.critical.spec.ts` to fail with count = 0.

## Fix

One-line change in `VendorFinancialSummary.tsx`: corrected the testid on the empty state div.

## QA Results

- `vendor-track-derived.critical.spec.ts`: ✅ 12/12 pass (all 3 previously-failing empty-state tests now green)
