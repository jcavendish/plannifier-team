# Ticket 2: Installment pipeline end-to-end

**Status:** In Progress
**Assignee:** Devon
**Severity:** High
**Size:** M
**Type:** Feature
**Depends On:** Ticket 1

## Summary

Complete end-to-end pipeline for payment installments. Add installments array to extraction schema, update prompts to populate it, create budget_item_installments rows in RPC, and display in review UI. Solves #27 (no installment rows created) and #26 partial (payment terms dumped in description).

## Goal

Enable extraction of vendor payment installments as structured data, creating proper `budget_item_installments` rows instead of burying details in budget item descriptions.

### Architecture

Vendor contract "Total: R$ 85,000, Pay R$ 17,000 now, 4 installments of R$ 17,000" should produce:
- 1 `budget_item` with amount=85,000, category, vendor
- 5 `budget_item_installments` with (amount, dueDate, description, status)

## Changes Required

### 1. Schema (`extraction-schemas.ts`)

- Add `installments: { amount, dueDate, description, status }[]` to `ExtractedBudgetItemSchema`
- Update zod validation

### 2. Prompts

Both `extraction-prompt.ts` and `extraction-provider.ts:buildCombinedPrompt`:
- Tell AI to populate installments array instead of putting terms in description
- Example: "If contract shows installment schedule, extract into installments array: [{amount: 17000, dueDate: '2026-07-15', description: 'Installment 1', status: 'pending'}]"

### 3. RPC Migration (`create_wedding_from_import`)

- Loop through budgetItems[].installments
- Create `budget_item_installments` rows after budget_item created

### 4. wedding-creator.ts

- Extract installment data from extracted budget items
- Pass through to RPC

### 5. Review UI

- Show installments as sub-rows under budget items
- Display status badges (pending/paid/overdue)

## Files Changed

- `extraction-schemas.ts`
- `extraction-prompt.ts`
- `extraction-provider.ts`
- `wedding-creator.ts`
- Review UI components
- Migration RPC

## Related Issues

- #27 — No installment rows created
- #26 — Payment details now have proper home instead of description field
- #15 — Prompt-only fix (Batch E) — needs schema extension
- #17 — Partial (installment tracking)

## Epic

Import Feature - Batch G

## Parallel Work

Can proceed in parallel with Tickets 3-4 after Ticket 1 is solid.
