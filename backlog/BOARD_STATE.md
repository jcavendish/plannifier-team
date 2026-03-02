# Board State
Updated: 2026-03-02T13:16:52.446Z

## In Progress

### Ticket 5: Incremental import for existing weddings
- **Assignee:** Qamar
- **Plan Status:** Approved
- **Latest Update:** Dev complete, awaiting QA (PR #338)
- **Severity:** Medium | **Size:** L | **Type:** Feature
- **Depends On:** Tickets 1-4 (foundational)
- **PR:** https://github.com/jcavendish/plannifier/pull/338
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/in-progress/05-ticket-5-incremental-import.md
> Enable per-section import into existing weddings. Planners need to backfill missing data into manually-created weddings (guests, vendors, budgets). Includes preview/diff mode showing what will create/update/skip before committing.

### Ticket 9: Mobile layout — vendor name hidden behind contract badges
- **Assignee:** Qamar
- **Severity:** Medium | **Size:** XS | **Type:** Bug Fix
- **Depends On:** Ticket 3 (landed)
- **PR:** https://github.com/jcavendish/plannifier/pull/336
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/in-progress/09-ticket-9-mobile-vendor-contract-badge-overflow.md
> On small viewports (≤390px), when a vendor contract card shows both the "Source document attached" badge AND a status badge simultaneously, the vendor name `<span>` collapses to width:0 and becomes invisible.

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

### Ticket 6: Review UI redesign — swipable per-type cards
- **Assignee:** —
- **Plan Status:** Approved
- **Latest Update:** Merged to staging 2026-03-02T11:53Z (PR #337)
- **Severity:** Medium | **Size:** M | **Type:** Refactor
- **Depends On:** Tickets 2-3 (schema expansion)
- **PR:** https://github.com/jcavendish/plannifier/pull/337 (merged)
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
