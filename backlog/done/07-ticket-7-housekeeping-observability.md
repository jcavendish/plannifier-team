# Ticket 7: Housekeeping & observability

**Status:** Done
**Assignee:** —
**PR:** https://github.com/jcavendish/plannifier/pull/339 (merged)
**Plan Status:** Approved
**Latest Update:** Merged to staging 2026-03-02. Conflicts resolved (confidence badge locator scoping).
**Severity:** Low
**Size:** S
**Type:** Refactor

## Summary

Add import analytics tracking, per-import token budget ceiling, file retention cleanup, and cross-document vendor dedup heuristic (same category + same contact → merge).

Three independent quality-of-life improvements: observability (analytics), cost control (token budget), data hygiene (file cleanup), and edge case handling (vendor dedup).

## #23 — Import analytics and cost tracking

### Analytics

- Track: import count, extraction accuracy (confidence distribution), time-to-confirmation, entity-type breakdown
- Create `import_analytics` table: wedding_id, planner_id, file_count, entity_counts (vendors, guests, budget), avg_confidence, time_taken, token_usage, cost
- Log after each import completes
- Helps measure feature adoption and identify quality bottlenecks

### Cost ceiling

- Add per-import token budget: $1 ceiling (abort if exceeded)
- Track token usage in extraction provider
- Warn user if approaching limit: "This import is using 80% of token budget"

### File retention

- Files stored in Supabase Storage indefinitely
- Goal: 48-hour retention with automatic cleanup
- Create scheduled job (pg_cron) to delete files older than 2 days
- Policy: store retention date on file metadata

## #13 — Cross-document vendor name dedup (edge case heuristic)

### Problem

Two documents reference same vendor with different names (legal vs brand name). Levenshtein similarity fails because names have zero string overlap.

### Solution

Heuristic — if two vendors have:
- Same service category AND
- Shared contact info (phone OR email match)
- → Merge them, preferring brand name

### Implementation

- Add `heuristicVendorMerge()` in `extraction-merger.ts`
- Call after Levenshtein pass
- Log merge reasoning for transparency

### Example

"CM Oliveira comércio de bolos" + "CASSIA PEREIRA BOLOS ARTISTICOS" → both have phone 11-98765-4321, both category=Catering → merge

## Files Changed

- `process/route.ts`
- `entity-filter.ts`
- `extraction-merger.ts` (new heuristic)
- New analytics table
- Supabase Storage policies

## Related Issues

- #23 — Import analytics
- #13 — Vendor dedup heuristic

## Epic

Import Feature - Polish

## Notes

No dependencies; can be done in parallel or anytime
