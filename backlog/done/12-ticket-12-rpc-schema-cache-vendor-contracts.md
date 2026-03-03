**Status:** Done
**Assignee:** Review: Joao
**Severity:** Critical | **Size:** XS | **Type:** Bug Fix
**PR:** https://github.com/jcavendish/plannifier/pull/341
**Latest Update:** QA PASS (Qamar, Round 2, 2026-03-03) — 5/5 E2E pass. db reset clean. Ready for Joao review.

---

# Ticket 12: Fix RPC overload — create_wedding_from_import fails with p_vendor_contracts

## Problem

"Create wedding" button in the import review wizard fails 100% of the time with:
```
RPC create_wedding_from_import failed: Could not find the function
public.create_wedding_from_import(...p_vendor_contracts...) in the schema cache
```

## Root Cause

`CREATE OR REPLACE FUNCTION` with a different parameter count creates a new overloaded function, not a replacement. After migration `20260228100000` added `p_vendor_contracts`, two overloads coexisted in the DB:
- 10-param (from `20260227200000` — installments)
- 11-param (from `20260228100000` — vendor contracts)

PostgREST could not unambiguously resolve the correct overload → schema cache error → 500.

On a fresh `db reset`, `20260228100000` also failed because its `COMMENT ON FUNCTION` (without parameter types) raises `42725` when two overloads exist — halting the reset before `20260302120000` could run.

## Fix

Three migrations in the chain:
1. **`20260227210000_drop_10param_rpc_overload.sql`** (new) — drops the 10-param overload before `20260228100000` runs, so the COMMENT ON FUNCTION succeeds
2. **`20260228100000_add_vendor_contracts_to_import_rpc.sql`** — unchanged, now runs cleanly
3. **`20260302120000_fix_rpc_drop_old_overloads.sql`** — drops all known overloads, creates canonical 11-param version, calls `NOTIFY pgrst, 'reload schema'`

Also added RPC overload discipline rule to `CLAUDE.md`.

## QA Results

### Round 1 (2026-03-03) — FAIL
- `db reset` failed at `20260228100000` — COMMENT ON FUNCTION raised 42725
- `20260302120000` never ran
- Returned to Devon with: add `20260227210000` to drop 10-param overload first

### Round 2 (2026-03-03) — PASS ✅
After commits `d77b33c` (add `20260227210000`) and `f5f1178` (`extensions.gen_random_bytes` fix):

**Migration verification:**
- `db reset` completes without error ✅
- `SELECT count(*) FROM pg_proc WHERE proname = 'create_wedding_from_import'` → 1 ✅

**API flows (5/5 pass):**
- Flow 1: Full payload with vendor_contracts → HTTP 200 + weddingId ✅
- Flow 2: Minimal payload (couple only) → HTTP 200 + weddingId ✅
- Flow 3: Unauthenticated → HTTP 401 ✅
- Flow 4: Import in 'creating' state → HTTP 409 ✅
- Flow 5: Malformed payload → HTTP 400 ✅

**Static analysis:**
- Lint: 0 errors (86 pre-existing warnings, unchanged from staging) ✅
- Type-check: 0 errors ✅

**E2E test added:** `tests/document-import-create-wedding.critical.spec.ts` — 5 tests covering the real API endpoint (no mocks).
