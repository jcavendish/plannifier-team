# Ticket 12: Fix RPC overload — create_wedding_from_import fails with p_vendor_contracts

**Status:** In Progress
**Assignee:** Qamar
**PR:** https://github.com/jcavendish/plannifier/pull/341
**Severity:** Critical
**Size:** XS
**Type:** Bug Fix
**Depends On:** —

## Summary

"Create wedding" button in the import review wizard fails 100% of the time with:

```
RPC create_wedding_from_import failed: Could not find the function
public.create_wedding_from_import(p_budget_items, p_client, p_guest_groups,
p_meal_options, p_phases, p_planner_id, p_tasks, p_timeline, p_vendor_contracts,
p_vendors, p_wedding) in the schema cache
```

The import feature cannot create weddings. This is a complete blocker.

## Root Cause

PostgreSQL `CREATE OR REPLACE FUNCTION` with a **different parameter list** creates a new
overloaded function — it does **not** replace the existing one.

Migration chain:
1. `20260227200000_add_installments_to_import_rpc.sql` — defines function with **10 params** (no `p_vendor_contracts`)
2. `20260228100000_add_vendor_contracts_to_import_rpc.sql` — calls `CREATE OR REPLACE FUNCTION` with **11 params** (adds `p_vendor_contracts`)

Result: DB now has **two overloaded versions** of `create_wedding_from_import`:
- v1: 10 params (no `p_vendor_contracts`)
- v2: 11 params (with `p_vendor_contracts`)

PostgREST resolves named-parameter RPC calls via its schema cache. When the Supabase API service starts, it caches the known function signatures. If the cache was built before v2 was created — or if the multiple overloads cause resolution ambiguity — PostgREST cannot find the correct function and throws the "not found in schema cache" error.

## Why QA Didn't Catch It

The E2E test for `create_wedding_from_import` (`document-import-review.critical.spec.ts`)
tests the full review UI flow but mocks the API response at the network layer — it does not
call the actual Supabase RPC in the test environment. The overloading issue only surfaces
against a live Supabase instance where PostgREST is running.

This is a gap in test coverage: no integration test verifies the RPC is callable end-to-end
against a real Supabase instance. That's a known tradeoff for E2E tests that use network mocks.

## Fix

Two-part fix:

### Part 1 — Hotfix migration (immediate)

Drop ALL existing overloads of `create_wedding_from_import` before recreating the canonical
11-parameter version. This removes the ambiguity and forces PostgREST to learn only one signature.

**File:** `supabase/migrations/YYYYMMDDHHMMSS_fix_rpc_drop_old_overloads.sql`

```sql
-- Drop all existing overloads to eliminate PostgREST resolution ambiguity
DROP FUNCTION IF EXISTS public.create_wedding_from_import(UUID, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB);
DROP FUNCTION IF EXISTS public.create_wedding_from_import(UUID, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB);

-- Then: CREATE OR REPLACE with the canonical 11-param signature (copy from 20260228100000)
```

After applying the migration, also signal PostgREST to reload its schema cache:
```sql
NOTIFY pgrst, 'reload schema';
```

### Part 2 — Future migrations discipline

Every future migration that changes the RPC signature MUST drop old overloads first:
```sql
-- Always drop old overloads before redefining
DROP FUNCTION IF EXISTS public.create_wedding_from_import(UUID, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB, JSONB);
CREATE OR REPLACE FUNCTION public.create_wedding_from_import(...new signature...)
```

Document this in `CLAUDE.md` under migration discipline.

## Immediate Workaround (while fix is deployed)

On Supabase Cloud dashboard: **Settings → API → Reload schema cache** (or restart API service).
This may temporarily resolve the cache issue without a migration, but the overload ambiguity
will persist until Part 1 is applied.

## Files to Change

- New migration `supabase/migrations/YYYYMMDDHHMMSS_fix_rpc_drop_old_overloads.sql`
- `frontend/CLAUDE.md` — add note under migrations: DROP old overloads before CREATE OR REPLACE

## Complexity

XS — migration only, no application code changes.
