# Ticket 16: Vendor contracts not extracted — import review shows empty contracts tab

**Status:** Open
**Assignee:** —
**Severity:** High
**Size:** S
**Type:** Bug Fix
**Depends On:** Ticket 15 (rate limit fix)

## Summary

When importing vendor contract documents (PDFs with contract details, dates, service terms), the import review screen shows 0 vendor contracts in the "Vendor Contracts" tab — even though the documents clearly contain contract information.

## Problem

The `ImportReviewVendorContractsSection` UI component exists and renders correctly. The extraction schema (`ExtractedVendorContractSchema`) and RPC insertion path are all wired up. The contracts array is simply arriving empty after extraction.

## Root Cause (Primary — Ticket 15 dependency)

The classify step is failing with 429 errors (see Ticket 15). When classification fails, the file errors out entirely and contributes no data to the merged extraction result. `vendorContracts` defaults to `[]`. This is the most likely explanation.

## Root Cause (Secondary — to verify after Ticket 15 is fixed)

Even if classification succeeds, there may be an extraction quality issue. The extraction prompt for `vendor_contract` type documents instructs Claude to populate `vendorContracts`, but LLM output variance could cause the array to be omitted or empty for certain document formats (e.g., Brazilian Portuguese contracts with non-standard layouts).

## Proposed Fix

1. **Fix Ticket 15 first** — once classify stops failing, verify whether contracts start appearing.
2. **If still empty after Ticket 15**: Debug extraction output for a `vendor_contract` type document. Log the raw LLM response before Zod parsing to check if `vendorContracts` is present in the JSON.
3. **If LLM omits `vendorContracts`**: Strengthen the extraction prompt with a more explicit instruction and an example JSON structure for the `vendorContracts` array. Consider adding a dedicated validation step that warns when a `vendor_contract` classified document produces an empty `vendorContracts` array.

## Acceptance Criteria

- [ ] Importing a vendor contract PDF produces at least one entry in the vendor contracts tab of the review screen
- [ ] Contract data includes: vendor name, service description, contract amount, deposit, dates, status
- [ ] Empty tab shows only when there genuinely are no contract fields in the document
- [ ] If extraction quality is low (LLM variance), a `confidence: 'low'` warning is shown

## Files

- `frontend/src/lib/services/import/extraction-provider.ts` — extraction output
- `frontend/src/lib/services/import/prompts/extraction-prompt.ts` — `vendor_contract` instructions
- `frontend/src/lib/services/import/extraction-schemas.ts` — `ExtractedVendorContractSchema`
- `frontend/src/components/documents/ImportReviewVendorContractsSection.tsx`

## Latest Update

Blocked on Ticket 15. Once rate limit bug is fixed, verify if contracts appear. If not, debug extraction quality.
