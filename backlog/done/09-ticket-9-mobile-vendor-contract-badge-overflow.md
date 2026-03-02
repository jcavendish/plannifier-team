# Ticket 9: Mobile layout — vendor name hidden behind contract badges

**Status:** Done
**Assignee:** —
**PR:** https://github.com/jcavendish/plannifier/pull/336 (merged 2026-03-02)
**Latest Update:** QA passed (Qamar, 9/9 E2E) — merged to staging 2026-03-02
**Severity:** Medium
**Size:** XS
**Type:** Bug Fix
**Depends On:** Ticket 3 (landed)

## Summary

On small viewports (≤390px), when a vendor contract card shows both the "Source document attached" badge AND a status badge simultaneously, the vendor name `<span>` collapses to width:0 and becomes invisible.

## Root Cause

In `ImportReviewVendorContractsSection.tsx`, the header row uses:
- Left side: `flex items-center gap-2 min-w-0` with `<span className="font-medium truncate">` for the vendor name
- Right side: `flex items-center gap-2 shrink-0` for the badges

The `shrink-0` on the right div prevents it from compressing, but when two badges are present on a narrow screen they consume all available width, leaving the left flex child (even with `min-w-0`) with zero effective space before truncation kicks in.

## Fix Applied

Option A — badges wrap to a second line on narrow screens:
```tsx
<div className="flex flex-wrap items-center gap-2 justify-end">
```

## Files Changed

- `frontend/src/components/documents/ImportReviewVendorContractsSection.tsx`

## Found by

Qamar — QA Round 2 on Ticket 3 (non-blocking medium, deferred to follow-up)

## Epic

Import Feature — Polish

---

## QA Results — 2026-03-02

**Status:** ✅ PASS

**Build / Lint / Type-check / Unit tests:** ✅ All pass (967/967 unit tests)

**E2E: 9/9 pass** — `document-import-vendor-contracts.critical.spec.ts`

Key tests verified:
- Mobile viewport (390px): all 3 contract cards visible, no horizontal overflow ✅
- pt-BR locale: labels and badges correctly localised ✅
- Terms expand/collapse: functional ✅
- Source document badge visibility: correct ✅

No new E2E tests needed — existing mobile test covers the fixed flow.
