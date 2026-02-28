# Ticket 1: Prompt quality — extraction accuracy fixes

**Status:** Done
**Assignee:** —
**Severity:** High
**Size:** S
**Type:** Bug Fix

## Summary

Fix three related prompt quality issues:
1. Refine rule #2 to allow contracting party ('Contratante') as legitimate partner source while still blocking vendor contact inference (#11)
2. Add task date inference from event date and filter vendor obligations from task extraction (#16)
3. Strengthen category→predefined list routing to prevent vendor names leaking into category field (#26 partial)

No schema changes, no migrations, no RPC changes.

## Changes Required

### #11 — Contracting party name not detected as partner

- Refine `BASE_EXTRACTION_INSTRUCTIONS` rule #2 in `extraction-prompt.ts:118-120`
- Currently says "NEVER infer couple/partner fields from vendor contracts" (too aggressive)
- Should distinguish:
  - **Contracting party** ("Contratante", "Parte contratante") → allowed for `partner1Name` with confidence "medium"
  - **Vendor contact** ("Responsável", contact section) → still prohibited for couple fields

### #16 — Tasks lack inferred dates and include vendor obligations

- Add date inference: "When extracting tasks from vendor contracts, use the event/service date as reference. Infer reasonable due dates (e.g., confirm headcount → 2 weeks before event)"
- Add task definition: "Tasks are planner responsibilities. Skip vendor contractual obligations (what vendor will provide/deliver)"
- Add to both `extraction-prompt.ts` and `extraction-provider.ts` buildCombinedPrompt

### #26 (partial) — Budget categories with vendor names

- Review `entity-filter.ts:normalizeBudgetCategory()` — ensure mapping table catches vendor names
- Add post-extraction validation: if category contains vendor-like words (company patterns), route to "Miscellaneous"

## Files Changed

- `extraction-prompt.ts`
- `extraction-provider.ts`
- `entity-filter.ts`
- tests

## Related Issues

- #11, #16, #26 (category part)

## Epic

Import Feature - Batch F

## Notes

Foundational ticket. Tickets 2-4 can proceed in parallel after this one is solid.

## Latest Updates

See `01-ticket-1-prompt-quality.comments.md` for full discussion history.
