# Ticket 17: Source documents not linked to wedding after import creation

**Status:** Open
**Assignee:** —
**Severity:** Medium
**Size:** L
**Type:** Feature
**Depends On:** Tickets 14, 15, 16 (import pipeline stable)

## Summary

When a wedding is created from an import, the original uploaded PDF/document files are not attached to the wedding, vendors, or budget items. Planners lose the connection between the raw source documents and the entities that were created from them. There is also no UI to review or manage attached documents inside a wedding.

## Problem

`wedding-creator.ts` has no code to copy, link, or reference the original uploaded files when creating wedding entities. The only partial tracking is `sourceFileId` on `vendor_contracts`, which carries the Supabase storage path of the source file — but even this isn't surfaced anywhere in the wedding UI.

Specifically missing:
1. No code to copy or move uploaded files from `import-files/` bucket to a wedding-scoped path in storage
2. No DB records linking `vendors` or `budget_items` to source documents
3. No tab or section in the wedding detail view to see attached documents
4. No review screen in the import wizard for document-to-entity linking (the planner cannot see/change which document maps to which vendor or contract)

## Scope (to be refined with Joao/Polly)

This is a large feature. Suggested breakdown:

### Phase 1 — Storage linkage (minimum viable)
- On wedding creation, copy/move each import file from `import-files/{userId}/{importId}/` to a wedding-scoped path: `wedding-documents/{weddingId}/`
- Record the storage path in a new `wedding_documents` table (or reuse existing `vendor_documents` model), with FK to `wedding_events`
- The `sourceFileId` on `vendor_contracts` should reference this record

### Phase 2 — Entity linkage
- Link `vendors` → `wedding_documents` (which document this vendor came from)
- Link `budget_items` → `wedding_documents`
- This allows "View source document" from a vendor or budget item card

### Phase 3 — Review UI in import wizard
- Add a "Source Documents" step or panel to the import review screen
- Shows each uploaded file, its classification, and which entities were derived from it
- Allows the planner to re-classify a document or exclude it before committing

### Phase 4 — Documents tab in wedding UI
- Add a "Documents" tab to the wedding detail page
- Lists all attached files with download links, classification, and derived entities

## Proposed Fix

Scope with Joao before estimating. Phase 1 is likely the right first cut (storage + basic DB linkage). Phases 2-4 can follow in separate tickets.

## Acceptance Criteria (Phase 1 minimum)

- [ ] After wedding creation, uploaded documents appear in `wedding-documents/{weddingId}/` in Supabase storage
- [ ] A DB record exists linking each document to the wedding
- [ ] `vendor_contracts.sourceFileId` correctly references the migrated storage path
- [ ] Old import files are cleaned up from `import-files/` after successful wedding creation

## Files

- `frontend/src/lib/services/import/wedding-creator.ts`
- `frontend/src/app/api/import/create-wedding/route.ts`
- `supabase/migrations/` — new migration for `wedding_documents` table
- `frontend/src/components/documents/ImportReviewScreen.tsx` (Phase 3)
- `frontend/src/app/[locale]/planner/weddings/[id]/` (Phase 4)

## Latest Update

Open. Needs scoping discussion before any implementation starts.
