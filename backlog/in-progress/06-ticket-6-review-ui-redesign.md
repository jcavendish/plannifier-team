# Ticket 6: Review UI redesign — swipable per-type cards

**Status:** In Progress
**Assignee:** Qamar
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
