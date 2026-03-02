# Ticket 6: Review UI redesign — swipable per-type cards

**Status:** Done (merged — follow-up required)
**Assignee:** Devon (follow-up: add ?scenario=vendor-update to test page)
**PR:** https://github.com/jcavendish/plannifier/pull/337 (merged 2026-03-02T11:53Z)
**Plan Status:** Approved
**Latest Update:** QA re-test complete 2026-03-02 — 58/63 pass, 5 failures need follow-up fix
**Severity:** Medium
**Size:** M
**Type:** Refactor
**Depends On:** Tickets 2-3 (schema expansion)

## Summary

Redesign review screen from single-scroll page to swipable cards/tabs per entity type. Needed for scalability as schema expands to 10+ entity types.

## Current UX

- One long page: scroll through all vendors, then all guests, then all budget
- Hard to navigate when 100+ entities
- Hard to see overall progress

## Target UX

- Card/tab per entity type
- Swipe left/right (mobile) or tab buttons (desktop)
- Show "3 of 5 cards reviewed" progress
- Each card shows entities of that type with inline edit, accept/reject
- Visual confidence badges on each entity
- Ability to collapse reviewed types

## Changes Required

### 1. UI Components

- Create `ReviewCardStack` component with swipe/tab navigation
- Refactor existing review components into per-type cards
- Add progress indicator
- Add skip-type button

### 2. Navigation

- Keyboard: arrow keys, tab
- Mobile: swipe gestures (detect left/right drag)
- Desktop: prev/next buttons

### 3. useDocumentImport.ts

- Track active card type
- Track card-level state (expanded, collapsed, reviewed)

## Design Notes

- Mobile-first: cards should be full-width on mobile, narrower on desktop
- Each card shows count badge: "Vendors (3)" or "Guests (15)"
- Skip button for types with no extracted data
- Confidence color-coding on entities

## Files Changed

- Import review UI components
- `useDocumentImport.ts`

## Related Issues

- #18

## Epic

Import Feature - UI Enhancement

## Timing

Should follow Tickets 2-3 so we're designing for final schema

---

## QA Results — 2026-03-01 (initial)

**Status:** ❌ FAIL — returned to Devon

**Build/Lint/Unit:** ✅ All pass (build clean, 959/959 unit tests, 0 lint errors introduced)

**High severity issues found:**

1. **Existing E2E tests broken** — `document-import-review.critical.spec.ts` not updated for card-stack architecture:
   - `'review screen renders with all entity sections'` — expects all sections in DOM at once; new UI renders only active tab's section
   - `'entity sections show correct item counts'` — same root cause
   - 5× `'section can be expanded and collapsed'` — expects collapsed by default; new code uses `defaultExpanded={true}`
   - `'confidence indicators visible on sections'` — checks multiple sections simultaneously

**New E2E tests written by Qamar:** `tests/document-import-review-card-nav.critical.spec.ts` (11 tests)

**Action required from Devon:** Update `document-import-review.critical.spec.ts` to navigate to each tab before asserting section content, and invert expand/collapse initial state assertions.

---

## QA Results — 2026-03-02 (re-test by Qamar)

**Status:** ⚠️ MERGED WITH OPEN ISSUE — follow-up fix needed

PR merged to staging at 11:53 before re-test was complete.

**Build / Lint / Type-check / Unit tests:** ✅ All pass (967/967 unit tests)

**E2E: 58 / 63 pass**

### ✅ All passing
- `document-import-review.critical.spec.ts` — all updated tests pass
- `document-import-review-card-nav.critical.spec.ts` (11 new tests) — all pass
- `document-import-vendor-contracts.critical.spec.ts` — all pass

### ❌ 5 failures in `document-import-vendor-update.critical.spec.ts`

**Root cause:** Tests use `?scenario=vendor-update` but the test helper page (`src/app/[locale]/test/import-review/page.tsx`) never implements this scenario — vendors have no `catalogMatch` data, so badges/dismiss button are never rendered.

**Severity:** Medium — test infrastructure only; the Ticket 4 feature itself is unaffected in production. The previous "QA PASS" note in this file referencing commit `cdced5f` was inaccurate (no such commit exists in staging).

**Follow-up required from Devon:** Add `?scenario=vendor-update` to the test helper page populating vendors with `catalogMatch` data. Full details in PR #337 comment.
