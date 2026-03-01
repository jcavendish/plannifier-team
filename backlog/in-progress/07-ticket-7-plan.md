# Ticket 7: Housekeeping & Observability — Implementation Plan

## Executive Summary

Four independent improvements to the document import feature. All are **low-risk, low-complexity** and can be completed in a Size:S ticket with the recommended scope below.

---

## 1. Import Analytics (#23)

### Analysis

The `document_imports` table already tracks `file_count` and `total_size_bytes`. Adding analytics columns directly to this table is simpler than creating a separate table (avoids JOINs for basic reporting).

### Schema Changes

**File:** `supabase/migrations/YYYYMMDDHHMMSS_add_import_analytics.sql`

```sql
ALTER TABLE public.document_imports
  ADD COLUMN entity_counts jsonb,           -- {"vendors": 5, "guests": 120, ...}
  ADD COLUMN avg_confidence numeric(3,2),   -- 0.00-1.00 scale
  ADD COLUMN processing_time_ms integer,    -- extraction wall-clock time
  ADD COLUMN token_usage jsonb,             -- {"input": 50000, "output": 2000, "total": 52000}
  ADD COLUMN estimated_cost_usd numeric(6,4); -- estimated API cost

COMMENT ON COLUMN public.document_imports.entity_counts IS 'Count of extracted entities by type: {vendors, guests, budgetItems, tasks, ...}';
COMMENT ON COLUMN public.document_imports.processing_time_ms IS 'Wall-clock time for AI extraction phase in milliseconds';
COMMENT ON COLUMN public.document_imports.token_usage IS 'Cumulative token usage across all files: {input, output, total}';
COMMENT ON COLUMN public.document_imports.estimated_cost_usd IS 'Estimated API cost based on token usage and model pricing';
```

### Code Changes

**File to modify:** `frontend/src/app/api/import/process/route.ts`

1. Track start time before `processAllFiles()`
2. Aggregate token usage from provider responses
3. Compute entity counts from merged result
4. Compute average confidence across all entities
5. Update `document_imports` in `finalizeImport()` with analytics data

**New types:**

```typescript
interface ImportAnalytics {
  entityCounts: Record<string, number>;
  avgConfidence: number;
  processingTimeMs: number;
  tokenUsage: { input: number; output: number; total: number };
  estimatedCostUsd: number;
}
```

### Pricing Constants

```typescript
// Haiku 4.5 pricing (2026 rates)
const HAIKU_INPUT_PRICE_PER_M = 1.0;   // $1.00 per 1M input tokens
const HAIKU_OUTPUT_PRICE_PER_M = 5.0;  // $5.00 per 1M output tokens
```

---

## 2. Token Budget Ceiling (#23)

### Analysis

Current extraction providers (`ClaudeExtractionProvider`, `ClaudeCliExtractionProvider`) don't expose token usage. The Anthropic SDK response includes `usage.input_tokens` and `usage.output_tokens` which we need to track.

**Challenge:** The CLI provider shells out to `claude` CLI and doesn't have direct token access. We'll need to estimate based on character counts for the CLI provider.

### Design Decisions

- **Ceiling:** $1.00 USD per import (configurable via env var)
- **Abort mechanism:** Check cumulative cost after each file extraction; if exceeded, skip remaining files and mark import as partially complete
- **80% warning:** Add to extraction response, surface in frontend review screen

### Code Changes

**File to modify:** `frontend/src/lib/services/import/extraction-provider.ts`

Add to `ExtractionResult` interface (new interface wrapping extraction response):

```typescript
interface ExtractionResult {
  data: ExtractedWedding;
  tokenUsage: {
    input: number;
    output: number;
    total: number;
    estimatedCostUsd: number;
  };
}
```

**File to modify:** `frontend/src/lib/services/import/claude-cli-extraction-provider.ts`

- Estimate tokens from character count (rough: ~4 chars per token for English/Portuguese)
- Return `ExtractionResult` with estimated usage

**File to modify:** `frontend/src/app/api/import/process/route.ts`

```typescript
const TOKEN_BUDGET_USD = parseFloat(process.env.IMPORT_TOKEN_BUDGET_USD || '1.00');
const WARNING_THRESHOLD = 0.8; // 80%

// In processAllFiles():
let cumulativeCostUsd = 0;
let budgetWarningTriggered = false;

for (const batch of fileBatches) {
  // ... existing processing ...

  cumulativeCostUsd += result.tokenUsage.estimatedCostUsd;

  if (cumulativeCostUsd >= TOKEN_BUDGET_USD) {
    // Abort remaining files
    console.warn(`Token budget exceeded: $${cumulativeCostUsd.toFixed(4)} > $${TOKEN_BUDGET_USD}`);
    break;
  }

  if (\!budgetWarningTriggered && cumulativeCostUsd >= TOKEN_BUDGET_USD * WARNING_THRESHOLD) {
    budgetWarningTriggered = true;
    // Include warning in response
  }
}
```

**File to modify:** `frontend/src/app/api/import/process/route.ts` (response)

Add `budgetWarning` field to response:

```typescript
return NextResponse.json({
  data: mergedWedding,
  stats: { ... },
  budgetWarning: budgetWarningTriggered
    ? `This import used ${Math.round(cumulativeCostUsd / TOKEN_BUDGET_USD * 100)}% of the token budget`
    : undefined,
  fileErrors: fileErrors.length > 0 ? fileErrors : undefined,
});
```

### Frontend Changes (Deferred)

The 80% warning UI update can be deferred to a follow-up ticket. For now, the warning is logged and returned in the API response but not displayed prominently in the UI.

---

## 3. File Retention Cleanup (#23)

### Analysis

- Files are stored in Supabase Storage bucket `documents`
- `pg_cron` is already enabled and used for other scheduled jobs (RSVP reminders, payment status updates)
- Existing pattern: `cron.schedule()` daily jobs

### Schema Changes

**File:** `supabase/migrations/YYYYMMDDHHMMSS_add_import_file_retention.sql`

```sql
-- Add retention column with 48-hour default
ALTER TABLE public.document_import_files
  ADD COLUMN retention_expires_at timestamptz
    DEFAULT (now() + interval '48 hours');

-- Backfill existing rows (if any)
UPDATE public.document_import_files
SET retention_expires_at = created_at + interval '48 hours'
WHERE retention_expires_at IS NULL;

-- Index for cleanup queries
CREATE INDEX idx_document_import_files_retention
  ON public.document_import_files(retention_expires_at)
  WHERE retention_expires_at IS NOT NULL;

-- Cleanup function
CREATE OR REPLACE FUNCTION public.cleanup_expired_import_files()
RETURNS TABLE(deleted_count bigint, storage_paths_deleted text[])
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = ''
AS $$
DECLARE
  v_storage_paths text[];
  v_deleted_count bigint;
BEGIN
  -- Get storage paths of files to delete
  SELECT array_agg(storage_path)
  INTO v_storage_paths
  FROM public.document_import_files
  WHERE retention_expires_at < now();

  -- Delete the database records (storage deletion handled separately)
  DELETE FROM public.document_import_files
  WHERE retention_expires_at < now();

  GET DIAGNOSTICS v_deleted_count = ROW_COUNT;

  RETURN QUERY SELECT v_deleted_count, COALESCE(v_storage_paths, ARRAY[]::text[]);
END;
$$;

COMMENT ON FUNCTION public.cleanup_expired_import_files() IS
  'Deletes import files past retention (48h). Returns paths for storage cleanup. Run via pg_cron.';

GRANT EXECUTE ON FUNCTION public.cleanup_expired_import_files() TO service_role;

-- Schedule daily cleanup at 3 AM UTC
SELECT cron.schedule(
  'cleanup-import-files-daily',
  '0 3 * * *',
  $$SELECT public.cleanup_expired_import_files();$$
);
```

### Storage Cleanup Strategy

The pg_cron function returns `storage_paths_deleted` but doesn't directly delete from Supabase Storage (RPC can't call storage API). Two options:

**Option A (Recommended):** Edge Function triggered by pg_cron
- Create a Supabase Edge Function that:
  1. Calls `cleanup_expired_import_files()`
  2. Uses the returned paths to delete from storage

**Option B (Simpler):** Vercel Cron API route
- Create `/api/cron/cleanup-import-files`
- Schedule via `vercel.json` (like existing RSVP reminders)
- Uses service role client for both DB and storage cleanup

For Size:S, **Option B** is simpler and follows existing patterns. The API route handles both DB and storage cleanup atomically.

**File to create:** `frontend/src/app/api/cron/cleanup-import-files/route.ts`

```typescript
export const dynamic = 'force-dynamic';
export const runtime = 'nodejs';

export async function GET(request: NextRequest) {
  // Verify Vercel Cron secret
  const authHeader = request.headers.get('authorization');
  if (authHeader \!== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const supabase = createServiceRoleClient();

  // Get expired files
  const { data: expiredFiles } = await supabase
    .from('document_import_files')
    .select('id, storage_path')
    .lt('retention_expires_at', new Date().toISOString());

  if (\!expiredFiles?.length) {
    return NextResponse.json({ deleted: 0 });
  }

  // Delete from storage
  const storagePaths = expiredFiles.map(f => f.storage_path);
  await supabase.storage.from('documents').remove(storagePaths);

  // Delete DB records
  const ids = expiredFiles.map(f => f.id);
  await supabase.from('document_import_files').delete().in('id', ids);

  return NextResponse.json({ deleted: expiredFiles.length });
}
```

**File to modify:** `frontend/vercel.json` (add cron entry)

```json
{
  "crons": [
    {
      "path": "/api/cron/cleanup-import-files",
      "schedule": "0 3 * * *"
    }
  ]
}
```

---

## 4. Cross-Document Vendor Name Dedup Heuristic (#13)

### Analysis

**Problem:** Two vendor names with zero string overlap but same phone/email:
- "CM Oliveira comércio de bolos" (legal name from contract)
- "CASSIA PEREIRA BOLOS ARTISTICOS" (brand name from receipt)
- Same phone, same category (Catering) → should be merged

**Current flow in `extraction-merger.ts`:**
1. `deduplicateVendors()` uses `areVendorsSimilar()` (Levenshtein on names)
2. Fails for the above case because names have 0% similarity

**Solution:** Add a second pass after Levenshtein that checks for:
- Same `serviceCategory` AND
- Matching contact info (phone OR email)

### Code Changes

**File to modify:** `frontend/src/lib/services/import/extraction-merger.ts`

Add after existing `deduplicateVendors()` function:

```typescript
/**
 * Normalize phone for comparison (remove formatting)
 */
function normalizePhone(phone: string | null | undefined): string | null {
  if (\!phone) return null;
  // Remove all non-digits
  const digits = phone.replace(/\D/g, '');
  // Brazilian phones: keep last 11 digits (DDD + number)
  if (digits.length >= 11) return digits.slice(-11);
  if (digits.length >= 9) return digits.slice(-9); // Mobile without DDD
  return digits.length >= 8 ? digits : null;
}

/**
 * Normalize email for comparison (lowercase, trim)
 */
function normalizeEmail(email: string | null | undefined): string | null {
  if (\!email) return null;
  const normalized = email.toLowerCase().trim();
  return normalized.includes('@') ? normalized : null;
}

/**
 * Check if two vendors share contact info (phone or email)
 */
function vendorsShareContact(a: ExtractedVendor, b: ExtractedVendor): boolean {
  const aPhone = normalizePhone(a.phone);
  const bPhone = normalizePhone(b.phone);
  if (aPhone && bPhone && aPhone === bPhone) return true;

  const aEmail = normalizeEmail(a.email);
  const bEmail = normalizeEmail(b.email);
  if (aEmail && bEmail && aEmail === bEmail) return true;

  return false;
}

/**
 * Check if two vendors are in the same service category
 */
function vendorsSameCategory(a: ExtractedVendor, b: ExtractedVendor): boolean {
  const aCat = a.serviceCategory?.toLowerCase().trim();
  const bCat = b.serviceCategory?.toLowerCase().trim();
  if (\!aCat || \!bCat) return false;
  return aCat === bCat || areNamesSimilar(aCat, bCat, 0.8);
}

/**
 * Heuristic dedup: merge vendors with same category AND shared contact info.
 * Catches cases where Levenshtein fails (legal name vs brand name, zero overlap).
 *
 * Runs AFTER the standard Levenshtein pass.
 */
function heuristicVendorMerge(vendors: ExtractedVendor[]): ExtractedVendor[] {
  if (vendors.length <= 1) return vendors;

  const result: ExtractedVendor[] = [];
  const processed = new Set<number>();

  for (let i = 0; i < vendors.length; i++) {
    if (processed.has(i)) continue;

    let merged = vendors[i];

    for (let j = i + 1; j < vendors.length; j++) {
      if (processed.has(j)) continue;

      // Same category AND shared contact → merge
      if (vendorsSameCategory(merged, vendors[j]) && vendorsShareContact(merged, vendors[j])) {
        console.log(`[heuristic-merge] Merging by contact: "${merged.companyName}" + "${vendors[j].companyName}"`);
        merged = mergeVendors(merged, vendors[j]);
        processed.add(j);
      }
    }

    result.push(merged);
    processed.add(i);
  }

  return result;
}
```

**Modify `mergeExtractedWeddings()`:**

```typescript
export function mergeExtractedWeddings(documents: ExtractedWedding[]): ExtractedWedding {
  // ... existing code ...

  // Collect all vendors from all documents
  const allVendors = documents.flatMap((doc) => doc.vendors);

  // Pass 1: Levenshtein-based dedup (existing)
  const afterLevenshtein = deduplicateVendors(allVendors);

  // Pass 2: Heuristic merge by contact info (new)
  const deduplicatedVendors = heuristicVendorMerge(afterLevenshtein);

  // ... rest of function unchanged ...
}
```

### Testing

Add test case for the heuristic merge:

```typescript
// In extraction-merger.test.ts
it('merges vendors with zero name overlap but same phone and category', () => {
  const vendors: ExtractedVendor[] = [
    { companyName: 'CM Oliveira comércio de bolos', phone: '11999998888', serviceCategory: 'Catering', ... },
    { companyName: 'CASSIA PEREIRA BOLOS ARTISTICOS', phone: '(11) 99999-8888', serviceCategory: 'Catering', ... },
  ];

  const merged = heuristicVendorMerge(vendors);
  expect(merged).toHaveLength(1);
  // Higher confidence wins
});
```

---

## Summary: Files to Create/Modify

### New Files

| File | Purpose |
|------|---------|
| `supabase/migrations/YYYYMMDDHHMMSS_add_import_analytics.sql` | Analytics columns on document_imports |
| `supabase/migrations/YYYYMMDDHHMMSS_add_import_file_retention.sql` | Retention column + cleanup function |
| `frontend/src/app/api/cron/cleanup-import-files/route.ts` | Vercel Cron handler for storage cleanup |

### Files to Modify

| File | Changes |
|------|---------|
| `frontend/src/lib/services/import/extraction-provider.ts` | Add `ExtractionResult` type with token usage |
| `frontend/src/lib/services/import/claude-cli-extraction-provider.ts` | Return estimated token usage |
| `frontend/src/app/api/import/process/route.ts` | Track analytics, token budget, abort logic |
| `frontend/src/lib/services/import/extraction-merger.ts` | Add `heuristicVendorMerge()` |
| `frontend/vercel.json` | Add cleanup cron job |

---

## Edge Cases & Risks

### Token Budget
- **Risk:** CLI provider token estimation is imprecise
- **Mitigation:** Use conservative 3.5 chars/token ratio; log actual vs estimated when SDK is used

### File Retention
- **Risk:** Files deleted before import completion
- **Mitigation:** 48h retention is generous; imports typically complete in < 5 minutes
- **Risk:** Orphaned storage files if DB deletion succeeds but storage fails
- **Mitigation:** Log storage deletion failures; manual cleanup via admin dashboard if needed

### Vendor Heuristic
- **Risk:** False positive merge (two different vendors with same phone/email)
- **Mitigation:** Require BOTH same category AND shared contact; log all heuristic merges
- **Risk:** Different vendors legitimately sharing a phone (agency representing multiple vendors)
- **Mitigation:** Rare case; category check prevents most false positives

---

## Testing Approach

### Unit Tests
- `extraction-merger.test.ts`: New test cases for `heuristicVendorMerge()`
- `extraction-provider.test.ts`: Mock token usage tracking

### Integration Tests
- Test analytics columns are populated after successful import
- Test token budget abort works (mock provider to return high token counts)

### E2E Tests
- Existing import tests should continue passing
- No new E2E tests needed (changes are backend-only)

### Manual Testing
- Run import with multiple files, verify analytics in DB
- Trigger cleanup cron manually, verify files deleted from storage

---

## Recommended Scope for Size:S

### Include (Core Scope)
1. **Analytics columns** — simple schema change + logging
2. **Token tracking** — add to provider response, log to DB
3. **File retention** — schema + cron job
4. **Vendor heuristic** — new merge pass

### Defer (Follow-up Ticket)
- **80% warning UI** — frontend changes to display budget warning
- **Analytics dashboard** — reporting UI for import metrics
- **Edge Function for cleanup** — if Vercel Cron proves insufficient

---

## Implementation Order

1. **Database migrations first** — analytics columns, retention column
2. **Vendor heuristic** — self-contained, easy to test
3. **Token tracking** — requires provider changes
4. **File retention cron** — depends on retention column
5. **Wire up analytics logging** — final step, connects everything

Estimated effort: **4-6 hours** for core scope.
