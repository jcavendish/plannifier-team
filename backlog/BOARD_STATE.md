# Board State
Updated: 2026-03-04T10:40:12.699Z

## Open (unassigned)

### Ticket 14: No user feedback after wedding creation
- **Assignee:** —
- **Severity:** High | **Size:** XS | **Type:** Bug Fix
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/open/14-ticket-14-no-feedback-after-wedding-creation.md
> After clicking "Create Wedding" in the import review wizard, the button stops loading but nothing else happens — no success toast, no navigation to the new wedding. The wedding may actually be created in the database but the user has no way to know or reach it.

### Ticket 15: Import rate limiting — 429 on concurrent file classification
- **Assignee:** —
- **Severity:** High | **Size:** S | **Type:** Bug Fix
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/open/15-ticket-15-import-rate-limit-429-concurrent-classify.md
> When importing multiple files, the classify step fails with HTTP 429 (rate limit exceeded). Both concurrent classify calls hit the Anthropic API simultaneously, exhaust the 50K input tokens/minute org limit together, and then retry at the same time — causing repeated failures and degraded/failed imports.

### Ticket 16: Vendor contracts not extracted — import review shows empty contracts tab
- **Assignee:** —
- **Severity:** High | **Size:** S | **Type:** Bug Fix
- **Depends On:** Ticket 15 (rate limit fix)
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/open/16-ticket-16-vendor-contracts-not-extracted-in-import.md
> When importing vendor contract documents (PDFs with contract details, dates, service terms), the import review screen shows 0 vendor contracts in the "Vendor Contracts" tab — even though the documents clearly contain contract information.

### Ticket 17: Source documents not linked to wedding after import creation
- **Assignee:** —
- **Severity:** Medium | **Size:** L | **Type:** Feature
- **Depends On:** Tickets 14, 15, 16 (import pipeline stable)
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/open/17-ticket-17-source-documents-not-linked-to-wedding.md
> When a wedding is created from an import, the original uploaded PDF/document files are not attached to the wedding, vendors, or budget items. Planners lose the connection between the raw source documents and the entities that were created from them. There is also no UI to review or manage attached documents inside a wedding.

## Done

### Ticket 1: Prompt quality — extraction accuracy fixes
- **Assignee:** —
- **Severity:** High | **Size:** S | **Type:** Bug Fix
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/01-ticket-1-prompt-quality.md
> Fix three related prompt quality issues:

### Ticket 2: Installment pipeline end-to-end
- **Assignee:** Devon
- **Severity:** High | **Size:** M | **Type:** Feature
- **Depends On:** Ticket 1
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/02-ticket-2-installment-pipeline.md
> Complete end-to-end pipeline for payment installments. Add installments array to extraction schema, update prompts to populate it, create budget_item_installments rows in RPC, and display in review UI. Solves #27 (no installment rows created) and #26 partial (payment terms dumped in description).

### Ticket 3: Document attachment & vendor contracts
- **Assignee:** Qamar
- **Severity:** High | **Size:** L | **Type:** Feature
- **Depends On:** Ticket 1
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/03-ticket-3-document-attachment-vendor-contracts.md
> Create vendor_contracts as first-class entities and attach source documents to vendors/contracts. Track which file produced which entity through extraction, merge, and wedding creation pipelines.

### Ticket 4: Vendor update-on-match instead of skip
- **Assignee:** Qamar
- **Severity:** Medium | **Size:** S | **Type:** Feature
- **Depends On:** Ticket 1
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/04-ticket-4-vendor-update-on-match.md
> When planner import detects existing vendor (by email/phone/name), update empty fields instead of skipping. Adopt admin import's 3-action model: create (new), update (fill gaps), match (no change).

### Ticket 5: Incremental import for existing weddings
- **Assignee:** —
- **Plan Status:** Approved
- **Latest Update:** Merged to staging 2026-03-02. QA passed Round 2 (bc25be8)
- **Severity:** Medium | **Size:** L | **Type:** Feature
- **Depends On:** Tickets 1-4 (foundational)
- **PR:** https://github.com/jcavendish/plannifier/pull/338 (merged)
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/05-ticket-5-incremental-import.md
> Enable per-section import into existing weddings. Currently, import only creates new weddings. Planners need to backfill missing data into manually-created weddings (guests, vendors, budgets). Includes preview/diff mode showing what will create/update/skip before committing.

### Ticket 6: Review UI redesign — swipable per-type cards
- **Assignee:** —
- **Plan Status:** Approved
- **Latest Update:** Fully resolved — Ticket 10 fixed the 5 remaining E2E failures. 63/63 pass.
- **Severity:** Medium | **Size:** M | **Type:** Refactor
- **Depends On:** Tickets 2-3 (schema expansion)
- **PR:** https://github.com/jcavendish/plannifier/pull/337 (merged 2026-03-02T11:53Z)
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/06-ticket-6-review-ui-redesign.md
> Redesign review screen from single-scroll page to swipable cards/tabs per entity type. Needed for scalability as schema expands to 10+ entity types.

### Ticket 7: Housekeeping & observability
- **Assignee:** —
- **Plan Status:** Approved
- **Latest Update:** Merged to staging 2026-03-02. Conflicts resolved (confidence badge locator scoping).
- **Severity:** Low | **Size:** S | **Type:** Refactor
- **PR:** https://github.com/jcavendish/plannifier/pull/339 (merged)
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/07-ticket-7-housekeeping-observability.md
> Add import analytics tracking, per-import token budget ceiling, file retention cleanup, and cross-document vendor dedup heuristic (same category + same contact → merge).

### Ticket 8: Fix type-check OOM crash during pnpm run test
- **Assignee:** Devon
- **Severity:** High | **Size:** M | **Type:** Bug Fix
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/08-ticket-8-fix-type-check-oom.md
> The TypeScript type-check phase during `pnpm run test` crashes with out-of-memory errors on systems with limited RAM, blocking CI/CD pipelines and local development. Requires optimization of type-checking configuration and/or tooling.

### Ticket 9: Mobile layout — vendor name hidden behind contract badges
- **Assignee:** —
- **Latest Update:** QA passed (Qamar, 9/9 E2E) — merged to staging 2026-03-02
- **Severity:** Medium | **Size:** XS | **Type:** Bug Fix
- **Depends On:** Ticket 3 (landed)
- **PR:** https://github.com/jcavendish/plannifier/pull/336 (merged 2026-03-02)
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/09-ticket-9-mobile-vendor-contract-badge-overflow.md
> On small viewports (≤390px), when a vendor contract card shows both the "Source document attached" badge AND a status badge simultaneously, the vendor name `<span>` collapses to width:0 and becomes invisible.

### Ticket 10: Fix E2E — missing `?scenario=vendor-update` on test page
- **Assignee:** —
- **Latest Update:** Merged to staging 2026-03-02 (PR #340).
- **Severity:** Medium | **Size:** XS | **Type:** Bug Fix
- **Depends On:** Ticket 6 (landed)
- **PR:** https://github.com/jcavendish/plannifier/pull/340
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/10-ticket-10-e2e-vendor-update-scenario.md
> 5 E2E tests in `document-import-vendor-update.critical.spec.ts` were failing because the test helper page does not implement `?scenario=vendor-update`. As a result, vendors never receive `catalogMatch` data and the match/update badges never render — causing assertion failures on badge visibility.

### Ticket 11: Translate vendor update field names in import review badge
- **Assignee:** Review: Joao
- **Plan Status:** Approved
- **Latest Update:** QA passed (Qamar, 2026-03-03) — code review + i18n verification. Build clean, all flows correct. Ready for Joao review.
- **Severity:** Low | **Size:** XS | **Type:** Bug Fix
- **PR:** https://github.com/jcavendish/plannifier/pull/342
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/11-ticket-11-vendor-update-field-labels.md
> In the import review wizard (Vendors tab), when an extracted vendor matches an existing catalog entry and has fields to gap-fill, the badge reads "Vai atualizar: contact_person" or "Will update: phone" — raw DB column names instead of human-readable labels.

### Ticket 12: Fix RPC overload — create_wedding_from_import fails with p_vendor_contracts
- **Assignee:** —
- **Severity:** Critical | **Size:** XS | **Type:** Bug Fix
- **PR:** https://github.com/jcavendish/plannifier/pull/341
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/12-ticket-12-rpc-schema-cache-vendor-contracts.md
> "Create wedding" button in the import review wizard fails 100% of the time with:

### Ticket 13: Fix vendor-financial-empty data-testid on empty state
- **Assignee:** Review: Joao
- **Latest Update:** QA PASS (Qamar, 2026-03-03) — 12/12 vendor-track-derived tests pass. Ready for Joao review.
- **Severity:** Medium | **Size:** XS | **Type:** Bug Fix
- **PR:** https://github.com/jcavendish/plannifier/pull/343
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/13-ticket-13-vendor-financial-empty-testid.md
> `VendorFinancialSummary.tsx` empty state branch rendered `data-testid="vendor-financial-summary-{id}"` instead of the expected `data-testid="vendor-financial-empty"`. This caused 3 tests in `vendor-track-derived.critical.spec.ts` to fail with count = 0.
