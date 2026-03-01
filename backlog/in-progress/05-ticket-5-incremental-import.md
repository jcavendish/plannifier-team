# Ticket 5: Incremental import for existing weddings

**Status:** In Progress
**Assignee:** Devon
**Severity:** Medium
**Size:** L
**Type:** Feature
**Depends On:** Tickets 1-4 (foundational)

## Summary

Enable per-section import into existing weddings. Currently, import only creates new weddings. Planners need to backfill missing data into manually-created weddings (guests, vendors, budgets). Includes preview/diff mode showing what will create/update/skip before committing.

## Current State

- Import creates new wedding only
- Planners who manually set up weddings cannot backfill data

## Goal

- Browse to existing wedding
- Click "Import vendors" in vendors section
- Upload contract PDFs
- See diff: "3 new vendors, 2 vendor updates, 1 already exists"
- Commit or cancel

## Changes Required

### 1. New API Route

- `/api/weddings/[weddingId]/import/[section]` (section = vendors|guests|budget|timeline)
- Process files like main import, but merge against wedding's existing data

### 2. Merge Logic (`wedding-creator.ts`)

- For vendors: match by email/phone/name, compute create/update/skip actions
- For guests: match by email, compute upsert
- For budget: match by vendor name + category, compute upsert
- For timeline: match by event date, compute upsert

### 3. Frontend UI

- Add import buttons to Vendors, Guests, Budget sections
- Route to upload modal
- Show review screen with diff against existing data
- Preview: "Update 2 vendors", "Skip 1 (already exists)", "Create 3 new"

### 4. Review Screen Enhancement (ties to #21)

- Add "Existing" column: show what's already there
- Add "Action" column: create/update/skip per entity
- Allow per-entity override (skip this vendor, force-create, etc.)

## Files Changed

- New API route
- Frontend import UI
- `wedding-creator.ts` (merge logic)

## Related Issues

- #22, #21

## Epic

Import Feature - P2 (deferred)

## Dependencies

Tickets 1-4 must be solid first (conflict resolution, update logic, schema expansion)
