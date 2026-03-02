# Board State
Updated: 2026-03-02T13:30:00.000Z

## In Progress

### Ticket 5: Incremental import for existing weddings
- **Assignee:** Qamar
- **Plan Status:** Approved
- **Latest Update:** Dev complete, awaiting QA (PR #338)
- **Severity:** Medium | **Size:** L | **Type:** Feature
- **Depends On:** Tickets 1-4 (foundational)
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/in-progress/05-ticket-5-incremental-import.md
- **Comments:** /home/jcavendish/workspace/plannifier-team/backlog/in-progress/05-ticket-5-incremental-import.comments.md
> Enable per-section import into existing weddings. Currently, import only creates new weddings. Planners need to backfill missing data into manually-created weddings (guests, vendors, budgets). Includes preview/diff mode showing what will create/update/skip before committing.

## Done

### Ticket 1: Prompt quality — extraction accuracy fixes
- **Assignee:** —
- **Severity:** High | **Size:** S | **Type:** Bug Fix
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/01-ticket-1-prompt-quality.md
> Fix three related prompt quality issues.

### Ticket 2: Installment pipeline end-to-end
- **Assignee:** Devon
- **Severity:** High | **Size:** M | **Type:** Feature
- **Depends On:** Ticket 1
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/02-ticket-2-installment-pipeline.md
> Complete end-to-end pipeline for payment installments.

### Ticket 3: Document attachment & vendor contracts
- **Assignee:** Qamar
- **Severity:** High | **Size:** L | **Type:** Feature
- **Depends On:** Ticket 1
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/03-ticket-3-document-attachment-vendor-contracts.md
> Create vendor_contracts as first-class entities and attach source documents to vendors/contracts.

### Ticket 4: Vendor update-on-match instead of skip
- **Assignee:** Qamar
- **Severity:** Medium | **Size:** S | **Type:** Feature
- **Depends On:** Ticket 1
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/04-ticket-4-vendor-update-on-match.md
> When planner import detects existing vendor, update empty fields instead of skipping.

### Ticket 6: Review UI redesign — swipable per-type cards
- **Assignee:** Devon (follow-up: ?scenario=vendor-update in test page)
- **Plan Status:** Approved
- **Latest Update:** Merged to staging 2026-03-02. 58/63 E2E pass — 5 failing tests in document-import-vendor-update.critical.spec.ts need test helper page fix.
- **Severity:** Medium | **Size:** M | **Type:** Refactor
- **PR:** https://github.com/jcavendish/plannifier/pull/337 (merged)
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/06-ticket-6-review-ui-redesign.md
> Redesign review screen from single-scroll page to swipable cards/tabs per entity type.

### Ticket 7: Housekeeping & observability
- **Assignee:** —
- **Plan Status:** Approved
- **Latest Update:** Merged to staging 2026-03-02.
- **Severity:** Low | **Size:** S | **Type:** Refactor
- **PR:** https://github.com/jcavendish/plannifier/pull/339 (merged)
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/07-ticket-7-housekeeping-observability.md
> Add import analytics tracking, per-import token budget ceiling, file retention cleanup, and cross-document vendor dedup heuristic.

### Ticket 8: Fix type-check OOM crash during pnpm run test
- **Assignee:** Devon
- **Severity:** High | **Size:** M | **Type:** Bug Fix
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/08-ticket-8-fix-type-check-oom.md
> The TypeScript type-check phase during `pnpm run test` crashes with OOM on limited RAM systems.

### Ticket 9: Mobile layout — vendor name hidden behind contract badges
- **Assignee:** Review: Joao
- **Severity:** Medium | **Size:** XS | **Type:** Bug Fix
- **Latest Update:** QA passed — ready for PR merge (2026-03-02)
- **PR:** https://github.com/jcavendish/plannifier/pull/336
- **Depends On:** Ticket 3 (landed)
- **File:** /home/jcavendish/workspace/plannifier-team/backlog/done/09-ticket-9-mobile-vendor-contract-badge-overflow.md
> On small viewports (≤390px), vendor name collapses behind badge container. Fixed with flex-wrap on badge div.
