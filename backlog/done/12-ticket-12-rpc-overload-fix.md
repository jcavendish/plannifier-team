# Ticket 12: Fix create_wedding_from_import RPC overload schema cache error

**Status:** Done
**Assignee:** Review: Joao
**Severity:** Critical | **Size:** S | **Type:** Bug Fix
**PR:** https://github.com/jcavendish/plannifier/pull/341
**Latest Update:** QA PASS (Qamar, 2026-03-03) — db reset clean, 5/5 integration E2E. Ready for Joao review.

## Summary

"Create Wedding" button fails 100% in production with RPC schema cache error.
Root cause: `CREATE OR REPLACE FUNCTION` with a different parameter count creates a new overload
rather than replacing the existing one. PostgREST cannot resolve ambiguous overload → 500.

## Fix (3 commits)

1. `20260227210000` — drop 10-param overload before `20260228100000` runs (fixes `db reset`)
2. `20260302120000` — drop both overloads, recreate canonical 11-param, NOTIFY pgrst
3. `f5f1178` — qualify `gen_random_bytes` → `extensions.gen_random_bytes` (local Supabase fix)
4. `914ae38` — add integration E2E test covering the full creation round-trip (Qamar, QA)

## QA Results

- `supabase db reset --local`: ✅ clean
- PostgREST: ✅ resolves single canonical function (permission denied, not not-unique)
- Integration E2E 5/5: full payload with vendor_contracts ✅, minimal ✅, 401 ✅, 409 ✅, 400 ✅
