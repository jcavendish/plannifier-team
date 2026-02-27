# Ticket 2: Installment pipeline end-to-end

**Status:** Done
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

---

## QA Results — 2026-02-28

**Status: FAIL** — Build does not compile. Blocking issues found.

### Issues Found

**[HIGH] TypeScript build failure — `installments` field not added to existing test fixtures**

The new `installments` field added to `ExtractedBudgetItemSchema` is required in the output type but missing from several existing mock/test files. Build fails with type errors.

**Affected files (all introduced by this branch):**

1. `frontend/src/app/[locale]/test/import-review/page.tsx` — all `budgetItems` mock entries missing `installments: []`
2. `frontend/src/lib/services/import/__tests__/entity-filter.test.ts` — 6 type errors, mock budget items missing `installments`
3. `frontend/src/lib/services/import/__tests__/extraction-merger.test.ts` — `installments` type mismatch (`undefined` not assignable to `[]`)
4. `frontend/src/lib/services/import/__tests__/extraction-schemas-installments.test.ts:66` — `toHaveLength` not found (Jest array matcher type issue)

**Steps to reproduce:**
```bash
NODE_OPTIONS="--max-old-space-size=4096" pnpm run build
pnpm run test
```

**Expected:** Zero errors
**Actual:** `pnpm run build` → type error at `import-review/page.tsx:77`. `pnpm run test` → 8+ type errors across 3 test files.

**Fix:** Add `installments: []` to every `ExtractedBudgetItem` mock literal in the above files. Fix `toHaveLength` assertion in new unit test.

---

*Note: `file-text-extractor.ts` unused `@ts-expect-error` is PRE-EXISTING on staging — not introduced by this branch.*

---

## QA Sign-off — 2026-02-28

**Status: PASS** ✅

Devon addressed all issues from the first QA cycle. Second run:

- ✅ `pnpm run build` — zero errors
- ✅ `pnpm run lint` — 0 errors, 86 warnings (all pre-existing)
- ✅ `pnpm run test` (type-check) — zero errors
- ✅ Migration `20260227200000_add_installments_to_import_rpc.sql` applied successfully
- ✅ E2E tests: 6/6 passed (new tests added for installments toggle UI)

**Flows tested:**
1. ✅ Toggle row appears only for items with installments
2. ✅ Sub-table hidden by default
3. ✅ Click expands sub-table with correct data (Entrada/paid, 2ª parcela/pending, Final payment/pending)
4. ✅ Second click collapses sub-table
5. ✅ Count label shows "3 installments"
6. ✅ Items without installments show no toggle (Photography confirmed)
7. ✅ i18n strings present in both en + pt-BR

**QA sign-off: Qamar**
