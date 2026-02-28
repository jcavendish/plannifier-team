# Ticket 3: Document attachment & vendor contracts

**Status:** Done
**Assignee:** Qamar
**Severity:** High
**Size:** L
**Type:** Feature
**Depends On:** Ticket 1

## Summary

Create vendor_contracts as first-class entities and attach source documents to vendors/contracts. Track which file produced which entity through extraction, merge, and wedding creation pipelines.

## Current State (Incomplete)

- Budget item (amount only)
- Vendor (company info)
- ❌ vendor_contracts row (service description, dates, terms, payment schedule)
- ❌ vendor_contracts document attachment (PDF/image lost)

## Target State (Goal)

- Budget item (amount, category)
- Vendor (company info, contact)
- ✅ vendor_contracts row (service_description, contract_amount, deposit_amount, payment_schedule, start_date, end_date)
- ✅ vendor_contracts document attachment (link to source file)

## Changes Required

### 1. Extraction Pipeline

- Track source `fileId` on each extracted entity during processing
- After merge, preserve file associations (merged vendor links to all source files)

### 2. Schema (`extraction-schemas.ts`)

- Add `vendorContracts: { vendorName, serviceDescription, contractAmount, depositAmount, paymentSchedule, startDate, endDate, sourceFileId }[]` to `ExtractedWedding`

### 3. Prompts

Both `extraction-prompt.ts` and `buildCombinedPrompt`:
- Extract contract-specific fields: service description, contract dates, payment terms, not just amounts

### 4. RPC Migration (`create_wedding_from_import`)

- Match vendor_contracts to vendors by name
- Create `vendor_contracts` rows
- Attach documents via `vendor_contracts_documents` join table

### 5. wedding-creator.ts

- Extract vendor contract data
- Pass file references through

### 6. Review UI

- Show vendor_contracts as reviewable entity type
- Display service descriptions, dates, amounts

## Files Changed

- Extraction pipeline
- `extraction-schemas.ts`
- `extraction-prompt.ts`
- `extraction-provider.ts`
- `wedding-creator.ts`
- Review UI components
- Migration RPC

## Related Issues

- #28 — Documents not attached after import
- #20 — Vendor contracts lost during flattening
- #17 — Partial (document tracking)

## Epic

Import Feature - Batch G

## Parallel Work

Can proceed in parallel with Tickets 2 and 4 after Ticket 1 is solid.

## Latest Updates

See `03-ticket-3-document-attachment-vendor-contracts.comments.md` for full discussion.

**Recent:**
- Qamar — 2026-02-28 09:45: Starting QA testing

---

## QA Sign-off (Qamar)

**Round 1 — FAIL:** TypeScript errors in 4 test fixtures (missing `vendorContracts: []` in mock factories). Fixed by Devon (commit cd0851f).

**Round 2 — PASS ✅**
- Type-check: clean
- Unit tests: 959 pass, 0 fail
- Browser testing: 30 flows across desktop, mobile, en + pt-BR locales
- E2E tests: 9 new tests written and passing (`document-import-vendor-contracts.critical.spec.ts`)

**Medium issue noted (not blocking):** Vendor name invisible on mobile when contract has both source badge + status badge. `shrink-0` on badge container collapses name span to `width:0`. Recommend follow-up ticket.

**Verdict:** PASS. Ready for PR. 🟢
