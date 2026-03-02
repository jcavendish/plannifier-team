# Ticket 5: Incremental Import for Existing Weddings

## Executive Summary

Enable planners to import vendors, guests, budget items, and timeline entries into **existing** weddings via document upload and AI extraction. The key difference from the current "new wedding" import flow is:

1. **No client/wedding creation** - the wedding already exists
2. **Merge logic** - match extracted entities against existing data, determine action (create/update/skip)
3. **Preview/Diff mode** - show planners what will happen before committing

---

## Design Decisions

### 1. API Design

**Decision**: Create a single unified API route with section as a path parameter.

```
POST /api/weddings/[weddingId]/import/[section]
```

Where `section` = `vendors` | `guests` | `budget` | `timeline`

**Rationale**:
- Each section has different matching logic and database operations
- Section-specific routes allow targeted permissions and rate limiting
- Single route pattern keeps the URL structure clean

**Two-Phase Flow**:
1. **Preview Phase**: `POST .../import/vendors?mode=preview` - returns diff without committing
2. **Commit Phase**: `POST .../import/vendors?mode=commit` - applies changes with overrides

This mirrors the existing flow where extraction happens first, then creation is a separate step.

### 2. Extraction Phase

**Decision**: Reuse the existing `/api/import/process` route for extraction.

**Flow**:
1. Frontend uploads files and calls `/api/import/process` (same as today)
2. Frontend receives `ExtractedWedding` data
3. Frontend calls `/api/weddings/[id]/import/[section]?mode=preview` with section-specific data
4. Preview route fetches existing wedding data, computes diff, returns preview
5. User reviews diff, can override actions per entity
6. Frontend calls `/api/weddings/[id]/import/[section]?mode=commit` with reviewed data + overrides

**Rationale**: The extraction pipeline (file upload, AI classification, AI extraction) is the same regardless of target wedding. Only the merge step differs.

### 3. Section Scope

**Decision**: Implement **vendors only** in this ticket. Add guests, budget, timeline as fast-follows.

**Rationale**:
- Vendors have the most complex matching (name, email, phone) and is already partially implemented in `conflict-resolver.ts`
- Scoping to one section reduces risk and allows faster iteration
- Other sections follow the same pattern, making follow-up tickets straightforward

---

## Merge Logic Design

### Vendor Matching (In Scope)

Existing logic in `conflict-resolver.ts`:
1. **Email match** (exact, case-insensitive) - highest priority
2. **Phone match** (E.164 normalized) - second priority
3. **Name match** (fuzzy, Levenshtein, 85% threshold) - third priority

**Actions**:
- `create` - No match found, insert new vendor
- `update` - Match found, fill empty fields ("gap-fill")
- `skip` - Match found, no fields to update (already complete)

### Guest Matching (Future - Ticket 5.1)

```typescript
// Match priority:
// 1. Email match (exact, case-insensitive)
// 2. Name + household match (fuzzy name within same household)

interface GuestMatchResult {
  action: 'create' | 'update' | 'skip'
  matchType?: 'email' | 'name_household'
  existingGuestId?: string
  fieldsToUpdate?: Record<string, unknown>
}
```

### Budget Item Matching (Future - Ticket 5.2)

```typescript
// Match priority:
// 1. Vendor name + category match
// 2. Description fuzzy match within category

interface BudgetMatchResult {
  action: 'create' | 'update' | 'skip'
  matchType?: 'vendor_category' | 'description'
  existingItemId?: string
  fieldsToUpdate?: Record<string, unknown>
}
```

### Timeline Matching (Future - Ticket 5.3)

```typescript
// Match priority:
// 1. Time + activity name match
// 2. Activity name fuzzy match

interface TimelineMatchResult {
  action: 'create' | 'update' | 'skip'
  matchType?: 'time_activity' | 'activity_name'
  existingEntryId?: string
  fieldsToUpdate?: Record<string, unknown>
}
```

---

## Implementation Plan

### Phase 1: Backend - Preview API

#### 1.1 Create new API route structure

**File**: `frontend/src/app/api/weddings/[id]/import/[section]/route.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server'
import { verifyPlannerAccess } from '@/lib/api/verifyPlannerAccess'
import { handleVendorImportPreview, handleVendorImportCommit } from '@/lib/services/import/incremental/vendor-merger'

export async function POST(
  request: NextRequest,
  context: { params: Promise<{ id: string; section: string }> }
) {
  const { id: weddingId, section } = await context.params
  const { searchParams } = new URL(request.url)
  const mode = searchParams.get('mode') as 'preview' | 'commit' || 'preview'

  // Verify planner access
  const accessResult = await verifyPlannerAccess(request, weddingId)
  if (\!accessResult.success) return accessResult.response

  const body = await request.json()

  switch (section) {
    case 'vendors':
      return mode === 'preview'
        ? handleVendorImportPreview(accessResult.supabase, weddingId, accessResult.user.id, body)
        : handleVendorImportCommit(accessResult.supabase, weddingId, accessResult.user.id, body)
    case 'guests':
    case 'budget':
    case 'timeline':
      return NextResponse.json({ error: 'Section not yet implemented' }, { status: 501 })
    default:
      return NextResponse.json({ error: 'Invalid section' }, { status: 400 })
  }
}
```

#### 1.2 Create vendor merger service

**File**: `frontend/src/lib/services/import/incremental/vendor-merger.ts`

```typescript
import { NextResponse } from 'next/server'
import type { SupabaseClient } from '@supabase/supabase-js'
import type { ExtractedVendor } from '../extraction-schemas'
import { findMatchingVendors, getVendorFieldsToUpdate, type DatabaseVendor } from '../conflict-resolver'

export interface VendorImportPreviewItem {
  extracted: ExtractedVendor
  action: 'create' | 'update' | 'skip'
  matchType?: 'email' | 'phone' | 'name'
  similarity?: number
  existingVendor?: DatabaseVendor
  fieldsToUpdate?: Record<string, unknown>
}

export interface VendorImportPreviewResponse {
  items: VendorImportPreviewItem[]
  summary: {
    create: number
    update: number
    skip: number
    total: number
  }
}

export interface VendorImportCommitRequest {
  vendors: ExtractedVendor[]
  overrides: Record<number, 'create' | 'update' | 'skip'> // index -> forced action
}

export async function handleVendorImportPreview(
  supabase: SupabaseClient,
  weddingId: string,
  plannerId: string,
  body: { vendors: ExtractedVendor[] }
): Promise<NextResponse> {
  // Fetch existing vendors for this planner
  const matches = await findMatchingVendors(supabase, plannerId, body.vendors)

  const items: VendorImportPreviewItem[] = body.vendors.map(extracted => {
    const match = matches.get(extracted.companyName)

    if (\!match) {
      return { extracted, action: 'create' }
    }

    const fieldsToUpdate = getVendorFieldsToUpdate(extracted, match.existingVendor)
    const hasUpdates = Object.keys(fieldsToUpdate).length > 0

    return {
      extracted,
      action: hasUpdates ? 'update' : 'skip',
      matchType: match.matchType,
      similarity: match.similarity,
      existingVendor: match.existingVendor,
      fieldsToUpdate: hasUpdates ? fieldsToUpdate : undefined,
    }
  })

  const summary = {
    create: items.filter(i => i.action === 'create').length,
    update: items.filter(i => i.action === 'update').length,
    skip: items.filter(i => i.action === 'skip').length,
    total: items.length,
  }

  return NextResponse.json({ items, summary })
}

export async function handleVendorImportCommit(
  supabase: SupabaseClient,
  weddingId: string,
  plannerId: string,
  body: VendorImportCommitRequest
): Promise<NextResponse> {
  const matches = await findMatchingVendors(supabase, plannerId, body.vendors)
  const results = { created: 0, updated: 0, skipped: 0 }

  for (let i = 0; i < body.vendors.length; i++) {
    const vendor = body.vendors[i]
    const override = body.overrides[i]
    const match = matches.get(vendor.companyName)

    let action: 'create' | 'update' | 'skip' = override || 'skip'
    if (\!override) {
      if (\!match) action = 'create'
      else {
        const updates = getVendorFieldsToUpdate(vendor, match.existingVendor)
        action = Object.keys(updates).length > 0 ? 'update' : 'skip'
      }
    }

    if (action === 'create') {
      const { error } = await supabase.from('vendors').insert({
        planner_id: plannerId,
        company_name: vendor.companyName,
        contact_person: vendor.contactName,
        email: vendor.email,
        phone: vendor.phone,
        service_category: vendor.serviceCategory || 'other',
        active: true,
      })
      if (\!error) results.created++
    } else if (action === 'update' && match) {
      const updates = getVendorFieldsToUpdate(vendor, match.existingVendor)
      if (Object.keys(updates).length > 0) {
        const { error } = await supabase
          .from('vendors')
          .update(updates)
          .eq('id', match.existingVendor.id)
          .eq('planner_id', plannerId)
        if (\!error) results.updated++
      }
    } else {
      results.skipped++
    }
  }

  return NextResponse.json({ success: true, results })
}
```

### Phase 2: Frontend - Import Trigger UI

#### 2.1 Add import button to WeddingVendorManager

**File**: `frontend/src/components/WeddingVendorManager.tsx` (modify)

Add import button in the card header actions area.

#### 2.2 Create IncrementalImportModal component

**File**: `frontend/src/components/documents/IncrementalImportModal.tsx` (new)

Multi-step modal: upload -> extract -> review -> commit -> complete

#### 2.3 Create IncrementalReviewScreen component

**File**: `frontend/src/components/documents/IncrementalReviewScreen.tsx` (new)

Shows diff table with:
- "Existing" column showing matched vendor details
- "Action" column with dropdown to override (create/update/skip)
- Summary stats at top
- Commit/Cancel buttons

---

## Files to Create

| File | Purpose |
|------|---------|
| `frontend/src/app/api/weddings/[id]/import/[section]/route.ts` | API route for preview/commit |
| `frontend/src/lib/services/import/incremental/vendor-merger.ts` | Vendor matching and merge logic |
| `frontend/src/lib/services/import/incremental/types.ts` | Shared types for incremental import |
| `frontend/src/components/documents/IncrementalImportModal.tsx` | Modal wrapper for import flow |
| `frontend/src/components/documents/IncrementalReviewScreen.tsx` | Diff review UI with overrides |
| `frontend/tests/incremental-vendor-import.critical.spec.ts` | E2E test |

## Files to Modify

| File | Changes |
|------|---------|
| `frontend/src/components/WeddingVendorManager.tsx` | Add import button, modal trigger |
| `frontend/messages/en.json` | Add i18n keys |
| `frontend/messages/pt-BR.json` | Add i18n keys (Portuguese) |

## Database Changes

**None required for vendors.** The existing `vendors` table and `conflict-resolver.ts` logic handle everything.

---

## Complexity Estimate

| Component | Effort | Notes |
|-----------|--------|-------|
| API route (preview + commit) | 4h | Route structure, handlers, validation |
| Vendor merger service | 3h | Mostly reusing conflict-resolver.ts |
| IncrementalImportModal | 3h | Multi-step flow, state management |
| IncrementalReviewScreen | 4h | Table with overrides, summary |
| WeddingVendorManager changes | 1h | Button + state |
| i18n keys | 1h | Both languages |
| E2E test | 2h | Setup, test cases |
| Manual QA + fixes | 2h | Edge cases, polish |

**Total: ~20 hours (2.5 days)**

---

## Trade-offs and Risks

### Trade-offs

1. **No transaction wrapper for commit** - Simpler implementation but partial failures possible
   - Mitigation: Return detailed results showing what succeeded/failed

2. **Re-computing matches on commit** - Could cache from preview, but risks stale data
   - Mitigation: Accept slight overhead for correctness

3. **Section-by-section import** - User must import vendors, then guests separately
   - Mitigation: Clear UX, each import is focused. Can add "import all" later.

### Risks

1. **Extraction quality** - AI may extract poor data
   - Mitigation: Clear diff UI lets planner review and override

2. **Performance with many vendors** - `findMatchingVendors` loads all planner vendors
   - Mitigation: Pagination or lazy matching if vendor count > 500 (rare)

---

## Future Work (Out of Scope)

- **Ticket 5.1**: Guest Import (matching by email, name+household)
- **Ticket 5.2**: Budget Import (matching by vendor+category)
- **Ticket 5.3**: Timeline Import (matching by time+activity)
- **Ticket 5.4**: Multi-Section Import ("import all" option)

---

## Testing Approach

### Unit Tests
- Preview with no matches -> all "create"
- Preview with exact email match -> "skip"
- Preview with name match, missing fields -> "update"
- Override from "skip" to "create"

### E2E Tests
1. Happy path: Upload PDF -> Extract -> Review -> Commit -> Vendors appear
2. Cancel flow: Upload -> Extract -> Cancel -> No changes
3. Override flow: Change "skip" to "create" -> New vendor created

### QA Validation Checklist
- [ ] Build passes (pnpm run build)
- [ ] Lint passes (pnpm run lint)
- [ ] Desktop screenshot: Import button visible
- [ ] Desktop screenshot: Modal with diff table
- [ ] Mobile screenshot (390x844): Modal responsive
- [ ] Test with 0 matches (all create)
- [ ] Test with partial matches
- [ ] Test override functionality
- [ ] Test network error handling

---

## Implementation Order

1. Backend: API route + vendor-merger service
2. Frontend: IncrementalReviewScreen (can test with mock data)
3. Frontend: IncrementalImportModal (connects extraction + review)
4. Frontend: WeddingVendorManager integration
5. i18n: Add all keys
6. E2E test
7. QA validation
